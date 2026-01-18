# Demo Spring Boot CI/CD Project

## Project Overview

This is a minimal Java Spring Boot application demonstrating a complete CI/CD pipeline using GitHub Actions, Docker, and Kubernetes. The application exposes a single REST endpoint for health checks and is designed for academic evaluation.

## Application Details

- **Framework**: Spring Boot 3.2.0
- **Java Version**: 17
- **Build Tool**: Maven
- **Endpoint**: `GET /health` returns "OK"

## Technologies Used

- **Java 17** - Programming language
- **Spring Boot** - Application framework
- **Maven** - Build and dependency management
- **Docker** - Containerization
- **Kubernetes** - Container orchestration
- **GitHub Actions** - CI/CD automation
- **Kind** - Local Kubernetes cluster for testing
- **DockerHub** - Container registry

## CI Pipeline

The Continuous Integration pipeline (`ci.yml`) is triggered on every push to the master branch or manually via workflow dispatch.

### CI Pipeline Steps:
1. **Checkout Code** - Retrieves the repository code
2. **Setup Java 17** - Configures the Java environment with Temurin distribution
3. **Run Tests** - Executes Maven tests to validate code quality
4. **Build JAR** - Compiles and packages the application into a JAR file
5. **Build Docker Image** - Creates a Docker container image from the JAR
6. **Login to DockerHub** - Authenticates using stored credentials
7. **Push Image** - Uploads the Docker image to DockerHub registry

The CI pipeline ensures code quality through automated testing and prepares the application for deployment by creating a container image.

## CD Pipeline

The Continuous Deployment pipeline (`cd.yml`) is triggered automatically after the CI pipeline completes successfully.

### CD Pipeline Steps:
1. **Checkout Code** - Retrieves the latest code
2. **Setup kubectl** - Installs Kubernetes command-line tool
3. **Create Kind Cluster** - Provisions a local Kubernetes cluster for testing
4. **Update Deployment** - Replaces placeholder with actual DockerHub username
5. **Apply Manifests** - Deploys the application and service to Kubernetes
6. **Verify Deployment** - Confirms the deployment rolled out successfully
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
