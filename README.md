# Tech Journey Deployment

Docker-based deployment configuration for the Tech Journey application.

This repository brings together the Angular frontend and Spring Boot API as separate containerized services and provides the production orchestration used to run Tech Journey on DigitalOcean.

**Live application:**  
https://museum.techjourney.dev

---

# Architecture

Tech Journey uses separate repositories for the frontend, backend, and deployment configuration.

```text
                    Internet
                       │
                       ▼
             museum.techjourney.dev
                       │
                       ▼
              DigitalOcean Droplet
                       │
                       ▼
                  Docker Compose
                 ┌───────────────┐
                 │               │
                 ▼               ▼
        tech-journey-ui   tech-journey-api
        Angular + Nginx    Spring Boot
                 │               ▲
                 │    /api       │
                 └───────────────┘
```

The frontend and backend remain independently buildable while Docker Compose provides a single way to run the complete application stack.

---

# Repositories

Tech Journey is split across three repositories.

### Angular Frontend

https://github.com/wjones-dev/tech-journey

Contains:

- Technology Museum
- Technology Detail views
- API Explorer
- Engineering Lab
- interactive frontend experiments

---

### Spring Boot API

https://github.com/wjones-dev/tech-journey-api

Contains:

- Timeline REST API
- CRUD Sandbox API
- Engineering Lab services
- Java Stream experiments
- Spring Security demonstrations
- Spring Data JPA persistence

---

### Deployment

https://github.com/wjones-dev/tech-journey-deployment

Contains the Docker Compose configuration responsible for running the frontend and backend together.

---

# Container Architecture

The production environment currently consists of two primary application containers:

```text
tech-journey-ui
    Angular production build
    served by Nginx

tech-journey-api
    Spring Boot application
    running on Java 17
```

Docker Compose creates the application network that allows the services to communicate using their service names rather than hard-coded container IP addresses.

---

# Request Flow

Browser requests enter through the frontend application.

```text
Browser
   │
   ▼
Angular / Nginx
   │
   ├──────── Static Angular application
   │
   └── /api/*
          │
          ▼
     Spring Boot API
```

The Angular application does not need to know the physical location of the backend container.

Requests using `/api` are routed to the Spring Boot service through the container network.

This keeps the public-facing application under a single origin while maintaining separate frontend and backend services internally.

---

# Technology Stack

| Technology | Purpose |
|---|---|
| Docker | Application containerization |
| Docker Compose | Multi-container orchestration |
| Nginx | Angular production server and API proxy |
| Angular | Frontend application |
| Spring Boot | Backend REST API |
| Java 17 | Backend runtime |
| Ubuntu | Production host operating system |
| DigitalOcean | Cloud infrastructure |
| GitHub | Source control and deployment source |

---

# Production Host

The application currently runs on an Ubuntu DigitalOcean Droplet.

The deployment keeps the three repositories separate on the server:

```text
tech-journey/
tech-journey-api/
tech-journey-deployment/
```

The deployment repository acts as the orchestration layer between the frontend and backend projects.

---

# Ports

The current container configuration exposes:

```text
Frontend
Host 4200 → Container 80

Backend
Host 8080 → Container 8080
```

Inside the Docker environment, application communication occurs through Docker networking rather than relying on external IP addresses.

---

# Prerequisites

To run the complete stack:

```text
Docker
Docker Compose
Git
```

The Angular and Spring Boot repositories should also be available alongside this deployment repository.

A typical directory structure is:

```text
techjourney/
├── tech-journey/
├── tech-journey-api/
└── tech-journey-deployment/
```

---

# Start the Application

From the deployment repository:

```bash
docker compose up -d --build
```

This will:

```text
Build Angular frontend
        ↓
Build Spring Boot API
        ↓
Create Docker network
        ↓
Create containers
        ↓
Start application stack
```

The `-d` option runs the containers in detached mode.

---

# View Running Containers

```bash
docker compose ps
```

or:

```bash
docker ps
```

Typical services include:

```text
tech-journey-ui
tech-journey-api
```

---

# View Logs

View logs for the entire stack:

```bash
docker compose logs -f
```

Frontend only:

```bash
docker compose logs -f tech-journey-ui
```

Backend only:

```bash
docker compose logs -f tech-journey-api
```

---

# Stop the Application

```bash
docker compose down
```

This stops and removes the application containers and Compose network.

---

# Rebuild the Application

After application code has changed:

```bash
docker compose up -d --build
```

Docker rebuilds the affected images and recreates the services using the updated application code.

---

# Production Update Workflow

Application updates follow a straightforward Git-based deployment process.

```text
Develop locally
      ↓
Commit changes
      ↓
Push to GitHub
      ↓
SSH into DigitalOcean
      ↓
Pull latest repositories
      ↓
Rebuild Docker services
      ↓
Run updated containers
```

For example:

```bash
cd tech-journey
git pull

cd ../tech-journey-api
git pull

cd ../tech-journey-deployment
docker compose up -d --build
```

---

# Containerization Strategy

The frontend and backend use different container build strategies.

## Angular Frontend

The frontend uses a multi-stage Docker build:

```text
Node.js Build Environment
        ↓
Angular Production Build
        ↓
Nginx Runtime Image
```

Only the compiled Angular application and Nginx configuration are required in the final runtime image.

---

## Spring Boot API

The backend is packaged as a Java application and runs using a Java 17 runtime image.

```text
Spring Boot Build
       ↓
Application JAR
       ↓
Java 17 Runtime
```

This keeps the frontend and backend independently deployable while allowing Docker Compose to operate them as one application.

---

# Why a Separate Deployment Repository?

Infrastructure configuration is intentionally kept separate from application code.

```text
Frontend Repository
      │
      ├── Angular concerns
      │
Backend Repository
      │
      ├── Java / Spring concerns
      │
Deployment Repository
      │
      └── infrastructure concerns
```

This separation makes it easier to change application code without mixing deployment configuration into either project.

It also allows the entire production topology to be understood from one repository.

---

# Deployment Goal

The deployment layer demonstrates the path from source code to a running production application.

```text
SOURCE CODE
     ↓
BUILD
     ↓
CONTAINERIZE
     ↓
ORCHESTRATE
     ↓
DEPLOY
     ↓
RUN
```

Tech Journey is not only an Angular frontend and Spring Boot API — it is a complete deployed application with independently containerized services communicating as part of a production environment.
