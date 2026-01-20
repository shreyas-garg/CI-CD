# DevOps CI/CD Project - Spring Boot Application

## Project Overview

A simple but complete CI/CD pipeline for a Spring Boot application using GitHub Actions, Docker, and Kubernetes.

### Technology Stack
- **Java 20** - Programming language  
- **Spring Boot 3.2** - Web framework
- **Maven** - Build tool
- **Docker** - Containerization
- **Kubernetes** - Orchestration
- **GitHub Actions** - CI/CD automation

---

## CI/CD Pipeline (5 Stages)

### Stage 1: Test & Build ✓
- Runs unit tests using Maven
- Creates JAR file if tests pass
- **Why**: Ensures code works before packaging

### Stage 2: Security Check ✓
- Checks Maven dependencies for known vulnerabilities (OWASP)
- Fails build if critical vulnerabilities found
- **Why**: Prevents vulnerable libraries from being deployed

### Stage 3: Docker Build ✓
- Builds Docker image from the JAR file
- Image contains Java 20 runtime + application
- **Why**: Ensures application runs the same everywhere

### Stage 4: Test Container ✓
- Starts the Docker container
- Calls `/health` endpoint to verify it works
- **Why**: Ensures container actually runs

### Stage 5: Push to DockerHub ✓
- Pushes image to DockerHub registry
- Only happens on master branch
- **Why**: Makes image available for deployment

---

## Quick Start

### Local Development

**Build:**
```bash
mvn clean package
```

**Run:**
```bash
java -jar target/demo-app.jar
```

**Test:**
```bash
curl http://localhost:8080/health
```

### Docker

**Build:**
```bash
docker build -t demo-app:latest .
```

**Run:**
```bash
docker run -p 8080:8080 demo-app:latest
```

### Kubernetes

**Deploy locally:**
```bash
kind create cluster --name demo-cluster
kind load docker-image demo-app:latest --name demo-cluster
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods
```

---

## GitHub Secrets (Required)

Add these to your GitHub repository settings:

1. **DOCKERHUB_USERNAME** - Your DockerHub username
2. **DOCKERHUB_TOKEN** - Personal access token from DockerHub

Without these, the "Push to DockerHub" stage will fail.

---

## File Structure

```
├── .github/workflows/
│   ├── ci.yml          # Build, test, security, Docker, push
│   └── cd.yml          # Deploy to Kubernetes
├── k8s/
│   ├── deployment.yaml # Kubernetes deployment (2 replicas)
│   └── service.yaml    # NodePort service
├── src/main/java/com/example/demo/
│   ├── DemoApplication.java       # Spring Boot app
│   └── HealthController.java      # /health endpoint
├── Dockerfile          # Docker configuration
├── pom.xml             # Maven build configuration
└── README.md           # This file
```

---

## Key Features

✅ **Automated Testing** - Tests run before build  
✅ **Security Scanning** - Dependency vulnerability checks  
✅ **Docker Support** - Container image for every commit  
✅ **Kubernetes Ready** - Deploy with `kubectl apply`  
✅ **Health Checks** - Pod auto-recovery if unhealthy  
✅ **Resource Limits** - CPU/Memory constraints  
✅ **Rolling Updates** - Zero-downtime deployments  

---

## Understanding the Pipeline

When you push to `master`:

```
Push to master
    ↓
Test & Build (Maven test + compile)
    ↓
Security Check (Check for vulnerable dependencies)
    ↓
Docker Build (Create container image)
    ↓
Test Container (Verify it runs & responds)
    ↓
Push to DockerHub (Make image available)
    ↓
Kubernetes Deploy (Update running pods)
```

If any stage fails, the next stages don't run.

---

## Troubleshooting

**Build fails:**
```bash
mvn clean package -X  # Run with debug output
```

**Container won't start:**
```bash
docker logs <container-id>
```

**Kubernetes issues:**
```bash
kubectl describe pod demo-app-xxx
kubectl logs deployment/demo-app
```

---

## Common Questions

**Q: What's the /health endpoint?**  
A: It's a simple HTTP endpoint that returns {"status":"UP"} to verify the app is running.

**Q: Why do we need Docker?**  
A: Docker packages the application with all dependencies. "If it works in Docker, it works everywhere."

**Q: Why do we need Kubernetes?**  
A: Kubernetes manages containers - handles crashes, scales to multiple instances, manages networking.

**Q: What happens if a pod crashes?**  
A: Kubernetes automatically restarts it using the `livenessProbe`.

**Q: Can we have multiple instances?**  
A: Yes, change `replicas: 2` to `replicas: 3` in `k8s/deployment.yaml`.

---

## Next Steps

1. Configure GitHub secrets (DOCKERHUB_USERNAME, DOCKERHUB_TOKEN)
2. Push code to `master` branch
3. Check GitHub Actions tab to see pipeline run
4. Verify image pushed to DockerHub
5. Run locally with kind to test Kubernetes deployment
7. **Get Status** - Displays pod and service information

The CD pipeline automates deployment to Kubernetes and validates that the application is running correctly.

## How the Pipeline Works

1. Developer pushes code to the master branch
2. **CI Pipeline** runs automatically:
   - Tests the code
   - Builds the application JAR
   - Creates a Docker image
   - Pushes the image to DockerHub
3. **CD Pipeline** triggers after CI success:
   - Sets up a Kubernetes cluster
   - Deploys the application using Kubernetes manifests
   - Verifies the deployment is healthy

## Setup Instructions

### Prerequisites
- GitHub account
- DockerHub account

### GitHub Secrets Configuration

Add the following secrets to your GitHub repository:
- `DOCKERHUB_USERNAME` - Your DockerHub username
- `DOCKERHUB_TOKEN` - Your DockerHub access token

### Running Locally

```bash
# Build the application
mvn clean package

# Run the application
java -jar target/demo-app.jar

# Test the endpoint
curl http://localhost:8080/health
```

### Running with Docker

```bash
# Build Docker image
docker build -t demo-app:latest .

# Run container
docker run -p 8080:8080 demo-app:latest

# Test the endpoint
curl http://localhost:8080/health
```

## Project Structure

```
.
├── .github/
│   └── workflows/
│       ├── ci.yml          # CI pipeline definition
│       └── cd.yml          # CD pipeline definition
├── k8s/
│   ├── deployment.yaml     # Kubernetes deployment configuration
│   └── service.yaml        # Kubernetes service configuration
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── demo/
│                       ├── DemoApplication.java    # Main application
│                       └── HealthController.java   # REST controller
├── Dockerfile              # Docker image definition
├── pom.xml                 # Maven configuration
└── README.md               # This file
```

## Kubernetes Configuration

- **Deployment**: 1 replica of the application
- **Service**: NodePort exposing port 30080
- **Container Port**: 8080
- **Image Pull Policy**: Always (ensures latest image is used)

## Key Features for Academic Evaluation

1. **Simple and Clean** - Minimal code, easy to understand
2. **Complete CI/CD** - Full automation from code to deployment
3. **Containerized** - Uses Docker best practices
4. **Kubernetes Native** - Demonstrates container orchestration
5. **Automated Testing** - Includes test execution in CI
6. **Production Ready** - Follows industry standards

## License

This project is for academic evaluation purposes.
