# How to Reduce Docker Image Size from 1.2GB to 10MB: A Complete Optimization Guide

## Table of Contents

- [Introduction: Why Docker Image Size Matters in 2025](#introduction-why-docker-image-size-matters-in-2025)
- [Understanding the Impact: Why Every Megabyte Counts](#understanding-the-impact-why-every-megabyte-counts)
  - [Storage Cost Implications](#storage-cost-implications)
  - [Deployment Speed and Scalability](#deployment-speed-and-scalability)
  - [Security Considerations](#security-considerations)
- [Technique 1: Start With Minimal Base Images](#technique-1-start-with-minimal-base-images)
  - [The Problem: Bloated Base Images](#the-problem-bloated-base-images)
  - [The Solution: Alpine Linux Images](#the-solution-alpine-linux-images)
  - [Understanding Alpine Linux](#understanding-alpine-linux)
  - [Compatibility Considerations](#compatibility-considerations)
  - [Alternative: Google Distroless Images](#alternative-google-distroless-images)
- [Technique 2: Optimize Layer Caching for Fast Rebuilds](#technique-2-optimize-layer-caching-for-fast-rebuilds)
  - [How Docker Layer Caching Works](#how-docker-layer-caching-works)
  - [The Problem: Cache Invalidation](#the-problem-cache-invalidation)
  - [The Solution: Strategic Layer Ordering](#the-solution-strategic-layer-ordering)
  - [Key Caching Principles](#key-caching-principles)
  - [Language-Specific Examples](#language-specific-examples)
- [Technique 3: Leverage .dockerignore Files](#technique-3-leverage-dockerignore-files)
  - [Understanding Build Context](#understanding-build-context)
  - [Creating an Effective .dockerignore](#creating-an-effective-dockerignore)
  - [Impact Measurement](#impact-measurement)
  - [Security Benefits](#security-benefits)
- [Technique 4: Master Layer Squashing and Immutability](#technique-4-master-layer-squashing-and-immutability)
  - [The Layer Immutability Problem](#the-layer-immutability-problem)
  - [Why Deletions Don't Save Space](#why-deletions-dont-save-space)
  - [The Solution: Combine Operations](#the-solution-combine-operations)
  - [Security Implications of Layer Immutability](#security-implications-of-layer-immutability)
  - [Best Practices for Layer Optimization](#best-practices-for-layer-optimization)
- [Technique 5: Implement Multi-Stage Builds](#technique-5-implement-multi-stage-builds-the-game-changer)
  - [The Concept](#the-concept)
  - [Real-World Example: React Application](#real-world-example-react-application)
  - [How Multi-Stage Builds Work](#how-multi-stage-builds-work)
  - [Multi-Stage Build Patterns](#multi-stage-build-patterns)
  - [Advanced Multi-Stage Techniques](#advanced-multi-stage-techniques)
- [Technique 6: Essential Docker Optimization Tools](#technique-6-essential-docker-optimization-tools)
  - [Dive: Image Layer Explorer](#dive-image-layer-explorer)
  - [Docker Slim: Automated Optimization](#docker-slim-automated-optimization)
  - [Additional Optimization Tools](#additional-optimization-tools)
- [Complete Optimization Workflow: Putting It All Together](#complete-optimization-workflow-putting-it-all-together)
  - [Before Optimization](#before-optimization)
  - [After Optimization](#after-optimization)
- [Performance Comparison and Metrics](#performance-comparison-and-metrics)
  - [Size Reduction Journey](#size-reduction-journey)
  - [Deployment Impact](#deployment-impact)
- [Best Practices and Common Pitfalls](#best-practices-and-common-pitfalls)
  - [Do's](#dos)
  - [Don'ts](#donts)
  - [Common Mistakes](#common-mistakes)
- [Conclusion: The 99% Reduction Framework](#conclusion-the-99-reduction-framework)
  - [Next Steps](#next-steps)
  - [Additional Resources](#additional-resources)

---

<div id="introduction-why-docker-image-size-matters-in-2025"></div>

## Introduction: Why Docker Image Size Matters in 2025

Docker image optimization is no longer optional, it's a critical skill for modern DevOps engineers and developers. In this comprehensive guide, I'll walk you through the exact steps I use to reduce Docker images from 1.2GB to just 10MB, achieving a 99% size reduction without sacrificing functionality.

Whether you're working with Node.js, Python, React, or any containerized application, these optimization techniques will help you:

- **Reduce storage costs** by up to 90%
- **Accelerate deployment times** significantly
- **Improve scalability** in Kubernetes and container orchestration platforms
- **Enhance security** by minimizing attack surfaces
- **Optimize CI/CD pipeline performance**

Even if you're an experienced Docker user, the advanced techniques covered here particularly multi-stage builds and layer squashing will transform how you build production containers.

<div id="understanding-the-impact-why-every-megabyte-counts"></div>

## Understanding the Impact: Why Every Megabyte Counts

Before diving into optimization techniques, let's understand why Docker image size matters:

<div id="storage-cost-implications"></div>

### Storage Cost Implications

Large Docker images consume expensive storage across your entire infrastructure:

- Container registries (Docker Hub, ECR, GCR)
- CI/CD pipeline artifacts
- Production cluster nodes
- Backup systems

<div id="deployment-speed-and-scalability"></div>

### Deployment Speed and Scalability

In Kubernetes environments, image size directly impacts:

- Pod startup times
- Horizontal scaling speed
- Rolling update duration
- Resource consumption during pulls

<div id="security-considerations"></div>

### Security Considerations

Smaller images mean:

- Fewer packages and dependencies
- Reduced attack surface
- Less vulnerability exposure
- Faster security scanning

<div id="technique-1-start-with-minimal-base-images"></div>

## Technique 1: Start With Minimal Base Images

The foundation of Docker optimization begins with choosing the right base image.

<div id="the-problem-bloated-base-images"></div>

### The Problem: Bloated Base Images

Many developers default to convenience images like `node:latest` or `python:latest`, which can exceed 1GB. These images include:

- Full operating system utilities
- Package managers
- Development tools
- Documentation
- Unnecessary system libraries

**Example:** A React application using `node:latest` starts at over 1GBlike using a cargo ship to deliver a letter.

<div id="the-solution-alpine-linux-images"></div>

### The Solution: Alpine Linux Images

Alpine Linux is purpose-built for containers, weighing only ~5MB for the base image.

**Before:**

```dockerfile
FROM node:latest
```

Image size: ~1.2GB

**After:**

```dockerfile
FROM node:alpine
```

Image size: ~250MB

**Result:** 80% size reduction with just seven characters.

<div id="understanding-alpine-linux"></div>

### Understanding Alpine Linux

Alpine advantages:

- Uses musl libc instead of glibc
- Minimal package installation
- Security-focused design
- APK package manager
- Regular security updates

<div id="compatibility-considerations"></div>

### Compatibility Considerations

Alpine isn't always the perfect choice:

- **Native modules:** May have compatibility issues with Node.js native addons
- **System libraries:** Different from standard Linux distributions
- **Build tools:** May require additional packages for compilation

**Best Practice:** Use full images for development, minimal variants for production.

<div id="alternative-google-distroless-images"></div>

### Alternative: Google Distroless Images

For maximum minimalism, consider Distroless images:

- No operating system components
- No shell or package manager
- No basic Linux utilities
- Only application runtime and dependencies

**Trade-off:** More complex setup and debugging, but unmatched security and size benefits.

<div id="technique-2-optimize-layer-caching-for-fast-rebuilds"></div>

## Technique 2: Optimize Layer Caching for Fast Rebuilds

Proper layer caching transforms rebuild times from minutes to seconds.

<div id="how-docker-layer-caching-works"></div>

### How Docker Layer Caching Works

Each Dockerfile instruction creates an immutable layer. Docker reuses cached layers when:

1. The instruction hasn't changed
2. Files being copied are identical
3. All previous layers match

<div id="the-problem-cache-invalidation"></div>

### The Problem: Cache Invalidation

**Inefficient Dockerfile:**

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
```

**Issue:** Any code change invalidates the cache, forcing complete dependency reinstallation.

<div id="the-solution-strategic-layer-ordering"></div>

### The Solution: Strategic Layer Ordering

**Optimized Dockerfile:**

```dockerfile
FROM node:alpine
WORKDIR /app

# Copy dependency manifests first (changes infrequently)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Copy source code last (changes frequently)
COPY . .
RUN npm run build
```

<div id="key-caching-principles"></div>

### Key Caching Principles

1. **Order by stability:** Most stable instructions first, frequently changing ones last
2. **Separate dependencies:** Copy `package.json` before source code
3. **Use specific copy commands:** Avoid `COPY . .` when possible
4. **Combine related operations:** Group commands that always change together

<div id="language-specific-examples"></div>

### Language-Specific Examples

**Python:**

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

**Go:**

```dockerfile
COPY go.mod go.sum ./
RUN go mod download
COPY . .
```

**Java/Maven:**

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
```

<div id="technique-3-leverage-dockerignore-files"></div>

## Technique 3: Leverage .dockerignore Files

The `.dockerignore` file is Docker's equivalent of `.gitignore`, preventing unnecessary files from bloating your build context.

<div id="understanding-build-context"></div>

### Understanding Build Context

When you run `docker build`, Docker sends the entire directory (build context) to the Docker daemon. Without `.dockerignore`, this includes:

- `node_modules/` (hundreds of MB)
- `.git/` directory
- Build artifacts
- Log files
- IDE configurations
- Secret files

<div id="creating-an-effective-dockerignore"></div>

### Creating an Effective .dockerignore

**Example .dockerignore:**

```gitignore
# Dependencies
node_modules/
npm-debug.log
yarn-error.log

# Build outputs
dist/
build/
*.log

# Development files
.git/
.gitignore
.env
.env.local
*.md
Dockerfile
docker-compose.yml

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Testing
coverage/
*.test.js
```

<div id="impact-measurement"></div>

### Impact Measurement

**Before .dockerignore:**

```text
Sending build context to Docker daemon: 856MB
```

**After .dockerignore:**

```text
Sending build context to Docker daemon: 2.3MB
```

<div id="security-benefits"></div>

### Security Benefits

Beyond size optimization, `.dockerignore` prevents:

- Accidentally copying secrets (`.env` files, credentials)
- Exposing source control history
- Including sensitive configuration files
- Leaking development-only credentials

<div id="technique-4-master-layer-squashing-and-immutability"></div>

## Technique 4: Master Layer Squashing and Immutability

Understanding Docker's layer system is critical for advanced optimization.

<div id="the-layer-immutability-problem"></div>

### The Layer Immutability Problem

**Inefficient approach:**

```dockerfile
FROM node:alpine
COPY . .
RUN npm install
RUN npm run build
RUN rm -rf node_modules
RUN npm cache clean --force
```

**Expected:** Smaller image after deletions
**Reality:** Image size unchanged or larger

<div id="why-deletions-dont-save-space"></div>

### Why Deletions Don't Save Space

Docker layers are immutable and additive:

1. Each `RUN` command creates a new layer
2. Layers only store changes (delta) from previous layers
3. Deletions create "whiteout" files marking content as hidden
4. Original data remains in earlier layers
5. Final image size = sum of all layers

<div id="the-solution-combine-operations"></div>

### The Solution: Combine Operations

**Optimized approach:**

```dockerfile
FROM node:alpine
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force
COPY . .
RUN npm run build && \
    rm -rf src && \
    rm -rf tests
```

<div id="security-implications-of-layer-immutability"></div>

### Security Implications of Layer Immutability

**Critical warning:** Files deleted in later layers remain accessible in the image history.

**Dangerous pattern:**

```dockerfile
COPY .env .
RUN use_secrets_here
RUN rm .env  # WARNING: Still extractable from earlier layer!
```

**Anyone can extract secrets:**

```bash
docker save myimage > image.tar
tar -xf image.tar
# Secrets visible in layer directories
```

<div id="best-practices-for-layer-optimization"></div>

### Best Practices for Layer Optimization

1. **Chain related commands:** Use `&&` to combine operations in single RUN instructions
2. **Clean in same layer:** Install and cleanup within the same RUN command
3. **Never commit secrets:** Use build arguments or mount secrets at runtime
4. **Minimize layers:** Combine commands that logically belong together

**Example:**

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        package1 \
        package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

<div id="technique-5-implement-multi-stage-builds-the-game-changer"></div>

## Technique 5: Implement Multi-Stage Builds (The Game Changer)

Multi-stage builds are the most powerful Docker optimization technique, enabling 95%+ size reductions.

<div id="the-concept"></div>

### The Concept

Multi-stage builds allow you to:

1. Use a full-featured image for building
2. Copy only necessary artifacts to a minimal runtime image
3. Discard build tools, dependencies, and source code

<div id="real-world-example-react-application"></div>

### Real-World Example: React Application

**Before (Single-stage):**

```dockerfile
FROM node:alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**Size:** ~250MB (includes Node, npm, source code, node_modules)

**After (Multi-stage):**

```dockerfile
# Build stage
FROM node:alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Size:** ~57MB (only nginx + static files)

<div id="how-multi-stage-builds-work"></div>

### How Multi-Stage Builds Work

1. **FROM ... AS builder:** Creates named build stage
2. **Build operations:** Install dependencies, compile, test
3. **FROM nginx:alpine:** Starts fresh with minimal runtime image
4. **COPY --from=builder:** Copies only built artifacts
5. **Everything else discarded:** Build tools, source code, intermediate files

<div id="multi-stage-build-patterns"></div>

### Multi-Stage Build Patterns

**Node.js API Application:**

```dockerfile
# Build stage
FROM node:alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Python Application:**

```dockerfile
# Build stage
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

**Go Application:**

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Production stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

<div id="advanced-multi-stage-techniques"></div>

### Advanced Multi-Stage Techniques

**Multiple build stages:**

```dockerfile
# Dependencies stage
FROM node:alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Build stage
FROM node:alpine AS builder
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Test stage
FROM builder AS tester
RUN npm run test

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

<div id="technique-6-essential-docker-optimization-tools"></div>

## Technique 6: Essential Docker Optimization Tools

Professional Docker optimization requires specialized tools for analysis and debugging.

<div id="dive-image-layer-explorer"></div>

### Dive: Image Layer Explorer

**Dive** provides interactive exploration of Docker image layers.

**Installation:**

```bash
# macOS
brew install dive

# Linux
wget https://github.com/wagoodman/dive/releases/download/v0.11.0/dive_0.11.0_linux_amd64.deb
sudo apt install ./dive_0.11.0_linux_amd64.deb

# Windows
scoop install dive
```

**Usage:**

```bash
dive myimage:latest
```

**Features:**

- Layer-by-layer size breakdown
- File system changes visualization
- Wasted space identification
- Efficiency score calculation
- Interactive file explorer

**What Dive Reveals:**

- Which layers consume the most space
- Duplicate files across layers
- Files added then removed (wasted space)
- Optimization opportunities

<div id="docker-slim-automated-optimization"></div>

### Docker Slim: Automated Optimization

**Docker Slim** automatically minifies images by up to 30x while maintaining functionality.

**Installation:**

```bash
# macOS
brew install docker-slim

# Linux
curl -L -o ds.tar.gz https://github.com/docker-slim/docker-slim/releases/download/1.40.0/dist_linux.tar.gz
tar -xvf ds.tar.gz
sudo mv dist_linux/* /usr/local/bin/
```

**Basic Usage:**

```bash
docker-slim build myimage:latest
```

**Advanced Features:**

**1. Automatic minification:**

```bash
docker-slim build --target myimage:latest --tag myimage:slim
```

**2. HTTP probing:**

```bash
docker-slim build --http-probe myimage:latest
```

**3. Security scanning:**

```bash
docker-slim xray myimage:latest
```

**4. Dockerfile linting:**

```bash
docker-slim lint Dockerfile
```

**Real Results:**

- **Original image:** 250MB
- **After multi-stage build:** 57MB
- **After Docker Slim:** 10MB

<div id="additional-optimization-tools"></div>

### Additional Optimization Tools

**Trivy (Security Scanner):**

```bash
trivy image myimage:latest
```

Identifies vulnerabilities and suggests smaller base images.

**Hadolint (Dockerfile Linter):**

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

Enforces best practices and optimization patterns.

**Docker Scout:**

```bash
docker scout cves myimage:latest
```

Analyzes supply chain security and suggests optimizations.

<div id="complete-optimization-workflow-putting-it-all-together"></div>

## Complete Optimization Workflow: Putting It All Together

Here's a comprehensive example combining all techniques:

<div id="before-optimization"></div>

### Before Optimization

**Dockerfile (inefficient):**

```dockerfile
FROM node:latest
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**Stats:**

- Image size: 1.2GB
- Build time: 5 minutes
- Layers: 15
- Vulnerabilities: 47

<div id="after-optimization"></div>

### After Optimization

**Dockerfile (optimized):**

```dockerfile
# Build stage
FROM node:alpine AS builder
WORKDIR /app

# Copy dependency files (cache optimization)
COPY package.json package-lock.json ./

# Install dependencies with cache cleaning
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build && \
    rm -rf src tests

# Production stage
FROM nginx:alpine

# Copy built assets
COPY --from=builder /app/dist /usr/share/nginx/html

# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**.dockerignore:**

```gitignore
node_modules
npm-debug.log
.git
.env
.env.local
README.md
Dockerfile
docker-compose.yml
.vscode
coverage
*.test.js
.DS_Store
```

**Stats:**

- Image size: 10MB (99% reduction)
- Build time: 45 seconds (90% faster)
- Layers: 8
- Vulnerabilities: 2

<div id="performance-comparison-and-metrics"></div>

## Performance Comparison and Metrics

<div id="size-reduction-journey"></div>

### Size Reduction Journey

| Optimization Step      | Image Size | Reduction | Cumulative |
| ---------------------- | ---------- | --------- | ---------- |
| Original (node:latest) | 1.2GB      | -         | -          |
| Alpine base            | 250MB      | 79%       | 79%        |
| Layer caching          | 250MB      | 0%        | 79%        |
| .dockerignore          | 248MB      | 1%        | 80%        |
| Layer squashing        | 245MB      | 1%        | 80%        |
| Multi-stage build      | 57MB       | 77%       | 95%        |
| Docker Slim            | 10MB       | 82%       | 99%        |

<div id="deployment-impact"></div>

### Deployment Impact

**Kubernetes Pod Startup:**

- **Before:** 3-5 minutes (1.2GB image pull)
- **After:** 5-10 seconds (10MB image pull)

**Storage Costs (100 nodes):**

- **Before:** 120GB x $0.10/GB = $12/month
- **After:** 1GB x $0.10/GB = $0.10/month

**CI/CD Pipeline:**

- **Before:** 8-minute build + push
- **After:** 1-minute build + push

<div id="best-practices-and-common-pitfalls"></div>

## Best Practices and Common Pitfalls

<div id="dos"></div>

### Do's

- Use Alpine or minimal base images
- Implement multi-stage builds for all production images
- Create comprehensive .dockerignore files
- Order Dockerfile instructions from least to most frequently changing
- Combine RUN commands with cleanup in single layers
- Use specific COPY commands instead of COPY . .
- Leverage build cache effectively
- Regularly audit images with security scanners
- Document optimization decisions

<div id="donts"></div>

### Don'ts

- Use :latest tags in production
- Install unnecessary development dependencies
- Copy secrets into images
- Run containers as root
- Ignore security vulnerabilities
- Skip .dockerignore files
- Use separate RUN commands for install + cleanup
- Include source code in production images
- Forget to test optimized images thoroughly

<div id="common-mistakes"></div>

### Common Mistakes

**Mistake 1: Premature optimization**
Test with standard images first, then optimize for production.

**Mistake 2: Breaking functionality**
Always test optimized images match original behavior.

**Mistake 3: Ignoring Alpine compatibility**
Native modules may fail with Alpine, test thoroughly.

**Mistake 4: Over-aggressive cleanup**
Ensure runtime dependencies remain in final image.

<div id="conclusion-the-99-reduction-framework"></div>

## Conclusion: The 99% Reduction Framework

Docker image optimization isn't just about saving disk space, it's about building faster, more secure, and more scalable applications. By implementing these six techniques, you can achieve dramatic size reductions:

1. **Start with minimal base images** (80% reduction)
2. **Optimize layer caching** (faster rebuilds)
3. **Use .dockerignore files** (smaller build context)
4. **Master layer squashing** (eliminate waste)
5. **Implement multi-stage builds** (95% reduction)
6. **Leverage optimization tools** (99% reduction)

The journey from 1.2GB to 10MB demonstrates that with proper techniques, you can achieve enterprise-grade optimization without sacrificing functionality.

<div id="next-steps"></div>

### Next Steps

1. Audit your current Docker images with Dive
2. Implement multi-stage builds for your applications
3. Create comprehensive .dockerignore files
4. Integrate Docker Slim into your CI/CD pipeline
5. Monitor image sizes in production
6. Share these techniques with your team

<div id="additional-resources"></div>

### Additional Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Alpine Linux Docker Images](https://hub.docker.com/_/alpine)
- [Google Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [Dive GitHub Repository](https://github.com/wagoodman/dive)
- [Docker Slim Project](https://github.com/docker-slim/docker-slim)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
