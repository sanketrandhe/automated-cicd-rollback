# Automated CI/CD Pipeline with Health Checks & Rollback on AWS

![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazonaws&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red?logo=jenkins&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containers-blue?logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-Flask-yellow?logo=python&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-Webhooks-black?logo=github&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Amazon%20Linux%202023-black?logo=linux&logoColor=white)

An AWS-based CI/CD project that automates application deployment using Jenkins, Docker, GitHub Webhooks, and Amazon EC2, with application health checks and automatic rollback on deployment failure.

## Project Overview

This project demonstrates an automated CI/CD workflow for a Python Flask application. Whenever code is pushed to GitHub, a webhook triggers Jenkins to retrieve the latest source code, validate the Python application, build a versioned Docker image, and push it to Docker Hub.

Jenkins then deploys the new image to a separate EC2 deployment server over SSH. After deployment, an HTTP health check verifies the application. If the health check fails, the pipeline restores the previously running Docker image and its application version.

## Architecture

**Infrastructure:**
- AWS Region: Mumbai (ap-south-1)
- EC2 Instance 1: Jenkins Controller
- EC2 Instance 2: Application Deployment Server
- Operating System: Amazon Linux 2023

### CI/CD Workflow

```text
Developer
    |
    | Git Push
    v
GitHub Repository
    |
    | Webhook Trigger
    v
Jenkins Controller (EC2)
    |
    | Checkout Source Code
    | Python Syntax Validation
    | Build Docker Image
    | Push Versioned Image
    v
Docker Hub
    |
    | SSH Deployment
    v
Deployment Server (EC2)
    |
    | Pull New Image
    | Run Flask Container
    | HTTP Health Check
    |
    +---- Healthy? ---- Yes ---> Deployment Successful
    |
    +---- No ------------------> Automatic Rollback
                                  to Previous Image
```

## Technologies Used

| Category | Technologies |
|---|---|
| Cloud | AWS EC2 |
| CI/CD | Jenkins, Jenkins Pipeline |
| Source Control | Git, GitHub |
| Automation | GitHub Webhooks |
| Containers | Docker |
| Image Registry | Docker Hub |
| Application | Python, Flask |
| Deployment | SSH, Bash |
| Validation | Python Compilation, cURL Health Check |
| Operating System | Amazon Linux 2023 |

## Pipeline Stages

1. **Checkout:** Jenkins retrieves the latest source code from GitHub.
2. **Test Application:** Python syntax is validated using `python3 -m py_compile`.
3. **Build and Push Docker Image:** Jenkins builds a Docker image tagged with the Jenkins build number and pushes it to Docker Hub.
4. **Deploy and Health Check:** Jenkins connects to the deployment EC2 instance over SSH, pulls the new image, and starts the Flask application container.
5. **Automatic Rollback:** If the application health check fails, Jenkins restores the previously running Docker image and its application version.

## Key Features

- GitHub webhook-based pipeline triggering.
- Automated source code checkout and Python syntax validation.
- Docker image versioning using Jenkins build numbers.
- Automated Docker Hub image publishing.
- Remote deployment to a separate EC2 instance using SSH.
- Application health verification through an HTTP endpoint.
- Automatic rollback to the previous image if deployment health validation fails.
- Preservation of the previous application version during rollback.

## Application Health Check

The Flask application exposes a health endpoint:

```bash
curl http://localhost:5000/health
```

Example healthy response:

```json
{
  "status": "healthy",
  "version": "6"
}
```

## Rollback Testing

The rollback mechanism was tested by deploying an intentionally unhealthy application version.

**Observed result:**
- The new deployment failed the application health check.
- Jenkins detected the failure.
- The previously running Docker image and application version were restored.
- Jenkins reported `Rollback completed`.
- The failed deployment build was marked as `FAILURE`, as expected.

A subsequent healthy deployment completed successfully.

## Project Structure

```text
automated-cicd-rollback/
│
├── app/
│   └── app.py
│
├── screenshots/
│
├── .dockerignore
├── Dockerfile
├── Jenkinsfile
├── README.md
└── requirements.txt
```
## Errors Faced & Solutions

1. **GitHub Webhook Not Triggering Jenkins:** Corrected the webhook configuration and verified the successful HTTP 200 response.

2. **SSH Deployment Issues:** Checked SSH key permissions, Jenkins credentials, and connectivity between the Jenkins controller and deployment server.

3. **Rollback Version Issue:** Initially, rollback restored the previous Docker image but not its application version. Updated the Jenkinsfile to restore both the previous image and `APP_VERSION`.

## Project Resources

- **GitHub Repository:** https://github.com/sanketrandhe/automated-cicd-rollback
- **Docker Hub:** https://hub.docker.com/r/sanketrandhe/automated-cicd-rollback

## Screenshots

See the [`screenshots/`](screenshots/) folder for project implementation evidence, including AWS infrastructure, Jenkins configuration, GitHub webhook, Docker image publishing, deployment, and application health checks.

## Key Learning Outcomes

- Implemented an end-to-end CI/CD pipeline using Jenkins.
- Integrated GitHub Webhooks with Jenkins.
- Automated Docker image build, versioning, and publishing.
- Performed remote application deployment on AWS EC2.
- Implemented health-check-based deployment validation and automatic rollback.
- Practiced troubleshooting across CI/CD, Docker, SSH, and AWS infrastructure.
