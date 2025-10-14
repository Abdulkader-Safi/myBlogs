# Building a Modern Text-to-Speech Application with AI-Powered Text Optimization

## Introduction

In an era where accessibility and content consumption are evolving rapidly, text-to-speech (TTS) technology has become increasingly important. Whether you're creating audiobooks, accessibility tools, or voice assistants, high-quality speech synthesis is essential. However, converting technical documentation, code snippets, and markdown-formatted content into natural-sounding speech presents unique challenges.

I recently built a modern **Text-to-Speech application** that addresses these challenges head-on by combining **Coqui TTS** for high-quality speech synthesis with **Ollama's LLM capabilities** for intelligent text preprocessing. The result is a full-stack application that can handle everything from simple text to complex technical documentation with code blocks, markdown formatting, and technical terminology.

In this article, I'll walk you through the architecture, implementation details, and key learnings from building this project.

**GitHub Repository:** [https://github.com/Abdulkader-Safi/TTS](https://github.com/Abdulkader-Safi/TTS)

---

## The Problem: Making Technical Content Speech-Friendly

Traditional TTS systems struggle with technical content. Consider these examples:

- **Technical Terms:** "Pyenv" sounds awkward when read literally. It should be pronounced as "Pie env"
- **Code Blocks:** Raw code with backticks and syntax markers creates unnatural speech
- **Markdown Syntax:** Headers like `## Section Title` are read as "hashtag hashtag Section Title"
- **Environment Variables:** `$PATH` gets mispronounced instead of being read as "PATH"
- **Emojis:** Symbols like ✅ 🚀 💻 interrupt the natural flow of speech

The challenge was to create a system that could intelligently transform this technical content into speech-friendly text while maintaining the original meaning and context.

---

## Architecture Overview

The application follows a modern microservices architecture with three main components:

```
┌─────────────────────────────────────────────────────────────┐
│                      Client Browser                         │
│                  (Vue.js + Tailwind CSS)                    │
└─────────────────────┬───────────────────────────────────────┘
                      │ HTTP/REST
                      ├──────────────────────────────────────┐
                      ↓                                      ↓
┌─────────────────────────────────┐    ┌──────────────────────────────┐
│     FastAPI Backend             │    │    Ollama Service            │
│  - TTS Synthesis (Coqui)       │←───┤  - Text Optimization (LLM)   │
│  - Audio Format Conversion      │    │  - llama3.2 Model           │
│  - API Endpoints                │    │  - Host: localhost:11434    │
└─────────────────────────────────┘    └──────────────────────────────┘
                      │
                      ↓
            ┌─────────────────┐
            │ Docker Compose  │
            │  - Networking   │
            │  - Volumes      │
            └─────────────────┘
```

### Key Architectural Decisions

1. **Separation of Concerns:** Frontend handles UI/UX, backend manages TTS processing, Ollama provides AI capabilities
2. **RESTful API Design:** Clean, documented endpoints using FastAPI's automatic OpenAPI generation
3. **Stateless Operations:** Each request is independent, enabling horizontal scaling
4. **Docker-First Approach:** Containerized deployment for consistency across environments

---

## Core Technologies

### Backend Stack

- **FastAPI** (Python 3.11+): Modern, fast web framework with automatic API documentation
- **Coqui TTS**: High-quality, open-source text-to-speech library with multiple model support
- **Ollama**: Local LLM runtime for intelligent text preprocessing (using llama3.2)
- **FFmpeg**: Audio format conversion (WAV to MP3)
- **Uvicorn**: Lightning-fast ASGI server

### Frontend Stack

- **Vue.js 3**: Progressive JavaScript framework with Composition API
- **TypeScript**: Type-safe development experience
- **Tailwind CSS**: Utility-first CSS framework for rapid UI development
- **Vite**: Next-generation frontend tooling

### Infrastructure

- **Docker & Docker Compose**: Containerization and orchestration
- **Nginx**: Production-grade web server for frontend deployment

---

## Building the Backend with FastAPI

FastAPI was chosen for its excellent performance, automatic API documentation, and modern Python features. Here's the core application setup:

### Application Initialization

```python
"""Main application entry point"""

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from app.config import (
    API_TITLE,
    API_VERSION,
    CORS_ORIGINS,
    DEFAULT_MODEL,
)
from app.routes.tts_routes import router as tts_router
from app.routes.models_routes import router as models_router
from app.services.tts_service import tts_service


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Lifespan event handler for startup and shutdown"""
    # Startup: Initialize TTS model
    tts_service.initialize(DEFAULT_MODEL)
    yield
    # Shutdown: cleanup if needed
    print("Shutting down TTS API...")


# Initialize FastAPI app
app = FastAPI(title=API_TITLE, version=API_VERSION, lifespan=lifespan)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(tts_router, tags=["TTS"])
app.include_router(models_router, tags=["Models"])


@app.get("/")
async def root():
    """Health check endpoint"""
    return {
        "message": "TTS API is running",
        "version": API_VERSION,
        "current_model": tts_service.get_current_model(),
    }
```

**Key Features:**

- **Lifespan Management:** TTS models are loaded once at startup, reducing per-request latency
- **CORS Configuration:** Enables cross-origin requests for the frontend
- **Automatic Documentation:** FastAPI generates interactive API docs at `/docs`
- **Health Check Endpoint:** Docker healthcheck integration for container orchestration

### TTS Synthesis Endpoint

The main synthesis endpoint handles text validation, model switching, and audio generation:

```python
@router.post("/synthesize", response_model=TTSResponse)
async def synthesize_speech(request: TTSRequest):
    """
    Synthesize speech from text
    """
    # Validate text is not empty
    if not request.text.strip():
        raise HTTPException(status_code=400, detail="Text cannot be empty")

    # Validate text length
    text_length = len(request.text.strip())

    # Minimum length check (TTS models need minimum characters to prevent kernel errors)
    if text_length < MIN_TEXT_LENGTH:
        raise HTTPException(
            status_code=400,
            detail=f"Text is too short. Please provide at least {MIN_TEXT_LENGTH} characters."
        )

    # Maximum length check
    if text_length > MAX_TEXT_LENGTH:
        raise HTTPException(
            status_code=400,
            detail=f"Text is too long ({text_length:,} characters). Maximum allowed is {MAX_TEXT_LENGTH:,} characters."
        )

    try:
        # Switch model if different from current
        tts_service.switch_model(request.model_name)

        # Generate unique filename
        file_id = str(uuid.uuid4())
        output_format = request.format.lower()
        output_file = OUTPUT_DIR / f"{file_id}.wav"

        # Generate speech
        tts_service.synthesize(text=request.text, output_path=str(output_file))

        # Convert to mp3 if requested
        if output_format == "mp3":
            mp3_file = OUTPUT_DIR / f"{file_id}.mp3"
            os.system(
                f"ffmpeg -i {output_file} -codec:a libmp3lame -qscale:a 2 {mp3_file} -y -loglevel quiet"
            )
            os.remove(output_file)
            output_file = mp3_file

        return TTSResponse(
            success=True,
            file_id=file_id,
            download_url=f"/download/{file_id}.{output_format}",
            format=output_format,
        )

    except Exception as e:
        raise HTTPException(
            status_code=500, detail=f"Error generating speech: {str(e)}"
        )
```

**Smart Text Validation:**

- **Minimum Length (15 characters):** Prevents TTS kernel errors with very short inputs
- **Maximum Length (20,000 characters):** Ensures reliable processing
- **Format Support:** WAV for quality, MP3 for smaller file sizes

---

## Integrating Ollama for Intelligent Text Processing

The standout feature of this application is the integration with Ollama for AI-powered text optimization. This is what sets it apart from traditional TTS systems.

### Ollama Service Architecture

````python
"""Ollama service for intelligent text optimization for TTS"""

import ollama
from typing import Optional


class OllamaService:
    """Service class for optimizing text using Ollama LLM"""

    def __init__(self, model_name: str = "llama3.2", host: str = "http://localhost:11434"):
        self.model_name = model_name
        self.host = host
        # Configure Ollama client with custom host
        self.client = ollama.Client(host=self.host)
        self.system_prompt = """You are a text-to-speech converter. Convert ALL markdown syntax to natural spoken text.

MANDATORY: Apply ALL rules to ENTIRE text. NO exceptions. NO skipping sections.

CRITICAL: Output ONLY converted text. NO explanations. NO meta-commentary.

RULES - APPLY TO EVERY LINE:

1. EMOJIS - DELETE ALL: ✅ ✓ ⚙️ 💡 🧠 🧩 🚀 💻 🔧 ⚡ 📦 🎯

2. MARKDOWN SYNTAX - ALWAYS CONVERT:
   # Text → "Heading: Text"
   ## Text → "Section: Text"
   ### Text → "Subsection: Text"
   **bold** → remove stars, keep text
   *italic* → remove stars, keep text
   --- → "Section break" or remove
   `code` → remove backticks

3. CODE BLOCKS - ALWAYS ANNOUNCE:
   ```language
   code here

→ "Here's the code in language: [read code naturally]"

4. TECHNICAL TERMS (only these specific terms):
    "Pyenv" or "pyenv" → "Pie env"
    "macOS" → "mac O S"
    "GitHub" → "Git Hub"
    "API" → "A P I"
    "JavaScript" → "Java Script"
    "TypeScript" → "Type Script"

5. ENVIRONMENT VARIABLES:
    $PATH → "PATH"
   $HOME → "HOME"
    $ at start of line in code → "Run in terminal:"

6. JSX/XML TAGS - READ NATURALLY:
    <Activity /> → "Activity component"
    <ChatPanel /> → "Chat Panel component"
    """

        def optimize_text(self, text: str) -> str:
            """
            Optimize text for TTS using Ollama

            Args:
                text: Input text to optimize

            Returns:
                Optimized text suitable for TTS
            """
            try:
                response = self.client.chat(
                    model=self.model_name,
                    messages=[
                        {"role": "system", "content": self.system_prompt},
                        {
                            "role": "user",
                            "content": f"Convert this to speech-friendly text. Output ONLY the converted text:\n\n{text}",
                        },
                    ],
                )

                optimized_text = response["message"]["content"].strip()

                # Remove meta-commentary that LLM might add
                # [Post-processing logic here]

                return optimized_text

            except Exception as e:
                print(f"Error optimizing text with Ollama: {e}")
                # Fallback to original text if Ollama fails
                return text

        def is_available(self) -> bool:
            """Check if Ollama service is available"""
            try:
                self.client.list()
                return True
            except Exception:
                return False

````

### The Optimization API Endpoint

```python
@router.post("/optimize", response_model=OptimizeResponse)
async def optimize_text(request: OptimizeRequest):
    """
    Optimize text for TTS using Ollama
    """
    # Validate text is not empty
    if not request.text.strip():
        raise HTTPException(status_code=400, detail="Text cannot be empty")

    # Get Ollama service instance
    ollama_service = get_ollama_service()

    # Check if Ollama is available
    if not ollama_service.is_available():
        raise HTTPException(
            status_code=503,
            detail="Ollama service is not available. Please ensure Ollama is running.",
        )

    try:
        original_text = request.text.strip()
        optimized_text = ollama_service.optimize_text(original_text)

        return OptimizeResponse(
            success=True,
            optimized_text=optimized_text,
            original_length=len(original_text),
            optimized_length=len(optimized_text),
        )

    except Exception as e:
        raise HTTPException(
            status_code=500, detail=f"Error optimizing text: {str(e)}"
        )
```

### Why This Approach Works

1. **Prompt Engineering:** The system prompt is carefully crafted to produce consistent, TTS-friendly output
2. **Graceful Degradation:** If Ollama is unavailable, the system falls back to the original text
3. **Post-Processing:** Removes common LLM artifacts like explanatory text
4. **Flexibility:** Easy to add new transformation rules by updating the system prompt

---

## Frontend Development with Vue.js

The frontend provides an intuitive interface for text input, optimization, and audio generation. Built with Vue 3's Composition API and TypeScript for type safety.

### Main Application Component

```typescript
<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import TextInput from "./components/TextInput.vue";
import FormatSelector from "./components/FormatSelector.vue";
import ModelSelector from "./components/ModelSelector.vue";
import OptimizeButton from "./components/OptimizeButton.vue";
import GenerateSpeechButton from "./components/GenerateSpeechButton.vue";
import AudioPlayer from "./components/AudioPlayer.vue";
import OllamaStatus from "./components/OllamaStatus.vue";

const API_URL = import.meta.env.VITE_API_URL || "http://localhost:8888";

const text = ref("");
const optimizedText = ref("");
const format = ref<"wav" | "mp3">("wav");
const model = ref("tts_models/en/ljspeech/vits");
const isOptimizing = ref(false);
const isGenerating = ref(false);
const audioUrl = ref<string | null>(null);

const isTextValid = computed(() => text.value.trim().length >= 15);
const hasOptimizedText = computed(() => optimizedText.value.trim().length > 0);

// Fetch available models on mount
onMounted(async () => {
    try {
        const response = await fetch(`${API_URL}/models`);
        if (response.ok) {
            const data = await response.json();
            availableModels.value = data.models || [];
            recommendedModels.value = data.recommended || [];
        }
    } catch (err) {
        console.error("Failed to fetch models:", err);
    }
});

const optimizeTextWithOllama = async () => {
    if (!isTextValid.value) {
        error.value = "Please enter at least 15 characters";
        return;
    }

    isOptimizing.value = true;

    try {
        const response = await fetch(`${API_URL}/optimize`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ text: text.value }),
        });

        const data = await response.json();
        optimizedText.value = data.optimized_text;
    } catch (err) {
        error.value = err instanceof Error ? err.message : "An error occurred";
    } finally {
        isOptimizing.value = false;
    }
};
</script>
```

### UI/UX Features

1. **Real-time Character Counter:** Color-coded feedback (green, orange, red) based on text length
2. **Component Architecture:** Modular, reusable Vue components
3. **Loading States:** Visual feedback during optimization and synthesis
4. **Error Handling:** User-friendly error messages
5. **Responsive Design:** Works seamlessly on desktop and mobile with Tailwind CSS

---

## Docker Deployment Strategy

The application is fully containerized for consistent deployment across any environment.

### Docker Compose Configuration

```yaml
version: "3.8"

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: tts-backend
    ports:
      - "8888:8888"
    volumes:
      - ./backend/output:/app/output
    environment:
      - PYTHONUNBUFFERED=1
      - OLLAMA_HOST=http://host.docker.internal:11434
    extra_hosts:
      - "host.docker.internal:host-gateway"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8888/"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s
    restart: unless-stopped
    networks:
      - tts-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        - VITE_API_URL=http://localhost:8888
    container_name: tts-frontend
    ports:
      - "3333:80"
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - tts-network

networks:
  tts-network:
    driver: bridge
```

### Key Docker Features

1. **Health Checks:** Ensures backend is ready before starting frontend
2. **Host Network Access:** Backend can communicate with Ollama running on host machine
3. **Volume Mounts:** Persistent storage for generated audio files
4. **Environment Configuration:** Flexible configuration via environment variables
5. **Network Isolation:** Services communicate through dedicated bridge network

### Quick Start

```bash
# Clone the repository
git clone https://github.com/Abdulkader-Safi/TTS.git
cd TTS

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Access the application
# Frontend: http://localhost:3333
# Backend API: http://localhost:8888
# API Docs: http://localhost:8888/docs
```

---

## Key Features and Implementation Details

### 1. Dynamic Model Switching

The application supports multiple TTS models with dynamic switching:

- **tts_models/en/ljspeech/vits** (Recommended): Best overall quality, natural-sounding
- **tts_models/en/vctk/vits**: Multi-speaker support with 109 voices
- **tts_models/en/ljspeech/neural_hmm**: Very natural prosody
- **tts_models/en/ljspeech/tacotron2-DCA**: Good quality, no espeak-ng required

Users can switch models on-the-fly without restarting the application.

### 2. Smart Text Validation

The system enforces intelligent text length validation:

- **Minimum: 15 characters** - Prevents TTS kernel errors
- **Recommended: 50-5,000 characters** - Optimal quality range
- **Maximum: 20,000 characters** - Technical API limit

Real-time UI feedback helps users stay within optimal ranges.

### 3. Multiple Audio Formats

Supports both WAV (high quality) and MP3 (compressed) output formats using FFmpeg for conversion.

### 4. Ollama Status Monitoring

The frontend displays real-time Ollama service status, helping users understand when AI optimization is available.

### 5. Graceful Error Handling

Comprehensive error handling at every layer:

- Input validation with clear error messages
- Service availability checks
- Fallback mechanisms when services are unavailable
- User-friendly error presentation in the UI

---

## Challenges and Solutions

### Challenge 1: TTS Model Loading Time

**Problem:** TTS models take 60-120 seconds to load, causing container health checks to fail.

**Solution:**

- Implemented startup health check delay (120 seconds)
- Load models during application lifespan initialization
- Added comprehensive logging for debugging

### Challenge 2: LLM Output Consistency

**Problem:** Ollama sometimes adds meta-commentary or explanations to the converted text.

**Solution:**

- Strict system prompt with explicit instructions
- Post-processing to remove common meta-commentary phrases
- Multiple iterations of prompt engineering

### Challenge 3: Docker Host Communication

**Problem:** Backend container needs to communicate with Ollama running on the host machine.

**Solution:**

- Used `host.docker.internal` with `extra_hosts` configuration
- Configurable Ollama host via environment variables
- Graceful degradation when Ollama is unavailable

### Challenge 4: Text Length Validation

**Problem:** Very short texts cause TTS kernel errors; very long texts cause processing issues.

**Solution:**

- Implemented min/max length validation (15-20,000 characters)
- Color-coded real-time character counter in UI
- Helpful error messages guiding users to optimal text length

---

## Performance Optimization

### Backend Optimizations

1. **Model Preloading:** TTS models loaded once at startup, not per request
2. **Async Operations:** FastAPI's async capabilities for non-blocking I/O
3. **Efficient File Handling:** UUID-based unique filenames prevent collisions
4. **Resource Cleanup:** Automatic cleanup endpoints for generated audio files

### Frontend Optimizations

1. **Vite Build Tool:** Lightning-fast development and optimized production builds
2. **Component Lazy Loading:** Code-splitting for faster initial load
3. **Computed Properties:** Efficient reactivity with Vue 3's Composition API
4. **Minimal Dependencies:** Lean dependency tree reduces bundle size

### Infrastructure Optimizations

1. **Docker Multi-stage Builds:** Smaller production images
2. **Nginx for Static Assets:** High-performance static file serving
3. **Health Checks:** Ensure services are ready before accepting traffic
4. **Restart Policies:** Automatic recovery from failures

---

## Future Enhancements

### Planned Features

1. **Speaker Selection:** Voice customization for multi-speaker models
2. **Batch Processing:** Generate audio for multiple texts simultaneously
3. **Voice Cloning:** Custom voice creation from audio samples
4. **Cloud Storage Integration:** S3/GCS for generated audio files
5. **WebSocket Support:** Real-time progress updates during generation
6. **Advanced Text Preprocessing:**
   - URL detection and conversion
   - Table-to-speech formatting
   - Math equation pronunciation
7. **API Rate Limiting:** Prevent abuse and ensure fair usage
8. **User Authentication:** Multi-user support with personal settings
9. **Audio Post-Processing:**
   - Speed adjustment
   - Pitch modification
   - Background music mixing

### Technical Improvements

1. **Caching Layer:** Redis for frequently requested text-to-speech conversions
2. **Job Queue:** Celery for background audio generation
3. **Monitoring & Logging:** Prometheus + Grafana for observability
4. **CDN Integration:** Fast global audio delivery
5. **A/B Testing Framework:** Compare different optimization strategies

---

## Conclusion

Building this Text-to-Speech application with AI-powered text optimization has been an incredible learning experience. The project demonstrates how combining modern web technologies (FastAPI, Vue.js) with AI capabilities (Ollama/LLM) can solve real-world problems in creative ways.

### Key Takeaways

1. **AI Integration is Powerful:** LLMs can intelligently transform content in ways traditional rule-based systems cannot
2. **Prompt Engineering Matters:** Careful prompt design is critical for consistent LLM output
3. **User Experience is King:** Real-time validation and feedback create a better user experience
4. **Docker Simplifies Deployment:** Containerization makes complex multi-service applications manageable
5. **Open Source is Viable:** Quality TTS without expensive proprietary solutions

### Technical Achievements

- Built a production-ready full-stack application with modern best practices
- Successfully integrated local LLM (Ollama) with traditional ML models (Coqui TTS)
- Implemented comprehensive error handling and graceful degradation
- Created an intuitive, responsive user interface
- Achieved fully containerized deployment with Docker Compose

### What I Learned

- **FastAPI Lifespan Events:** Essential for expensive startup operations like model loading
- **Prompt Engineering:** Iterative refinement is necessary for reliable LLM output
- **Docker Networking:** `host.docker.internal` enables container-to-host communication
- **Vue 3 Composition API:** More flexible and type-safe than Options API
- **TTS Model Characteristics:** Different models have different strengths and requirements

### Try It Yourself

The complete source code is available on GitHub: [https://github.com/Abdulkader-Safi/TTS](https://github.com/Abdulkader-Safi/TTS)

```bash
# Quick start with Docker
git clone https://github.com/Abdulkader-Safi/TTS.git
cd TTS
docker-compose up -d

# Open http://localhost:3333 in your browser
```

---

## Additional Resources

- **Coqui TTS Documentation:** [https://github.com/coqui-ai/TTS](https://github.com/coqui-ai/TTS)
- **FastAPI Documentation:** [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- **Ollama Documentation:** [https://ollama.com/](https://ollama.com/)
- **Vue.js 3 Guide:** [https://vuejs.org/](https://vuejs.org/)
- **Tailwind CSS:** [https://tailwindcss.com/](https://tailwindcss.com/)

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
