# Advanced DevOps CI/CD Project: Spring Boot Application

## Project Overview
This is a production-grade CI/CD pipeline implementation using GitHub Actions and Kubernetes for a Spring Boot application. The project demonstrates industry best practices in Continuous Integration, Continuous Deployment, security scanning, and containerization.

### Technology Stack
- **Language**: Java 21 (LTS)
- **Framework**: Spring Boot 3.2.0
- **Build Tool**: Maven 3.9
- **Container**: Docker
- **Orchestration**: Kubernetes
- **CI/CD**: GitHub Actions
- **Security**: CodeQL, Trivy, OWASP Dependency-Check

---

## CI/CD Pipeline Architecture

### Pipeline Stages Explained

#### 1. **Code Checkout** ✓
**Why it matters**: First step to retrieve source code from repository
- Fetches the latest code from master branch
- Enables all downstream stages to work with current codebase

#### 2. **Setup Runtime Environment** ✓
**Why it matters**: Ensures consistent Java environment across all builds
- Installs JDK 21 (LTS version)
- Caches Maven dependencies to speed up builds
- Prevents "works on my machine" issues

#### 3. **Code Linting (Checkstyle)** ✓
**Why it matters**: Enforces coding standards and prevents technical debt
- Validates code follows Google Java Style Guide
- Catches formatting issues early
- Improves code maintainability
- Prevents style-related code reviews

#### 4. **SAST - CodeQL Analysis** ✓
**Why it matters**: Detects security vulnerabilities in application code
- Uses GitHub's CodeQL engine
- Identifies OWASP Top 10 vulnerabilities
- Analyzes code patterns for potential security issues
- Results integrated into GitHub Security tab
- **Shift-left Security**: Catches vulnerabilities before deployment

#### 5. **SCA - Dependency Analysis** ✓
**Why it matters**: Identifies vulnerable dependencies (supply-chain security)
- Uses OWASP Dependency-Check
- Scans Maven dependencies for known CVEs
- Fails build if critical vulnerabilities found
- Ensures third-party libraries are secure

#### 6. **Unit Tests** ✓
**Why it matters**: Validates business logic correctness
- Runs all unit tests in the project
- Ensures code doesn't have regressions
- Generates test reports for documentation
- Prevents broken code from being deployed

#### 7. **Build Artifact** ✓
**Why it matters**: Creates executable JAR file
- Compiles Java code to bytecode
- Packages with dependencies
- Artifact uploaded for later stages
- Ensures only tested code is deployed

#### 8. **Docker Build** ✓
**Why it matters**: Creates container image for consistency across environments
- Builds image with Java 21 runtime
- Ensures application runs same way everywhere
- Image versioned by commit SHA
- Enables containerized deployment

#### 9. **Container Vulnerability Scan (Trivy)** ✓
**Why it matters**: Detects OS and library vulnerabilities in Docker image
- Scans container for security vulnerabilities
- Checks base image for known CVEs
- Prevents vulnerable images from reaching production
- Results integrated into GitHub Security tab

#### 10. **Runtime Container Test** ✓
**Why it matters**: Validates application works correctly in container
- Smoke test: Verifies application starts
- Health check: Confirms `/health` endpoint responds
- Integration test: Ensures container networking works
- Catches runtime issues before deployment

#### 11. **Push to Registry** ✓
**Why it matters**: Makes image available for deployment
- Only pushes if all checks pass (security gate)
- Only pushes from master branch (production safety)
- Tags image with commit SHA (traceability)
- Enables versioning and rollback capabilities

---

## Security Architecture (DevSecOps)

### Shift-Left Security Philosophy
This pipeline implements security checks at every stage to catch issues early:

```
Development → Commit → CodeQL → Dependency Check → Build → Container Scan → Deploy
   (IDE)     (Pre-commit)  (SAST)     (SCA)       (Artifact)   (Image)     (Secure)
```

### Security Gates
1. **Code-level vulnerabilities** blocked by CodeQL
2. **Dependency vulnerabilities** blocked by Dependency-Check
3. **Container vulnerabilities** identified by Trivy (advisory)
4. **Staging security** ensured by health checks

---

## Local Setup & Testing

### Prerequisites
```bash
# Java 21
java --version  # Should be 21.x.x

# Maven
mvn --version   # Should be 3.9+

# Docker
docker --version

# Kubernetes
kubectl version --client
```

### Build & Run Locally

#### 1. Build the Application
```bash
cd /Users/shreyasgarg/Desktop/ci
mvn clean package
```

#### 2. Run Application Locally
```bash
java -jar target/demo-app.jar
# Application runs on http://localhost:8080
```

#### 3. Access Health Endpoint
```bash
curl http://localhost:8080/health
# Expected: {"status":"UP"}
```

#### 4. Run Unit Tests
```bash
mvn test
```

#### 5. Run Code Quality Checks
```bash
mvn checkstyle:check
```

#### 6. Check Dependencies for Vulnerabilities
```bash
mvn org.owasp:dependency-check-maven:check
```

---

## Docker Usage

### Build Docker Image
```bash
docker build -t demo-app:latest .
```

### Run Container
```bash
docker run -d -p 8080:8080 --name demo-app demo-app:latest
```

### Verify Container
```bash
curl http://localhost:8080/health
docker logs demo-app
```

### Scan Image with Trivy
```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image demo-app:latest
```

---

## Kubernetes Deployment

### Local Testing with Kind (Kubernetes in Docker)

#### 1. Create Kind Cluster
```bash
kind create cluster --name demo-cluster
```

#### 2. Load Docker Image
```bash
kind load docker-image demo-app:latest --name demo-cluster
```

#### 3. Deploy Application
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

#### 4. Check Deployment Status
```bash
kubectl get pods
kubectl get svc

# Expected output:
# NAME              READY   STATUS    RESTARTS   AGE
# demo-app-xxxxx    1/1     Running   0          10s
```

#### 5. Access Application
```bash
# Get the node port
kubectl get svc demo-app-service

# Access the application
curl http://localhost:30080/health
```

#### 6. View Logs
```bash
kubectl logs -f deployment/demo-app
```

#### 7. Cleanup
```bash
kind delete cluster --name demo-cluster
```

---

## GitHub Secrets Configuration (REQUIRED)

### Required Secrets
Add these secrets to your GitHub repository:

1. **DOCKERHUB_USERNAME**
   - Your DockerHub username
   - Used for pushing images to DockerHub
   - Path: Settings → Secrets and variables → Actions → New repository secret

2. **DOCKERHUB_TOKEN**
   - DockerHub Personal Access Token
   - Create at: https://hub.docker.com/settings/security
   - Requires push permissions

### How to Add Secrets
1. Go to Repository → Settings
2. Click "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`

---

## File Structure

```
ci/
├── .github/
│   └── workflows/
│       ├── ci.yml          # Main CI pipeline (11 stages)
│       └── cd.yml          # CD pipeline (Kubernetes deployment)
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment manifest
│   └── service.yaml        # Kubernetes Service manifest
├── src/
│   └── main/
│       └── java/com/example/demo/
│           ├── DemoApplication.java      # Main Spring Boot app
│           └── HealthController.java     # Health endpoint
├── Dockerfile              # Docker image definition
├── pom.xml                 # Maven configuration with plugins
├── README.md               # This file
```

---

## Key Features

### Continuous Integration (CI)
- ✅ Automated builds on every push
- ✅ Maven dependency caching for fast builds
- ✅ Parallel stage execution where possible
- ✅ Artifact storage for traceability

### Code Quality
- ✅ Checkstyle validation (Google Java Style Guide)
- ✅ Static analysis for maintainability
- ✅ Code coverage reporting

### Security (DevSecOps)
- ✅ CodeQL SAST for application vulnerabilities
- ✅ OWASP Dependency-Check for supply-chain risks
- ✅ Trivy for container image scanning
- ✅ Security findings in GitHub Security tab
- ✅ Multiple security gates prevent vulnerable code from reaching production

### Containerization
- ✅ Docker image built for every commit
- ✅ Base image uses Java 21 LTS
- ✅ Multi-stage health checks
- ✅ Proper resource limits in Kubernetes

### Kubernetes Orchestration
- ✅ Declarative infrastructure (YAML)
- ✅ Rolling updates strategy
- ✅ Liveness and readiness probes
- ✅ Resource requests and limits
- ✅ Service discovery

### Automation
- ✅ Zero-touch deployment on master branch
- ✅ Workflow run on both push and manual trigger
- ✅ Artifact management for build traceability
- ✅ Automated test reporting

---

## Questions to Expect (and Answers)

### Q1: Why do you need CodeQL if you have Checkstyle?
**A**: Checkstyle validates code *style* (formatting, naming conventions). CodeQL detects *security vulnerabilities* (SQL injection, buffer overflows, XSS). They serve different purposes.

### Q2: What happens if Dependency-Check finds a vulnerability?
**A**: The build fails (gate). The team must either:
- Update the vulnerable dependency to a patched version
- Find an alternative library
- Explicitly suppress if false positive (documented)

### Q3: Why scan container images if code is already scanned?
**A**: Code scanning checks *application code*. Container scanning checks *OS libraries and base image*. A vulnerable Java library or OS package can compromise security.

### Q4: What's the difference between liveness and readiness probes?
**A**: 
- **Liveness**: Is the pod alive? If not, kill and restart it
- **Readiness**: Is the pod ready to serve traffic? If not, remove from load balancer

### Q5: Why use Kubernetes if Docker works fine?
**A**: Docker runs one container. Kubernetes:
- Manages multiple containers (replicas)
- Handles failures (auto-restart)
- Enables scaling
- Manages networking and storage
- Used in production environments

---

## Conclusion

This pipeline demonstrates a complete, production-grade CI/CD implementation that:
- ✅ Automates builds and tests
- ✅ Enforces code quality standards
- ✅ Integrates comprehensive security scanning
- ✅ Containerizes applications consistently
- ✅ Orchestrates deployments with Kubernetes
- ✅ Provides clear visibility into the entire process

Each stage has a clear purpose, contributing to faster delivery, higher quality, and better security.
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
