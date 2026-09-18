# Spring AWS Blueprint – 3-Tier Enterprise Deployment

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.0-brightgreen.svg)
![Java](https://img.shields.io/badge/Java-21-blue.svg)
![AWS](https://img.shields.io/badge/AWS-Cloud_Architecture-orange.svg)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue.svg)

## 📌 Project Overview
This repository contains the infrastructure blueprint and backend codebase for a **highly available, production-ready 3-tier enterprise application** deployed on Amazon Web Services (AWS). It bridges the gap between modern Spring Boot backend engineering and secure, scalable cloud DevOps practices.

The core objective is to host a containerized Spring Boot application within a secure custom Virtual Private Cloud (VPC), ensuring strict network isolation, automated load balancing, and resilient database management.

---

## 🏗️ System Architecture

The infrastructure follows a classic, secure 3-tier cloud design:

```mermaid
graph TD
    Users[Internet Users] --> Route53[AWS Route 53 / Cloudflare DNS]
    Route53 --> ALB[AWS Application Load Balancer]

    subgraph VPC [AWS VPC Deployment]
        direction TB
        
        subgraph PublicSubnet [Public Subnet - Internet Facing]
            ALB
            Bastion[Bastion Host / Jump Box]
            APIGW[Public EC2 - API Gateway]
        end

        subgraph PrivateSubnet [Private Subnet - Backend App Tier]
            SBA[Spring Boot Service A]
            SBB[Spring Boot Service B]
            KafkaRedis[Apache Kafka / Redis Cache]
        end

        subgraph IsolatedSubnet [Isolated Subnet - Database Tier]
            RDS[(AWS RDS - PostgreSQL/MySQL)]
            S3[(AWS S3 / MinIO Storage)]
        end
    end

    ALB --> APIGW
    APIGW --> SBA
    APIGW --> SBB
    SBA --> KafkaRedis
    SBB --> KafkaRedis
    SBA --> RDS
    SBA --> S3
    SBB --> RDS
    SBB --> S3
    
    Bastion -.->|Admin SSH Access| PrivateSubnet

🧰 Tech Stack & Tools
Backend Framework: Java 17, Spring Boot 3.x, Spring Data JPA, Spring Actuator, Prometheus

Database & ORM: AWS RDS (PostgreSQL 15), Hibernate, PostgreSQL Driver

Cloud Infrastructure (AWS): VPC, Public & Private Subnets, Internet Gateway, NAT Gateway, Security Groups, EC2, RDS, ALB, Bastion Host

DevOps & Observability: Docker, Docker Hub (hemantmishra1978/spring-aws-blueprint), GitHub Actions, Actuator Prometheus Endpoint

🚀 Deployment & Operational Workflow
1. Build & Containerization
Bash
# Compile and package Spring Boot executable JAR
mvn clean package -DskipTests

# Build Docker image locally
docker build -t hemantmishra1978/spring-aws-blueprint:latest .

# Run container locally with PostgreSQL connection
docker run -d -p 8080:8080 --name spring-app hemantmishra1978/spring-aws-blueprint:latest
2. AWS Cloud Provisioning Sequence
Network Layer Setup: Create Custom VPC (10.0.0.0/16), 6 subnets across 2 AZs (2 Public, 2 Private App, 2 Private DB), Internet Gateway, and NAT Gateway.

Security & Access Layer: Configure Security Groups:

app-sg: Ingress Port 8080 (App) and Port 22 (Bastion Access).

db-sg: Ingress Port 5432 allowed only from app-sg.

Database Tier: Provision Multi-AZ AWS RDS PostgreSQL instance in isolated DB Subnets.

Compute Tier: Launch Ubuntu EC2 instances in private/public subnets, initialize Docker runtime, and pull application image.

Load Balancing & Observability: Attach EC2 instances to Application Load Balancer target groups and verify health metrics via /actuator/prometheus.

⚙️ Automated CI/CD Pipeline
The project uses a GitHub Actions workflow (.github/workflows/ci-cd.yml) to ensure continuous integration:

Trigger: Pushes or Pull Requests targeting the main branch.

Build Phase: Sets up JDK 17 environment and compiles application via Maven.

Container Publishing Phase: Authenticates with Docker Hub and publishes the latest tagged image (hemantmishra1978/spring-aws-blueprint:latest).

📈 Observability & API Verification
Prometheus Metrics Endpoint: http://<EC2-PUBLIC-IP-OR-ALB>:8080/actuator/prometheus

Health Check Endpoint: http://<EC2-PUBLIC-IP-OR-ALB>:8080/actuator/health

Live Demonstration & Verification
Screen recording and live AWS deployment evidence featuring API Postman tests, RDS database persistence check, and VPC Resource Map visual architecture.
