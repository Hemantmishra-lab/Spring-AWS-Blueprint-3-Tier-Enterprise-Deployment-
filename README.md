
````markdown
# Spring AWS Blueprint – 3-Tier Enterprise Deployment

A production-oriented **3-Tier Spring Boot deployment architecture on AWS**, designed to demonstrate secure networking, high availability, containerized application deployment, and managed database integration.

The project focuses on applying real-world **Cloud, DevOps, Networking, Security, and Backend Architecture** practices to a Spring Boot REST API.

---

## 🏗️ Architecture Overview

The application follows a traditional **3-Tier Architecture**:

```text
                         ┌──────────────────────┐
                         │       Internet       │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   Application Load   │
                         │       Balancer       │
                         └──────────┬───────────┘
                                    │
                         ┌──────────▼───────────┐
                         │   Private Subnet     │
                         │                      │
                         │  Spring Boot APIs    │
                         │  Docker Containers   │
                         │                      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   Database Subnet    │
                         │                      │
                         │      AWS RDS         │
                         │  PostgreSQL / MySQL  │
                         │                      │
                         └──────────────────────┘

              ┌───────────────────────────────┐
              │        Bastion Host           │
              │     Administrative Access     │
              └───────────────┬───────────────┘
                              │
                              ▼
                     Private Application Tier
````

---

## 🚀 Key Features

### ☁️ Cloud Architecture

* Designed a **custom AWS VPC** for isolated network architecture.
* Implemented separate **public and private subnets**.
* Structured the environment into:

  * Public Tier
  * Application Tier
  * Database Tier
* Designed the infrastructure to support **secure and highly available application deployment**.
* Used AWS networking components to control communication between application layers.

### 🌐 Traffic Routing

* Deployed an **Application Load Balancer (ALB)** to handle incoming HTTP traffic.
* Distributed requests across Spring Boot application instances.
* Kept application servers inside **private subnets** instead of exposing them directly to the internet.
* Used health checks to route traffic only toward available application instances.

### 🔐 Network Security

* Implemented **AWS Security Groups** with controlled inbound and outbound rules.
* Restricted database access so that the RDS instance is reachable only from the application tier.
* Used a **Bastion Host** as a controlled entry point for administrative access to private instances.
* Avoided direct public access to internal application and database resources.
* Followed the principle of **least-privilege network access**.

### 🗄️ Database & Persistence

* Provisioned an **AWS RDS managed relational database**.
* Supported PostgreSQL/MySQL based on deployment configuration.
* Integrated the database with Spring Boot using:

  * Spring Data JPA
  * Hibernate
  * JDBC
* Kept database resources inside **isolated/private subnets**.
* Separated application and database responsibilities according to 3-tier architecture principles.

### 🐳 Containerized Deployment

* Containerized the Spring Boot REST API using **Docker**.
* Created repeatable application deployment using Docker images.
* Deployed containerized Spring Boot services on **AWS EC2**.
* Used environment-based configuration for database and deployment-specific settings.
* Reduced dependency differences between development and deployment environments.

---

## 🛠️ Technology Stack

### Backend

* Java
* Spring Boot
* Spring Data JPA
* Hibernate
* REST APIs

### Cloud – AWS

* Amazon VPC
* Public & Private Subnets
* Application Load Balancer (ALB)
* Amazon EC2
* Amazon RDS
* Security Groups
* Bastion Host
* Internet Gateway
* Route Tables

### DevOps & Infrastructure

* Docker
* Git
* GitHub

### Database

* PostgreSQL / MySQL

---

## 📁 Project Structure

```text
spring-aws-blueprint/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── ...
│   │   └── resources/
│   │       ├── application.properties
│   │       └── ...
│   │
│   └── test/
│
├── Dockerfile
├── pom.xml
├── .gitignore
└── README.md
```

---

## 🔄 Request Flow

A typical application request follows this path:

```text
Client
   │
   ▼
Internet Gateway
   │
   ▼
Application Load Balancer
   │
   ▼
Private Subnet
   │
   ├── Spring Boot Instance
   │       │
   │       ▼
   │   Docker Container
   │
   ▼
AWS RDS
(PostgreSQL / MySQL)
```

### Administrative Access

```text
Administrator
      │
      ▼
Bastion Host
(Public Subnet)
      │
      ▼
Private EC2 Instance
(Application Tier)
```

The Bastion Host provides a controlled administrative path to resources that are not directly exposed to the public internet.

---

## 🔒 Security Model

The infrastructure follows a layered security approach:

```text
Internet
   │
   ▼
   ALB
   │
   │  Allowed Application Traffic
   ▼
Application Tier
   │
   │  Database Port Only
   ▼
Database Tier
```

### Security Group Strategy

| Component       | Allowed Access                  |
| --------------- | ------------------------------- |
| ALB             | Internet → HTTP/HTTPS           |
| Application EC2 | ALB → Application Port          |
| RDS             | Application EC2 → Database Port |
| Bastion Host    | Admin → SSH                     |
| Private EC2     | Bastion → SSH                   |

This prevents unnecessary direct communication between network layers.

---

## 🐳 Docker Deployment

Build the Spring Boot application:

```bash
mvn clean package
```

Build the Docker image:

```bash
docker build -t spring-aws-blueprint .
```

Run the container:

```bash
docker run -d \
  --name spring-app \
  -p 8080:8080 \
  spring-aws-blueprint
```

Check running containers:

```bash
docker ps
```

View application logs:

```bash
docker logs spring-app
```

---

## ⚙️ Configuration

Application-specific configuration should be supplied through environment variables or deployment configuration rather than hardcoding credentials.

Example:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
```

Example environment variables:

```bash
DB_URL=jdbc:mysql://<rds-endpoint>:3306/<database>
DB_USERNAME=<username>
DB_PASSWORD=<password>
```

> Never commit database credentials, private keys, `.env` files, or AWS secrets to GitHub.

---

## 📊 Deployment Architecture

The deployment separates infrastructure responsibilities into three logical layers:

### 1. Presentation / Traffic Layer

Responsible for receiving and distributing external traffic.

```text
Internet
   ↓
Application Load Balancer
```

### 2. Application Layer

Responsible for application business logic and REST API processing.

```text
Private EC2
   ↓
Docker
   ↓
Spring Boot REST API
```

### 3. Data Layer

Responsible for persistent application data.

```text
Private RDS
   ↓
PostgreSQL / MySQL
```

---

## 🎯 Project Objectives

This project was built to gain practical experience with:

* AWS VPC architecture
* Public vs private subnet design
* 3-tier application architecture
* Application Load Balancing
* EC2-based application deployment
* AWS RDS integration
* Network-level security
* Bastion Host architecture
* Security Group configuration
* Docker-based deployment
* Spring Boot cloud deployment
* Secure database connectivity
* Production-oriented infrastructure design

---

## 📈 Future Improvements

Possible extensions to the architecture include:

* Auto Scaling Groups for application instances
* HTTPS using AWS Certificate Manager
* Route 53 DNS integration
* AWS Secrets Manager for credential management
* CloudWatch monitoring and centralized logging
* CI/CD pipeline using GitHub Actions
* Infrastructure as Code using Terraform or AWS CloudFormation
* Redis caching
* Amazon ElastiCache
* Container orchestration using Amazon ECS/EKS
* WAF integration for application-layer protection

---

## 📸 Infrastructure Screenshots

Add screenshots demonstrating the actual AWS implementation:

```text
docs/
├── vpc.png
├── subnets.png
├── route-tables.png
├── security-groups.png
├── load-balancer.png
├── ec2.png
├── rds.png
└── docker.png
```

Example:

```markdown
![AWS VPC Architecture](docs/vpc.png)

![Application Load Balancer](docs/load-balancer.png)

![EC2 Deployment](docs/ec2.png)

![RDS Configuration](docs/rds.png)
```

---

## 🧠 Architecture Principles Demonstrated

* Separation of concerns
* Network isolation
* Least-privilege access
* Defense-in-depth security
* Stateless application deployment
* Horizontal traffic distribution
* Managed database services
* Containerized application delivery
* Repeatable deployment practices

---

## 📌 Project Status

**Status:** Completed / Deployment Blueprint

**Development Period:** July 2026 – September 2026

**Repository:** GitHub

---

## 👨‍💻 Author

**Hemant Mishra**

B.Tech Computer Science Engineering

Focused on:

```text
Java Backend Development
Spring Boot
Microservices
Cloud Architecture
AWS
Docker
Distributed Systems
DevOps
```

---

## ⭐ If You Found This Project Useful

Feel free to explore the repository, review the architecture, and use it as a reference for building secure Spring Boot applications on AWS.

```

```
