# Mastering Docker: Essential Commands & Practical Examples

Docker has revolutionized how developers build, test, and deploy applications by containerizing code into lightweight, portable packages. Whether you're a beginner or an experienced developer, understanding Docker's core concepts and commands can streamline your workflow. In this blog, we’ll explore **essential Docker commands**, create a practical example of hosting a **Next.js application**, and demonstrate how to set up a **PostgreSQL database** using Docker.

---

## 🛠️ What is Docker?

Docker is a platform that packages applications with their dependencies into **containers**. Containers are isolated environments that run consistently across different systems, eliminating "it works on my machine" issues.

---

## 🔧 Top Docker Commands You Need to Know

1. **`docker run [IMAGE]`**  
   Launches a container from an existing image.

   ```bash
   docker run hello-world
   ```

2. **`docker images`**  
   Lists all Docker images on your system.

   ```bash
   docker images
   ```

3. **`docker ps`**  
   Displays running containers.

   ```bash final
   docker ps
   ```

4. **`docker build [OPTIONS] [PATH]`**  
   Builds a Docker image from a `Dockerfile`.

   ```bash
   docker build -t my-next-app .
   ```

5. **`docker-compose up`**  
   Orchestrate multi-container apps (e.g., a Next.js app + PostgreSQL).

6. **`docker stop [CONTAINER_ID]`**  
   Stops a running container.

7. **`docker rm [CONTAINER_ID]`**  
   Removes a stopped container.

8. **`docker rmi [IMAGE_ID]`**  
   Removes an unused image.

---

## 🧱 Example 1: Hosting a Next.js Application with Docker

### Step 1: Create a Simple Next.js App

Start by creating a basic Next.js project:

```bash
npx create-next-app@latest my-next-app
cd my-next-app
```

Update `pages/index.js` to display a simple message:

```js
export default function Home() {
  return <h1>Hello from Next.js!</h1>;
}
```

### Step 2: Build a Docker Image

Create a `Dockerfile` in your project root:

```Bash
# Use the official Node.js image as a parent
FROM node:18

# Create workspace directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose port for Next.js (3000)
EXPOSE 3000

# Command to run the app
CMD ["npm", "run", "dev"]
```

### Step 3: Run the Container

Build and run your Docker image:

```bash
docker build -t my-next-app .
docker run -d -p 3000:3000 my-next-app
```

**Verify**: Access `http://localhost:3000` in your browser to see the app!

---

## 🐢 Example 2: Hosting PostgreSQL with Docker

### Step 1: Use the Official PostgreSQL Image

Run a PostgreSQL container using `docker run`:

```bash
docker run --name my-postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=123456 \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -d postgres
```

**Explanation**:

- `--name`: Assigns a name to the container.
- `-e`: Sets environment variables for PostgreSQL credentials.
- `-p`: Maps the container’s port 5432 to your host’s port.
- `-d`: Runs the container in detached mode.

### Step 2: Connect to PostgreSQL

Use `psql` from the container shell to test the connection:

```bash
docker exec -it my-postgres psql -U admin -d mydb
```

**Tip**: To persist data, use a volume:

```bash
docker run --name my-postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=123456 \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v /path/to/host/data:/var/lib/postgresql/data \
  -d postgres
```

---

## 🚀 Docker Compose Example: Next.js + PostgreSQL

Use `docker-compose.yml` to run both services together:

```yaml
version: "3"
services:
  nextjs:
    build: ./next-app
    ports:
      - "3000:3000"
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: 123456
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Run with:

```bash
docker-compose up -d
```

---

## 📚 Resources to Deepen Your Docker Knowledge

1. [Docker Official Documentation](https://docs.docker.com/)
2. [Docker Hub (Images/Registry)](https://hub.docker.com/)
3. [Docker Compose Guide](https://docs.docker.com/compose/)
4. [PostgreSQL Docker Image Docs](https://hub.docker.com/_images/postgres)

---

## 📌 Final Thoughts

Docker simplifies development and deployment workflows by standardizing environments. Whether you're building a static site with Next.js or setting up a PostgreSQL database, Docker's flexibility ensures your app runs consistently everywhere. Start small with these examples and expand into more complex projects!

**Happy containerizing! 🎉**
