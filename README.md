# CI/CD Pipeline - Task-4

## Jenkins CI/CD Pipeline for React + Spring Boot Project

This project uses a Jenkins Declarative Pipeline to automate the build → test → scan → package → deploy lifecycle for a full-stack application consisting of a React frontend and Spring Boot backend. Docker is used for containerization, and the application is deployed to a remote Docker host using TLS.

---

### Prerequisites

- Jenkins up and running with required plugins:
  - Pipeline
  - Docker Pipeline
  - NodeJS
  - SonarQube Scanner
- Global tools configured:
  - JDK 17 (`jdk17`)
  - Maven 3 (`maven3`)
  - NodeJS 18 (`node18`)
- Jenkins credentials:
  - DockerHub credentials ID: `dockerhub-creds`
  - SonarQube token ID: `sonar-taken`
- SonarQube server named: `SonarQube-Local`
- TLS-enabled Docker daemon accessible at: `tcp://your.deploy.server.com:2376`
- Jenkins node has access to TLS certificates at `/path/to/docker/certs`

---

### Pipeline Configuration (Environment Variables)

| Variable | Description |
|---------|-------------|
| `SONARQUBE_SERVER` | SonarQube instance name configured in Jenkins |
| `DOCKER_IMAGE_FRONTEND` | Docker image name for the frontend |
| `DOCKER_IMAGE_BACKEND` | Docker image name for the backend |
| `DOCKER_REGISTRY_CREDENTIALS` | Jenkins credentials ID for DockerHub |
| `DOCKER_HOST` | TLS-enabled remote Docker daemon address |
| `DOCKER_TLS_VERIFY` | Docker TLS verification flag (`1`) |
| `DOCKER_CERT_PATH` | Path to Docker TLS certs on Jenkins node |
| `NODE_OPTIONS` | Node.js memory flag for large builds |

---

### Pipeline Stages Explained

#### 1. Checkout
- Pulls the latest source code from GitHub (branch: `master`).

#### 2. Build Frontend & Backend (Parallel)
- **Frontend**
  - Uses `npm ci` to install dependencies
  - Restores `node_modules` cache using `stash/unstash`
  - Builds frontend with `npm run build`
  - Runs tests via `npx jest` and generates JUnit XML report
- **Backend**
  - Uses Maven to clean, build, and run unit tests (`mvn clean verify`)

#### 3. SonarQube Scan (Backend)
- Uses SonarQube to run static code analysis on the Spring Boot app.
- Uses token-based authentication with `sonar-taken` credential.

#### 4. Archive Artifacts and Test Reports
- Archives:
  - Backend JAR files from `target/`
  - Frontend build artifacts
- Publishes:
  - JUnit reports from Jest (frontend)
  - Surefire reports from Maven (backend)

#### 5. Build Docker Images
- Builds and tags Docker images for:
  - Frontend: `${DOCKER_IMAGE_FRONTEND}:${BUILD_NUMBER}`
  - Backend: `${DOCKER_IMAGE_BACKEND}:${BUILD_NUMBER}`

#### 6. Push Docker Images to DockerHub
- Logs into DockerHub using credentials
- Pushes:
  - `:BUILD_NUMBER` tagged images
  - `:latest` tagged images

#### 7. Deploy to Remote Docker Host
- Connects to remote Docker host using TLS
- Stops and removes existing containers (if running)
- Runs updated containers:
  - Backend container on `9090:8080`
  - Frontend container on `80:80`

---

### Post Actions

- Always cleans the workspace (`cleanWs`)
- Logs success or failure
- Failure block available for future notification integration

---

### Expected Success Output

