# Orchestrated Streaming App

This repository houses the source code and infrastructure configurations for a highly available, containerized streaming application built on the MERN stack. 

The project demonstrates a production-grade DevOps lifecycle, utilizing **Jenkins** for Continuous Integration and Continuous Deployment (CI/CD). The application components are containerized using **Docker**, stored in **Amazon ECR**, and deployed to a managed **Amazon EKS** (Elastic Kubernetes Service) cluster via **Helm** charts. 

**Key Infrastructure Features:**
* **Automated CI/CD:** Jenkins pipelines trigger on Git commits to build, push, and deploy Docker images.
* **Kubernetes Orchestration:** Helm is used to package and manage the deployment of the frontend and backend microservices on EKS.
* **Cloud-Native AWS Integrations:** Utilizes Amazon ECR for image registry and EC2 for hosting the build server.
* **Observability & ChatOps:** Real-time monitoring and centralized logging managed via Amazon CloudWatch, with SNS integrations for deployment alerts.
