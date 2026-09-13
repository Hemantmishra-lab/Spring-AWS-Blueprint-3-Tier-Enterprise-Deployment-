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
