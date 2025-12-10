# End-to-End CI/CD Pipeline for Containerized Web Application

Personal DevOps project to practice building a full CI/CD pipeline using **Jenkins**, **Docker**, and **Kubernetes**.

The pipeline takes a small **FastAPI** web application from **GitHub** → **Jenkins** → **Docker Hub** → **Kubernetes cluster (Docker Desktop)** with a rolling update deployment.

---

## 🔧 Tech Stack

- **Application**
  - Python, FastAPI
  - Pytest for unit tests

- **DevOps / Platform**
  - Git, GitHub
  - Jenkins (Pipeline as Code, `Jenkinsfile`)
  - Docker, Docker Hub
  - Kubernetes (Docker Desktop Kubernetes)
  - kubectl
  - Windows 11, Git Bash / CMD

---

## 🧱 Architecture Overview

```text
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
     Build & Push      | Docker Image
                       v
              +----------------+
              |   Docker Hub   |
              | (Image Registry) 
              +--------+-------+
                       |
                       | kubectl set image
                       v
         +-------------------------------+
         |      Kubernetes Cluster       |
         | (Docker Desktop Kubernetes)   |
         +-------------------------------+
                |               |
         Deployment         Service (NodePort)
                |               |
                +-------  http://localhost:30080
