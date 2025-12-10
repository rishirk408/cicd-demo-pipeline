
# End-to-End CI/CD Pipeline for Containerized Web Application

Personal DevOps project to practice building a **complete CI/CD pipeline** using:

- **Jenkins** for CI/CD orchestration  
- **Docker** for containerization  
- **Kubernetes** (Docker Desktop) for deployment & rolling updates  
- **Docker Hub** as the image registry  
- **FastAPI** + **pytest** for a simple web API and tests  

> Code → GitHub → Jenkins → Docker Hub → Kubernetes (Docker Desktop)

---

## 🎯 Project Goals

- Build a small but realistic **DevOps-style project** for portfolio / CV.
- Take a simple web API from **source code** all the way to a **Kubernetes deployment**.
- Use **Pipeline as Code** (`Jenkinsfile`) and **Kubernetes manifests** (`deployment.yaml`, `service.yaml`).
- Practice **rolling updates**, **image tagging**, and **automated testing**.

---

## 🧱 Repository Structure

cicd-demo-pipeline/
├── main.py                 # FastAPI application
├── requirements.txt        # Python dependencies
├── tests/
│   └── test_main.py        # pytest unit test for the root endpoint
├── Dockerfile              # Docker image for the app
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment (2 replicas)
│   └── service.yaml        # Kubernetes Service (NodePort)
└── Jenkinsfile             # Jenkins CI/CD pipeline definition
````


## 🔧 Tech Stack

**Application**

* Python 3.x
* FastAPI
* Uvicorn
* pytest

**DevOps / Platform**

* Git, GitHub (`rishirk408`)
* Jenkins (Pipeline)
* Docker, Docker Hub (`HRISHIKESH`)
* Kubernetes (Docker Desktop Kubernetes)
* kubectl
* Windows (Git Bash / CMD)


## 🧾 Application Overview

* A small **FastAPI** application with a `/` endpoint.
* Returns a JSON response like:

{"message": "Hello from CI/CD pipeline!"}


* A unit test (`tests/test_main.py`) uses FastAPI’s `TestClient` and `pytest` to:

  * Call `/`
  * Assert HTTP 200
  * Assert the response message.


## 🧠 High-Level Architecture

           +----------------+
           |    GitHub      |
           |  (Source Code) |
           +--------+-------+
                    |
                    |  Webhook / Poll SCM
                    v
           +------------------------+
           |        Jenkins         |
           |  (CI/CD Pipeline)      |
           +-----------+------------+
                       |
       Build & Push    | Docker Image
                       v
              +----------------+
              |  Docker Hub    |
              |  (HRISHIKESH)  |
              +--------+-------+
                       |
                       | kubectl set image
                       v
         +-------------------------------+
         |      Kubernetes Cluster       |
         |  (Docker Desktop Kubernetes)  |
         +-------------------------------+
                |               |
         Deployment         Service (NodePort)
                |               |
                +-------  http://localhost:30080


## ✅ Features Implemented

* FastAPI web API with a simple root endpoint.
* Unit testing with pytest.
* **Dockerized application** with a slim Python base image.
* **Kubernetes deployment**:

  * 2 replicas of the app
  * `NodePort` service exposing the app on `localhost:30080`
* **Jenkins pipeline** that:

  1. Checks out code from GitHub
  2. Installs dependencies & runs tests
  3. Builds a Docker image
  4. Tags image with Jenkins build number + `latest`
  5. Pushes to Docker Hub (`HRISHIKESH/cicd-demo-api`)
  6. Uses `kubectl` to perform a rolling update on the Kubernetes Deployment

---

## 🧩 Prerequisites

On the machine where you run everything (Windows):

* **Python 3.x**
* **Git**
* **Docker Desktop** (with **Kubernetes enabled**)
* **Java 17+** (for Jenkins)
* **kubectl** (usually installed/managed by Docker Desktop Kubernetes)
* **Jenkins** (run via `jenkins.war`)

---

## 🛠 Local Development (without Docker/Kubernetes)

### 1. Clone the repository


git clone https://github.com/rishirk408/cicd-demo-pipeline.git
cd cicd-demo-pipeline


### 2. Set up virtual environment & install dependencies

**Git Bash (Windows):**

python -m venv venv
source venv/Scripts/activate

pip install -r requirements.txt
```

**CMD (Windows):**

python -m venv venv
venv\Scripts\activate

pip install -r requirements.txt
```

---

### 3. Run tests

pytest

You should see something like:

tests/test_main.py .                                         [100%]
1 passed in X.XXs

---

### 4. Run the app locally

uvicorn main:app --host 0.0.0.0 --port 8000

Open:

* [http://localhost:8000/](http://localhost:8000/)

You’ll see the JSON response from the API.

---

## 🐳 Docker

### 1. Build the image

From the project root:

docker build -t cicd-demo-api:local .

---

### 2. Run the container


docker run --rm -p 8000:8000 cicd-demo-api:local


Open in browser:

* [http://localhost:8000/](http://localhost:8000/)

---

### 3. Push the image to Docker Hub

Docker Hub username: **`HRISHIKESH`**


docker tag cicd-demo-api:local HRISHIKESH/cicd-demo-api:v1
docker tag cicd-demo-api:local HRISHIKESH/cicd-demo-api:latest

docker push HRISHIKESH/cicd-demo-api:v1
docker push HRISHIKESH/cicd-demo-api:latest


You should now see the `cicd-demo-api` repository and tags (`v1`, `latest`) in Docker Hub under the `HRISHIKESH` account.

---

## ☸️ Kubernetes (Docker Desktop Kubernetes)

### 1. Ensure Kubernetes is enabled in Docker Desktop

* Open **Docker Desktop → Settings → Kubernetes**
* Tick **“Enable Kubernetes”**
* Apply & wait for it to become ready

Verify in terminal:


kubectl get nodes


You should see the single Docker Desktop node.

---

### 2. Apply the Deployment and Service manifests

From project root:


kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml


Check resources:


kubectl get pods
kubectl get svc


You should see:

* Pods with names like `cicd-demo-api-xxxxx   Running`
* Service `cicd-demo-api` of type `NodePort` with `30080:...`

---

### 3. Access the service

Open in browser:

* [http://localhost:30080/](http://localhost:30080/)

or via curl:


curl http://localhost:30080/


This hits the Kubernetes Service, which load-balances between the pods.

---

## 🧬 Jenkins CI/CD Pipeline

The pipeline is defined in `Jenkinsfile` and uses **Pipeline as Code**.

### Stages

1. **Checkout**

   * Uses `checkout scm` to pull code from the GitHub repository configured in the Jenkins job.

2. **Unit Tests**

   * Installs dependencies using `pip install -r requirements.txt`
   * Runs `pytest` to execute unit tests.

3. **Build Docker Image**

   * Builds a Docker image with tag:
     `HRISHIKESH/cicd-demo-api:${BUILD_NUMBER}`
   * Also tags the image as:
     `HRISHIKESH/cicd-demo-api:latest`

4. **Push Docker Image**

   * Logs into Docker Hub using Jenkins credentials (`dockerhub-creds`)
   * Pushes both the build-number tag and `latest` tag to Docker Hub.

5. **Deploy to Kubernetes**

   * Runs:


     kubectl set image deployment/cicd-demo-api api=HRISHIKESH/cicd-demo-api:${BUILD_NUMBER} --record
     kubectl rollout status deployment/cicd-demo-api


   * This triggers a **rolling update** in Kubernetes and waits for the rollout to finish.



### Jenkinsfile (Conceptual Overview)

In the actual `Jenkinsfile`:

* `DOCKER_IMAGE` is set to:


  DOCKER_IMAGE = "HRISHIKESH/cicd-demo-api"


* Docker Hub credentials are stored with ID:


  dockerhub-creds


* On Windows, pipeline steps use `bat """ ... """` blocks to run:

  * `pip install` and `pytest`
  * `docker build`, `docker tag`, `docker push`
  * `kubectl set image`, `kubectl rollout status`

---

### Jenkins Setup (Summary)

* Jenkins is run via:


  java -jar jenkins.war --httpPort=8080


* Pipeline job is configured as:

  * Type: **Pipeline**
  * Definition: **Pipeline script from SCM**
  * SCM: **Git**
  * Repo URL: `https://github.com/rishirk408/cicd-demo-pipeline.git`
  * Branch: `main`

* Docker Hub credentials:

  * Added in **Manage Jenkins → Credentials** as:

    * Kind: `Username with password`
    * ID: `dockerhub-creds`
    * Username: `HRISHIKESH`
    * Password: (Docker Hub password)

---

## 🧪 Typical CI/CD Flow

1. Code changes are pushed to **GitHub** (`rishirk408/cicd-demo-pipeline`).
2. Jenkins job is triggered (manually, via poll, or via webhook).
3. Jenkins:

   * Checks out the new code
   * Installs dependencies and runs unit tests
   * Builds a new Docker image and tags it with the Jenkins build number
   * Pushes the image to Docker Hub under `HRISHIKESH/cicd-demo-api`
   * Updates the Kubernetes Deployment to use the new image
4. Kubernetes:

   * Pulls the new image from Docker Hub
   * Gradually replaces old pods with new pods (rolling update)
5. User:

   * Continues to call `http://localhost:30080/`
   * Gets responses from the newly deployed version with no manual steps.

---

## 🧩 Troubleshooting Notes

* If `docker` is not recognised inside Jenkins:

  * Make sure Docker Desktop is running and the Docker CLI is on PATH for the user running Jenkins.

* If `kubectl` is not recognised in Jenkins:

  * Ensure `kubectl` is installed and on PATH.
  * Ensure Jenkins is running under a user that has access to the kubeconfig used by Docker Desktop.

* If the app is not available on `localhost:30080`:

  * Check:


    kubectl get pods
    kubectl logs <pod-name>


  * Ensure the Service is of type `NodePort` and uses `targetPort: 8000`.

---

## 🔮 Possible Future Improvements

* Add **code quality checks** (e.g. Flake8, Black) as a separate Jenkins stage.
* Introduce **environment-based deployments** (dev/stage/prod) using namespaces.
* Replace raw YAML with **Helm charts**.
* Move from Docker Desktop Kubernetes to **AWS EKS** and from Docker Hub to **ECR**.
* Add health checks / liveness & readiness probes for better resiliency.

---

## 📜 License

This project is for learning and portfolio purposes.
You may adapt or extend it for your own personal use or learning.

---

```
```
