# RabbitMQ: A Complete Guide to Message Queuing with FastAPI

## Introduction

In modern distributed systems, components need to communicate asynchronously to handle tasks efficiently. RabbitMQ is a powerful message broker that implements the Advanced Message Queuing Protocol (AMQP), enabling applications to send and receive messages reliably. In this guide, we'll explore what RabbitMQ is, its use cases, how to install it, and how to integrate it with FastAPI.

## What is RabbitMQ?

RabbitMQ is an open-source message broker software that acts as an intermediary for messaging between applications. It accepts messages from producers and delivers them to consumers, providing features like message routing, acknowledgments, and persistence.

## Why Use RabbitMQ in Your Project?

### The Problem Without RabbitMQ

Imagine you're building a web application where users can upload images. Without a message queue:

1. **Blocking Operations**: When a user uploads an image, your API endpoint processes it immediately (resizing, watermarking, etc.), making the user wait 30+ seconds for a response
2. **Lost Tasks**: If your server crashes during processing, the task is lost forever
3. **No Scalability**: You can't add more workers to handle increased load - everything runs in the main application thread
4. **Resource Exhaustion**: During traffic spikes, all requests pile up, potentially crashing your server

### The Solution With RabbitMQ

With RabbitMQ, the same workflow becomes:

1. **Instant Response**: API receives upload → sends message to queue → immediately returns "Processing..." to user (< 1 second)
2. **Reliability**: Messages persist in the queue even if workers crash; they'll be processed when workers restart
3. **Horizontal Scaling**: Add 10 more worker instances during peak hours to process tasks 10x faster
4. **Decoupling**: Your API and processing logic are separate - update one without touching the other

### Real-World Benefits

**Performance**:

```text
Without RabbitMQ: User uploads image → waits 30 seconds → gets result
With RabbitMQ: User uploads image → gets immediate response → receives notification when done
```

**Reliability**:

- Messages are stored durably and survive server restarts
- Failed tasks can be automatically retried
- Dead letter queues catch problematic messages for manual review

**Flexibility**:

- Need to add thumbnail generation? Just add a new consumer listening to the queue
- Want to process videos too? Create a new queue without changing existing code
- Need to integrate with third-party services? They can consume from your queues

**Cost Efficiency**:

- Scale workers independently based on queue depth
- Use cheaper background workers instead of expensive web servers for heavy processing
- Auto-scale during peak hours, scale down during quiet periods

### When Should You Use RabbitMQ?

**Use RabbitMQ when:**

- Tasks take more than 2-3 seconds to complete
- You need to process tasks asynchronously (emails, notifications, reports)
- You want to distribute work across multiple servers
- You're building microservices that need to communicate
- You need reliability guarantees for important operations
- You want to decouple different parts of your application

**Don't use RabbitMQ when:**

- Your tasks complete in milliseconds and don't block the main thread
- You have a simple, single-server application with low traffic
- You need immediate, synchronous responses (use direct API calls instead)
- Your application is purely read-only with no background processing

### Can Frontend Use RabbitMQ Directly?

**Short Answer: No, and you shouldn't want to.**

RabbitMQ is a **backend-to-backend** communication tool, not meant for frontend applications. Here's why:

#### Architecture That Doesn't Work

```text
❌ WRONG: Frontend → RabbitMQ → Backend Worker
   (Frontend directly publishing to RabbitMQ)
```

**Why this is bad:**

1. **Security Risk**: Exposing RabbitMQ credentials to the browser means anyone can see them in the network inspector
2. **No Authentication**: You can't verify who is uploading - any user could spam your queues
3. **Direct Access**: Users could potentially read messages from other users or manipulate queues
4. **CORS Issues**: RabbitMQ doesn't handle browser CORS policies well
5. **File Size Limits**: Message queues aren't designed for large files (images, videos)

#### Architecture That Works

```text
✅ CORRECT: Frontend → Backend API → RabbitMQ → Background Worker
                      (validates, authenticates, stores file)
```

**How it actually works for image uploads:**

**Step 1: Frontend uploads to your API**

```javascript
// React/Vue/etc frontend code
async function uploadImage(file) {
  const formData = new FormData();
  formData.append("image", file);

  const response = await fetch("https://your-api.com/upload-image", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${userToken}`, // Secure authentication
    },
    body: formData,
  });

  const data = await response.json();
  console.log("Task ID:", data.task_id);

  // Now poll for status or use WebSockets for real-time updates
  checkTaskStatus(data.task_id);
}

async function checkTaskStatus(taskId) {
  const response = await fetch(`https://your-api.com/task-status/${taskId}`);
  const status = await response.json();

  if (status.completed) {
    console.log("Processed image URL:", status.result_url);
  } else {
    // Check again in 2 seconds
    setTimeout(() => checkTaskStatus(taskId), 2000);
  }
}
```

**Step 2: Backend API receives upload, publishes to RabbitMQ**

```python
from fastapi import FastAPI, UploadFile, File, Depends
import pika
import uuid
import os

app = FastAPI()

@app.post("/upload-image")
async def upload_image(
    image: UploadFile = File(...),
    current_user = Depends(get_current_user)  # Authentication
):
    # 1. Validate the user and file
    if not image.content_type.startswith('image/'):
        raise HTTPException(400, "Only images allowed")

    # 2. Save file temporarily or to storage (S3, etc.)
    task_id = str(uuid.uuid4())
    file_path = f"/tmp/{task_id}_{image.filename}"

    with open(file_path, "wb") as f:
        f.write(await image.read())

    # 3. Publish task to RabbitMQ
    connection = get_rabbitmq_connection()
    channel = connection.channel()
    channel.queue_declare(queue='image_processing', durable=True)

    message = {
        "task_id": task_id,
        "user_id": current_user.id,
        "file_path": file_path,
        "filename": image.filename
    }

    channel.basic_publish(
        exchange='',
        routing_key='image_processing',
        body=json.dumps(message),
        properties=pika.BasicProperties(delivery_mode=2)
    )

    connection.close()

    # 4. Return immediately to the frontend
    return {
        "task_id": task_id,
        "status": "processing",
        "message": "Image uploaded successfully and is being processed"
    }

@app.get("/task-status/{task_id}")
async def get_task_status(task_id: str):
    # Check task status from database
    task = await db.get_task(task_id)
    return {
        "task_id": task_id,
        "status": task.status,  # "processing", "completed", "failed"
        "result_url": task.result_url if task.status == "completed" else None
    }
```

**Step 3: Background worker processes the image**

```python
# worker.py - Separate process/service
import pika
import json
from PIL import Image

def process_image(task_data):
    # Do the heavy processing
    image = Image.open(task_data['file_path'])
    image.thumbnail((800, 800))

    # Upload to S3 or storage
    result_url = upload_to_s3(image, task_data['filename'])

    # Update database with result
    db.update_task(task_data['task_id'], {
        'status': 'completed',
        'result_url': result_url
    })

    # Optionally: send notification to user via WebSocket or push notification

def callback(ch, method, properties, body):
    task_data = json.loads(body.decode())
    print(f"Processing task: {task_data['task_id']}")

    try:
        process_image(task_data)
        ch.basic_ack(delivery_tag=method.delivery_tag)
    except Exception as e:
        print(f"Error: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=False)

# Connect and start consuming
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='image_processing', durable=True)
channel.basic_consume(queue='image_processing', on_message_callback=callback)

print('Worker waiting for tasks...')
channel.start_consuming()
```

#### How Frontend Gets Real-Time Updates

Since the frontend can't listen to RabbitMQ directly, use one of these patterns:

**Option 1: Polling** (Simple but less efficient)

```javascript
// Frontend polls the API every few seconds
const pollInterval = setInterval(async () => {
  const status = await checkTaskStatus(taskId);
  if (status.completed) {
    clearInterval(pollInterval);
    displayProcessedImage(status.result_url);
  }
}, 2000);
```

**Option 2: WebSockets** (Better for real-time updates)

```javascript
// Frontend connects to WebSocket
const ws = new WebSocket("wss://your-api.com/ws");

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.task_id === taskId && data.status === "completed") {
    displayProcessedImage(data.result_url);
  }
};
```

**Option 3: Server-Sent Events** (One-way real-time from server)

```javascript
const eventSource = new EventSource(`/task-updates/${taskId}`);

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.status === "completed") {
    displayProcessedImage(data.result_url);
    eventSource.close();
  }
};
```

#### Summary: The Complete Flow

```text
1. User selects image in browser
2. Frontend → uploads to Backend API (with authentication)
3. Backend API → saves file & publishes message to RabbitMQ
4. Backend API → returns task_id to Frontend immediately
5. RabbitMQ → holds message until worker is ready
6. Background Worker → receives message from RabbitMQ
7. Background Worker → processes image (resize, optimize, etc.)
8. Background Worker → updates database with result
9. Frontend → polls API or receives WebSocket notification
10. Frontend → displays processed image to user
```

**Key Takeaway**: RabbitMQ sits **behind your API**, not in front of it. The frontend only ever talks to your API, never to RabbitMQ directly.

### Single Backend vs Multiple Backends: When to Use RabbitMQ?

**Short Answer: Both! RabbitMQ is useful in both scenarios.**

#### Scenario 1: Single Backend Application (Most Common Starting Point)

You can absolutely use RabbitMQ within a **single codebase/project** to offload heavy tasks to background workers.

**Architecture:**

```text
Single Application with Multiple Processes:

┌─────────────────────────────────────────────────┐
│         Your Application (e.g., FastAPI)        │
├─────────────────────────────────────────────────┤
│                                                 │
│  Process 1: API Server (uvicorn)                │
│  └─ Receives HTTP requests                      │
│  └─ Publishes messages to RabbitMQ              │
│  └─ Returns immediately to user                 │
│                                                 │
│         ↓ (via RabbitMQ)                        │
│                                                 │
│  Process 2: Worker (python worker.py)           │
│  └─ Consumes messages from RabbitMQ             │
│  └─ Does heavy processing                       │
│  └─ Updates database                            │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Real Example: E-commerce Site (Single Backend)**

```python
# main.py - Your FastAPI application
from fastapi import FastAPI
import pika
import json

app = FastAPI()

# This runs as your web server (uvicorn main:app)
@app.post("/orders")
async def create_order(order_data: dict):
    # Save order to database
    order = await db.create_order(order_data)

    # Publish tasks to RabbitMQ for background processing
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()

    # Multiple tasks for different workers
    tasks = [
        {'queue': 'email_queue', 'data': {'order_id': order.id, 'type': 'confirmation'}},
        {'queue': 'inventory_queue', 'data': {'order_id': order.id, 'items': order.items}},
        {'queue': 'analytics_queue', 'data': {'order_id': order.id, 'amount': order.total}},
    ]

    for task in tasks:
        channel.queue_declare(queue=task['queue'], durable=True)
        channel.basic_publish(
            exchange='',
            routing_key=task['queue'],
            body=json.dumps(task['data'])
        )

    connection.close()

    # Return immediately - workers will process in background
    return {"order_id": order.id, "status": "confirmed"}
```

```python
# worker_email.py - Email worker (runs separately: python worker_email.py)
import pika
import json
from email_service import send_email

def callback(ch, method, properties, body):
    task = json.loads(body.decode())
    order = db.get_order(task['order_id'])

    # Send confirmation email (takes 2-3 seconds)
    send_email(order.customer_email, "Order Confirmation", order)

    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='email_queue', durable=True)
channel.basic_consume(queue='email_queue', on_message_callback=callback)

print('Email worker started...')
channel.start_consuming()
```

```python
# worker_inventory.py - Inventory worker (runs separately: python worker_inventory.py)
import pika
import json

def callback(ch, method, properties, body):
    task = json.loads(body.decode())

    # Update inventory (database operations)
    for item in task['items']:
        db.decrease_stock(item['product_id'], item['quantity'])

    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='inventory_queue', durable=True)
channel.basic_consume(queue='inventory_queue', on_message_callback=callback)

print('Inventory worker started...')
channel.start_consuming()
```

**How to Run This Setup:**

```bash
# Terminal 1: Start RabbitMQ
docker run -d -p 5672:5672 rabbitmq:3-management

# Terminal 2: Start your API server
uvicorn main:app --reload

# Terminal 3: Start email worker
python worker_email.py

# Terminal 4: Start inventory worker
python worker_inventory.py

# Terminal 5: Start analytics worker
python worker_analytics.py
```

**Benefits:**

- All workers share the same codebase (imports from your project)
- Easy to develop and test locally
- Workers can scale independently (run 10 email workers, 2 inventory workers)
- If one worker crashes, others keep working

#### Scenario 2: Multiple Backend Services (Microservices)

RabbitMQ excels at connecting **completely separate applications** that might be in different languages, repositories, or teams.

**Architecture:**

```text
Microservices Architecture:

┌─────────────────┐       ┌─────────────────┐
│  Order Service  │       │  Email Service  │
│   (FastAPI)     │       │   (Node.js)     │
│                 │       │                 │
│  Publishes:     │       │  Consumes:      │
│  - order.created│──────▶│  - order.created│
└─────────────────┘   │   └─────────────────┘
                      │
                RabbitMQ
                      │
┌─────────────────┐   │   ┌─────────────────┐
│Payment Service  │   │   │Shipping Service │
│   (Python)      │   │   │    (Go)         │
│                 │   │   │                 │
│  Consumes:      │   │   │  Consumes:      │
│  - order.created│◀──┴──▶│  - order.created│
└─────────────────┘       └─────────────────┘
```

**Real Example: Separate Services**

```python
# orders-service/main.py (Service 1 - Python/FastAPI)
from fastapi import FastAPI
import pika
import json

app = FastAPI()

@app.post("/orders")
async def create_order(order_data: dict):
    order = await db.create_order(order_data)

    # Publish event - other services will react
    connection = pika.BlockingConnection(
        pika.ConnectionParameters('rabbitmq.company.com')
    )
    channel = connection.channel()
    channel.exchange_declare(exchange='orders', exchange_type='fanout')

    event = {
        'event': 'order.created',
        'order_id': order.id,
        'customer_id': order.customer_id,
        'total': order.total,
        'items': order.items
    }

    channel.basic_publish(
        exchange='orders',
        routing_key='',
        body=json.dumps(event)
    )

    return {"order_id": order.id}
```

```javascript
// email-service/index.js (Service 2 - Node.js)
const amqp = require("amqplib");

async function startEmailService() {
  const connection = await amqp.connect("amqp://rabbitmq.company.com");
  const channel = await connection.createChannel();

  await channel.assertExchange("orders", "fanout");
  const q = await channel.assertQueue("", { exclusive: true });

  await channel.bindQueue(q.queue, "orders", "");

  console.log("Email service waiting for order events...");

  channel.consume(q.queue, async (msg) => {
    const event = JSON.parse(msg.content.toString());

    if (event.event === "order.created") {
      // Send email using SendGrid, Mailgun, etc.
      await sendOrderConfirmation(event.customer_id, event.order_id);
      console.log(`Sent confirmation for order ${event.order_id}`);
    }

    channel.ack(msg);
  });
}

startEmailService();
```

```go
// shipping-service/main.go (Service 3 - Go)
package main

import (
    "encoding/json"
    "log"
    "github.com/streadway/amqp"
)

type OrderEvent struct {
    Event     string `json:"event"`
    OrderID   string `json:"order_id"`
    Items     []Item `json:"items"`
}

func main() {
    conn, _ := amqp.Dial("amqp://rabbitmq.company.com")
    ch, _ := conn.Channel()

    ch.ExchangeDeclare("orders", "fanout", true, false, false, false, nil)
    q, _ := ch.QueueDeclare("", false, false, true, false, nil)

    ch.QueueBind(q.Name, "", "orders", false, nil)

    msgs, _ := ch.Consume(q.Name, "", false, false, false, false, nil)

    log.Println("Shipping service waiting for orders...")

    for msg := range msgs {
        var event OrderEvent
        json.Unmarshal(msg.Body, &event)

        if event.Event == "order.created" {
            // Create shipping label, notify warehouse, etc.
            createShippingLabel(event.OrderID, event.Items)
            log.Printf("Created shipping label for order %s", event.OrderID)
        }

        msg.Ack(false)
    }
}
```

#### Key Differences: Single vs Multiple Backends

| **Aspect**         | **Single Backend**           | **Multiple Backends**                    |
| ------------------ | ---------------------------- | ---------------------------------------- |
| **Codebase**       | Same repository              | Separate repositories                    |
| **Language**       | Same language                | Can be different languages               |
| **Communication**  | Within one application       | Between independent services             |
| **Deployment**     | Deploy together              | Deploy independently                     |
| **Use Case**       | Background jobs, task queues | Microservices, event-driven architecture |
| **Complexity**     | Simpler to start             | More complex infrastructure              |
| **Team Structure** | Single team                  | Multiple teams possible                  |

#### Which Should You Use?

**Start with Single Backend if:**

- You're building your first application with RabbitMQ
- You have one team working on the project
- You want to offload heavy tasks (emails, image processing, reports)
- You want to scale specific workers independently

**Move to Multiple Backends when:**

- You have distinct services with different responsibilities
- Different teams own different parts of the system
- Services need to be deployed independently
- Services use different programming languages
- You need true microservices architecture

#### Hybrid Approach (Most Real-World Systems)

Many applications use **both patterns**:

```text
┌────────────────────────────────────────┐
│     Main Application (Single Backend)  │
│  ┌──────────┐      ┌───────────────┐   │
│  │   API    │─────▶│ Workers (3)   │   │
│  └──────────┘      └───────────────┘   │
│       │                                │
└───────┼────────────────────────────────┘
        │
        │ (Also publishes events to other services)
        │
        ▼
    RabbitMQ
        │
        ├────▶ Notification Service (separate)
        ├────▶ Analytics Service (separate)
        └────▶ Reporting Service (separate)
```

**Example: Your API uses workers internally AND communicates with external services**

```python
@app.post("/orders")
async def create_order(order_data: dict):
    order = await db.create_order(order_data)

    # Internal workers (same backend)
    publish_to_queue('internal_email_queue', {'order_id': order.id})
    publish_to_queue('internal_inventory_queue', {'order_id': order.id})

    # External services (other backends)
    publish_event('orders_exchange', {
        'event': 'order.created',
        'order_id': order.id,
        'data': order.dict()
    })

    return {"order_id": order.id}
```

**Summary:**

- **Single Backend**: RabbitMQ connects different **processes** of the same application (API server + workers)
- **Multiple Backends**: RabbitMQ connects completely **separate applications/services**
- **You can use both patterns** depending on your needs
- **Start simple** (single backend with workers) and evolve to microservices if needed

### Key Concepts

- **Producer**: An application that sends messages
- **Consumer**: An application that receives messages
- **Queue**: A buffer that stores messages
- **Exchange**: Routes messages to queues based on routing rules
- **Binding**: Links an exchange to a queue with a routing key
- **Message**: Data sent between applications

## Use Cases for RabbitMQ

### 1. Task Queue Pattern

Distribute time-consuming tasks among multiple workers to prevent blocking the main application thread.

**Example**: Processing uploaded images, generating reports, or sending emails in the background.

### 2. Work Distribution

Balance load across multiple workers by distributing tasks evenly.

**Example**: Video encoding services where multiple workers process different video files.

### 3. Publish/Subscribe

Send messages to multiple consumers simultaneously.

**Example**: Broadcasting notifications to all connected users or microservices.

### 4. Request/Reply Pattern

Implement RPC-like functionality for microservices communication.

**Example**: A frontend service requesting data from a backend service asynchronously.

### 5. Event-Driven Architecture

Decouple services by using events to trigger actions across different parts of the system.

**Example**: E-commerce system where an order placement triggers inventory updates, payment processing, and shipping notifications.

### 6. Message Buffering

Handle traffic spikes by queuing messages when consumers are overwhelmed.

**Example**: Social media platforms handling sudden spikes in user activity.

## Installation

### Using Docker (Recommended)

The easiest way to get started with RabbitMQ is using Docker:

```bash
# Pull and run RabbitMQ with management plugin
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management
```

Access the management UI at `http://localhost:15672` (default credentials: guest/guest)

### Ubuntu/Debian

```bash
# Add RabbitMQ repository
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -

# Add repository
sudo apt-get install apt-transport-https
sudo add-apt-repository 'deb https://dl.bintray.com/rabbitmq/debian focal main'

# Update and install
sudo apt-get update
sudo apt-get install rabbitmq-server

# Enable and start service
sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server

# Enable management plugin
sudo rabbitmq-plugins enable rabbitmq_management
```

### macOS

```bash
# Using Homebrew
brew update
brew install rabbitmq

# Start RabbitMQ
brew services start rabbitmq
```

### Windows

Download the installer from [RabbitMQ official website](https://www.rabbitmq.com/download.html) and follow the installation wizard.

## Using RabbitMQ with FastAPI

### Installation of Required Packages

```bash
pip install fastapi uvicorn pika
```

### Basic Producer-Consumer Example

#### 1. Producer (Sending Messages)

Create a file `producer.py`:

```python
import pika
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class Task(BaseModel):
    task_id: str
    data: str

def get_rabbitmq_connection():
    """Establish connection to RabbitMQ"""
    try:
        credentials = pika.PlainCredentials('guest', 'guest')
        parameters = pika.ConnectionParameters(
            host='localhost',
            port=5672,
            credentials=credentials
        )
        return pika.BlockingConnection(parameters)
    except pika.exceptions.AMQPConnectionError as e:
        raise HTTPException(status_code=500, detail=f"Failed to connect to RabbitMQ: {str(e)}")

@app.post("/send-task")
async def send_task(task: Task):
    """Send a task to RabbitMQ queue"""
    connection = get_rabbitmq_connection()
    channel = connection.channel()

    # Declare queue (creates if doesn't exist)
    channel.queue_declare(queue='task_queue', durable=True)

    # Send message
    message = f"Task ID: {task.task_id}, Data: {task.data}"
    channel.basic_publish(
        exchange='',
        routing_key='task_queue',
        body=message,
        properties=pika.BasicProperties(
            delivery_mode=2,  # Make message persistent
        )
    )

    connection.close()

    return {"status": "success", "message": "Task sent to queue"}

@app.get("/")
async def root():
    return {"message": "RabbitMQ Producer API"}
```

#### 2. Consumer (Receiving Messages)

Create a file `consumer.py`:

```python
import pika
import time

def callback(ch, method, properties, body):
    """Process received message"""
    print(f" [x] Received {body.decode()}")

    # Simulate work
    time.sleep(2)

    print(f" [x] Done processing")

    # Acknowledge message
    ch.basic_ack(delivery_tag=method.delivery_tag)

def main():
    """Start consuming messages"""
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters(
        host='localhost',
        port=5672,
        credentials=credentials
    )

    connection = pika.BlockingConnection(parameters)
    channel = connection.channel()

    # Declare queue
    channel.queue_declare(queue='task_queue', durable=True)

    # Set prefetch count
    channel.basic_qos(prefetch_count=1)

    # Start consuming
    channel.basic_consume(
        queue='task_queue',
        on_message_callback=callback
    )

    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':
    main()
```

#### Running the Basic Producer-Consumer Example

**Prerequisites:**

- Ensure RabbitMQ is running (see Installation section above)
- Install required packages: `pip install fastapi uvicorn pika`

**Step 1: Start RabbitMQ**

```bash
# If using Docker
docker start rabbitmq

# Or if installed locally, RabbitMQ should already be running
```

**Step 2: Start the Consumer (in Terminal 1)**

```bash
python consumer.py
```

You should see: `[*] Waiting for messages. To exit press CTRL+C`

**Step 3: Start the FastAPI Producer (in Terminal 2)**

```bash
uvicorn producer:app --reload
```

The API will start at `http://localhost:8000`

**Step 4: Send a Test Message (in Terminal 3)**

Using curl:

```bash
curl -X POST "http://localhost:8000/send-task" \
  -H "Content-Type: application/json" \
  -d '{"task_id": "123", "data": "Process this task"}'
```

Or using Python requests:

```python
import requests

response = requests.post(
    "http://localhost:8000/send-task",
    json={"task_id": "123", "data": "Process this task"}
)
print(response.json())
```

Or visit `http://localhost:8000/docs` to use the interactive API documentation.

**Expected Output:**

- Terminal 2 (Producer): `{"status": "success", "message": "Task sent to queue"}`
- Terminal 1 (Consumer): Will show the received message and processing confirmation

### Advanced Example: Publish/Subscribe Pattern

#### Publisher with FastAPI

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import pika
import json

app = FastAPI()

class Notification(BaseModel):
    user_id: str
    message: str
    type: str

@app.post("/publish-notification")
async def publish_notification(notification: Notification):
    """Publish notification to all subscribers"""
    try:
        credentials = pika.PlainCredentials('guest', 'guest')
        parameters = pika.ConnectionParameters(host='localhost', credentials=credentials)
        connection = pika.BlockingConnection(parameters)
        channel = connection.channel()

        # Declare fanout exchange
        channel.exchange_declare(exchange='notifications', exchange_type='fanout')

        # Publish message
        message = json.dumps(notification.dict())
        channel.basic_publish(
            exchange='notifications',
            routing_key='',
            body=message
        )

        connection.close()

        return {"status": "success", "message": "Notification published"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

#### Subscriber

```python
import pika
import json

def callback(ch, method, properties, body):
    notification = json.loads(body.decode())
    print(f" [x] Received notification: {notification}")
    # Process notification (send email, SMS, push notification, etc.)

def main():
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters(host='localhost', credentials=credentials)
    connection = pika.BlockingConnection(parameters)
    channel = connection.channel()

    # Declare exchange
    channel.exchange_declare(exchange='notifications', exchange_type='fanout')

    # Declare exclusive queue for this consumer
    result = channel.queue_declare(queue='', exclusive=True)
    queue_name = result.method.queue

    # Bind queue to exchange
    channel.queue_bind(exchange='notifications', queue=queue_name)

    channel.basic_consume(
        queue=queue_name,
        on_message_callback=callback,
        auto_ack=True
    )

    print(' [*] Waiting for notifications. To exit press CTRL+C')
    channel.start_consuming()

if __name__ == '__main__':
    main()
```

#### Running the Publish/Subscribe Example

This example demonstrates how to broadcast messages to multiple subscribers simultaneously.

**Prerequisites:**

- Ensure RabbitMQ is running
- Install required packages: `pip install fastapi uvicorn pika`

##### Step 1: Start RabbitMQ

```bash
# If using Docker
docker start rabbitmq
```

##### Step 2: Create the Files

Save the publisher code as `publisher.py` and the subscriber code as `subscriber.py`

##### Step 3: Start Multiple Subscribers (in separate terminals)

Terminal 1:

```bash
python subscriber.py
```

Terminal 2 (optional - to see fanout in action):

```bash
python subscriber.py
```

Terminal 3 (optional - add as many subscribers as you want):

```bash
python subscriber.py
```

Each subscriber should show: `[*] Waiting for notifications. To exit press CTRL+C`

##### Step 4: Start the Publisher API (in a new terminal)

```bash
uvicorn publisher:app --reload
```

##### Step 5: Publish a Notification

Using curl:

```bash
curl -X POST "http://localhost:8000/publish-notification" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_123",
    "message": "Hello, World!",
    "type": "info"
  }'
```

Or using Python:

```python
import requests

notification = {
    "user_id": "user_123",
    "message": "Hello, World!",
    "type": "info"
}

response = requests.post(
    "http://localhost:8000/publish-notification",
    json=notification
)
print(response.json())
```

**Expected Output:**

- Publisher returns: `{"status": "success", "message": "Notification published"}`
- ALL running subscribers will receive and display the notification simultaneously
- This demonstrates the fanout exchange pattern where one message reaches all subscribers

### Connection Pool Pattern (Production-Ready)

For production use, implement connection pooling:

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
import pika
from pika.adapters.blocking_connection import BlockingChannel

class RabbitMQClient:
    def __init__(self):
        self.connection = None
        self.channel = None

    def connect(self):
        credentials = pika.PlainCredentials('guest', 'guest')
        parameters = pika.ConnectionParameters(
            host='localhost',
            port=5672,
            credentials=credentials,
            heartbeat=600,
            blocked_connection_timeout=300
        )
        self.connection = pika.BlockingConnection(parameters)
        self.channel = self.connection.channel()

    def close(self):
        if self.connection and not self.connection.is_closed:
            self.connection.close()

rabbitmq_client = RabbitMQClient()

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    rabbitmq_client.connect()
    yield
    # Shutdown
    rabbitmq_client.close()

app = FastAPI(lifespan=lifespan)

@app.post("/send-message")
async def send_message(message: str):
    rabbitmq_client.channel.queue_declare(queue='messages', durable=True)
    rabbitmq_client.channel.basic_publish(
        exchange='',
        routing_key='messages',
        body=message,
        properties=pika.BasicProperties(delivery_mode=2)
    )
    return {"status": "sent"}
```

#### Running the Connection Pool Example

This example shows a production-ready pattern with proper connection lifecycle management.

**Prerequisites:**

- Ensure RabbitMQ is running
- Install required packages: `pip install fastapi uvicorn pika`

##### Step 1: Start RabbitMQ

```bash
# If using Docker
docker start rabbitmq
```

##### Step 2: Create and Run the Application

Save the connection pool code as `app_pooled.py` and run it:

```bash
uvicorn app_pooled:app --reload
```

##### Step 3: Test the Application

Using curl:

```bash
curl -X POST "http://localhost:8000/send-message?message=Hello%20from%20pooled%20connection"
```

Or using Python:

```python
import requests

response = requests.post(
    "http://localhost:8000/send-message",
    params={"message": "Hello from pooled connection"}
)
print(response.json())
```

##### Step 4: Consume Messages

Create a simple consumer to receive the messages (save as `consumer_messages.py`):

```python
import pika

def callback(ch, method, properties, body):
    print(f"Received: {body.decode()}")
    ch.basic_ack(delivery_tag=method.delivery_tag)

credentials = pika.PlainCredentials('guest', 'guest')
connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost', credentials=credentials)
)
channel = connection.channel()
channel.queue_declare(queue='messages', durable=True)
channel.basic_consume(queue='messages', on_message_callback=callback)

print('Waiting for messages...')
channel.start_consuming()
```

Run the consumer:

```bash
python consumer_messages.py
```

**Benefits of Connection Pool Pattern:**

- Connection is established once during app startup
- Reused across all requests (better performance)
- Properly closed during app shutdown
- More suitable for production environments

## Best Practices

### 1. Message Acknowledgment

Always acknowledge messages after processing to prevent message loss:

```python
channel.basic_ack(delivery_tag=method.delivery_tag)
```

### 2. Message Persistence

Make messages persistent by setting delivery_mode=2:

```python
properties=pika.BasicProperties(delivery_mode=2)
```

### 3. Prefetch Count

Limit the number of unacknowledged messages per consumer:

```python
channel.basic_qos(prefetch_count=1)
```

### 4. Error Handling

Implement proper error handling and dead letter exchanges for failed messages:

```python
channel.queue_declare(
    queue='task_queue',
    durable=True,
    arguments={
        'x-dead-letter-exchange': 'dlx',
        'x-message-ttl': 60000  # 60 seconds
    }
)
```

### 5. Connection Management

Reuse connections and channels; create new ones only when necessary.

### 6. Monitoring

Use the RabbitMQ management UI to monitor queue depth, message rates, and consumer status.

## Common Pitfalls to Avoid

1. **Not acknowledging messages**: Can lead to memory issues
2. **Blocking operations in callbacks**: Use thread pools for CPU-intensive tasks
3. **Not handling connection failures**: Implement reconnection logic
4. **Ignoring queue depth**: Monitor and alert on growing queues
5. **Hardcoded credentials**: Use environment variables

## Conclusion

RabbitMQ is a robust message broker that enables building scalable, distributed systems. When combined with FastAPI, it provides a powerful foundation for building asynchronous microservices. Whether you're implementing background task processing, event-driven architectures, or load distribution, RabbitMQ offers the reliability and features needed for production systems.

Start with simple producer-consumer patterns and gradually adopt more complex patterns like publish/subscribe or topic exchanges as your application grows. Remember to follow best practices around message acknowledgment, persistence, and error handling to build resilient systems.

## Additional Resources

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [Pika Documentation](https://pika.readthedocs.io/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [AMQP Protocol Specification](https://www.amqp.org/)

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
