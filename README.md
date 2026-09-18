

````
# Spring AWS Blueprint – 3-Tier Enterprise Deployment

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen.svg)
![Java](https://img.shields.io/badge/Java-17-blue.svg)
![AWS](https://img.shields.io/badge/AWS-Cloud%20Architecture-orange.svg)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue.svg)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue.svg)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-black.svg)

## 📌 Project Overview

**Spring AWS Blueprint** is a production-oriented reference architecture for deploying a containerized Spring Boot application on Amazon Web Services (AWS) using a secure and highly available **3-tier architecture**.

The project demonstrates how modern backend engineering practices can be combined with AWS cloud infrastructure, containerization, CI/CD automation, database management, networking, security, and observability.

The application is deployed within a custom AWS VPC with clearly separated network tiers:

- **Public Tier** – Application Load Balancer, Bastion Host, and API Gateway
- **Private Application Tier** – Containerized Spring Boot services
- **Isolated Database Tier** – Amazon RDS PostgreSQL and object storage

The architecture is designed to provide **network isolation, controlled access, scalability, high availability, and operational visibility**.

---

## 🏗️ System Architecture

The deployment follows a secure 3-tier AWS architecture distributed across multiple Availability Zones.

```mermaid
graph TD
    Users[Internet Users] --> DNS[Route 53 / Cloudflare DNS]
    DNS --> ALB[AWS Application Load Balancer]

    subgraph VPC[AWS VPC - 10.0.0.0/16]
        direction TB

        subgraph PublicSubnet[Public Subnets - Internet Facing]
            ALB
            Bastion[Bastion Host / Jump Box]
            APIGW[API Gateway / Public EC2]
        end

        subgraph PrivateSubnet[Private Application Subnets]
            SBA[Spring Boot Service A]
            SBB[Spring Boot Service B]
            KafkaRedis[Kafka / Redis]
        end

        subgraph IsolatedSubnet[Isolated Database Subnets]
            RDS[(Amazon RDS PostgreSQL)]
            S3[(Amazon S3 / MinIO)]
        end
    end

    ALB --> APIGW
    APIGW --> SBA
    APIGW --> SBB

    SBA --> KafkaRedis
    SBB --> KafkaRedis

    SBA --> RDS
    SBB --> RDS

    SBA --> S3
    SBB --> S3

    Bastion -.->|Administrative Access| PrivateSubnet
````

 ### Architecture Flow

 1. Internet traffic enters through **Route 53 / Cloudflare DNS**.
2. Requests are routed to the **AWS Application Load Balancer (ALB)**.
3. The ALB forwards traffic to the API Gateway layer.
4. The API Gateway routes requests to Spring Boot application instances running in private subnets.
5. Application services communicate with **Redis/Kafka** for caching and asynchronous processing.
6. Persistent application data is stored in **Amazon RDS PostgreSQL**.
7. Object-based data is stored in **Amazon S3**.
8. Administrative access to private resources is provided through a **Bastion Host**.
9. Security Groups restrict communication between the individual tiers.

---

 ## 🧰 Technology Stack

 ### Backend

 - Java 17
- Spring Boot 3.x
- Spring Data JPA
- Hibernate
- Spring Boot Actuator
- Prometheus Metrics

 ### Database

 - Amazon RDS
- PostgreSQL 15
- PostgreSQL JDBC Driver

 ### AWS Infrastructure

 - Amazon VPC
- Public and Private Subnets
- Internet Gateway
- NAT Gateway
- Security Groups
- Amazon EC2
- Application Load Balancer (ALB)
- Amazon RDS
- Amazon S3
- Bastion Host

 ### DevOps & Containerization

 - Docker
- Docker Hub
- GitHub Actions
- Maven
- CI/CD Automation

 ### Observability

 - Spring Boot Actuator
- Prometheus-compatible metrics
- Application health checks

---

 ## ☁️ AWS Infrastructure Design

 The infrastructure is organized across **two Availability Zones** to improve resilience and availability.

 ### Network Configuration

 | Component | Configuration |
| --- | --- |
| VPC CIDR | `10.0.0.0/16` |
| Availability Zones | 2 |
| Public Subnets | 2 |
| Private Application Subnets | 2 |
| Private Database Subnets | 2 |
| Internet Gateway | 1 |
| NAT Gateway | 1+ |
| Load Balancer | Application Load Balancer |
| Database | Amazon RDS PostgreSQL |

### Subnet Architecture

```
AWS VPC
│
├── Availability Zone A
│   ├── Public Subnet
│   │   ├── Application Load Balancer
│   │   └── Bastion Host
│   │
│   ├── Private App Subnet
│   │   └── Spring Boot Application
│   │
│   └── Private DB Subnet
│       └── RDS PostgreSQL
│
└── Availability Zone B
    ├── Public Subnet
    │   └── Application Load Balancer
    │
    ├── Private App Subnet
    │   └── Spring Boot Application
    │
    └── Private DB Subnet
        └── RDS PostgreSQL
```

---

 ## 🔐 Security Architecture

 Network access is controlled using AWS Security Groups.

 ### Application Security Group

 `app-sg`

 - Application Port: `8080`
- SSH Port: `22` restricted to administrative access
- Application traffic is controlled through the load-balancing layer.

 ### Database Security Group

 `db-sg`

 - PostgreSQL Port: `5432`
- Database access is permitted **only from the application Security Group**.
- The database is not directly exposed to the public internet.

 ### Network Isolation

 The architecture separates resources into:

 - Internet-facing public resources
- Private application resources
- Isolated database resources

 This minimizes unnecessary public exposure and follows the principle of **least-privilege network access**.

---

 ## 🐳 Docker & Application Containerization

 The Spring Boot application is packaged as an executable JAR and deployed as a Docker container.

 ### Build the Application

```
mvn clean package -DskipTests
```

 ### Build the Docker Image

```
docker build -t hemantmishra1978/spring-aws-blueprint:latest .
```

 ### Run the Container Locally

```
docker run -d \
  -p 8080:8080 \
  --name spring-app \
  hemantmishra1978/spring-aws-blueprint:latest
```

 The application will be available locally at:

```
http://localhost:8080
```

---

 ## 🚀 AWS Deployment Workflow

 The infrastructure is provisioned and configured in the following sequence:

 ### 1\. Network Layer

 Create the AWS networking infrastructure:

 - Custom VPC – `10.0.0.0/16`
- Two Availability Zones
- Two public subnets
- Two private application subnets
- Two private database subnets
- Internet Gateway
- NAT Gateway
- Route tables

 ### 2\. Security Layer

 Configure Security Groups to control communication between:

```
Internet
   ↓
ALB
   ↓
Application Tier
   ↓
Database Tier
```

 The database layer accepts PostgreSQL traffic only from the application tier.

 ### 3\. Database Layer

 Provision an Amazon RDS PostgreSQL database using a DB subnet group spanning the private database subnets.

 The database is configured to remain inaccessible directly from the public internet.

 ### 4\. Compute Layer

 Launch EC2 instances within the private application subnets.

 Each instance is configured with:

 - Docker runtime
- Application container
- Required environment variables
- Database connectivity
- Application health checks

 ### 5\. Load Balancing

 Configure an AWS Application Load Balancer with:

 - Public-facing listeners
- Target groups
- EC2 application instances
- Health checks

 Traffic is distributed across healthy application instances.

 ### 6\. Monitoring & Verification

 Spring Boot Actuator provides health and Prometheus-compatible metrics endpoints.

 Example endpoints:

```
/actuator/health
/actuator/prometheus
```

---

 ## ⚙️ CI/CD Pipeline

 The project uses **GitHub Actions** to automate the application build and container publishing process.

 ### Pipeline Workflow

```
Developer Push / Pull Request
            │
            ▼
      GitHub Actions
            │
            ▼
       Setup JDK 17
            │
            ▼
        Maven Build
            │
            ▼
     Docker Image Build
            │
            ▼
      Docker Hub Login
            │
            ▼
   Push Docker Image
            │
            ▼
hemantmishra1978/spring-aws-blueprint
```

 ### Pipeline Trigger

 The workflow is configured to run for:

 - Pushes to the `main` branch
- Pull requests targeting the `main` branch

 ### Build Process

 The CI/CD workflow:

 1. Checks out the source code.
2. Configures JDK 17.
3. Builds the Spring Boot application using Maven.
4. Builds the Docker image.
5. Authenticates with Docker Hub.
6. Publishes the container image.

---

 ## 📊 Observability & Health Monitoring

 The application uses **Spring Boot Actuator** to expose operational endpoints.

 ### Application Health

```
http://<ALB-DNS>/actuator/health
```

 Example response:

```
{
  "status": "UP"
}
```

 ### Prometheus Metrics

```
http://<ALB-DNS>/actuator/prometheus
```

 The Prometheus endpoint exposes application and JVM metrics that can be consumed by a monitoring system.

 > For production deployments, Actuator endpoints should be protected using appropriate authentication, authorization, network controls, and/or a dedicated monitoring path rather than exposing sensitive management endpoints publicly.

---

 ## 🔎 API & Database Verification

 The deployment can be validated using tools such as **Postman** and AWS Console.

 Verification includes:

 - REST API request/response validation
- Application health checks
- Load Balancer target health
- Spring Boot application logs
- RDS database connectivity
- Database persistence verification
- Docker container status
- Prometheus metrics
- VPC Resource Map
- Security Group configuration
- Multi-AZ infrastructure configuration

---

 ## 🧪 Local Development

 ### Prerequisites

 Make sure the following tools are installed:

 - Java 17
- Maven
- Docker
- PostgreSQL (local development, if required)
- Git

 ### Clone the Repository

```
git clone <repository-url>
cd spring-aws-blueprint
```

 ### Build the Application

```
mvn clean package
```

 ### Run with Spring Boot

```
mvn spring-boot:run
```

 ### Run with Docker

```
docker build -t spring-aws-blueprint .
docker run -d -p 8080:8080 spring-aws-blueprint
```

---

 ## 📁 Project Structure

```
spring-aws-blueprint/
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   │
│   └── test/
│
├── Dockerfile
├── pom.xml
└── README.md
```

---

 ## 🎯 Key Engineering Concepts Demonstrated

 This project demonstrates practical implementation of:

 - 3-tier cloud architecture
- AWS VPC design
- Public/private subnet segmentation
- Multi-AZ deployment
- Network security using Security Groups
- Application Load Balancing
- Containerized Spring Boot deployment
- Docker image management
- Amazon RDS integration
- CI/CD with GitHub Actions
- Infrastructure and application separation
- Health checks and application observability
- Prometheus-compatible metrics
- Bastion-based administrative access
- Secure database connectivity
- Cloud-native deployment practices

---

 ## 📈 Deployment Validation

 The deployment has been validated through:

 - REST API testing using Postman
- Spring Boot health endpoint verification
- RDS persistence checks
- EC2 and Docker container verification
- Application Load Balancer health checks
- VPC Resource Map inspection
- AWS networking and Security Group verification
- Prometheus metrics endpoint verification

 A screen recording demonstrating the deployment, API testing, database persistence, and AWS infrastructure is also included as part of the project documentation.

---

 ## 🔮 Future Enhancements

 Potential improvements for extending this architecture include:

 - Infrastructure as Code using Terraform or AWS CloudFormation
- Amazon ECS/EKS for container orchestration
- AWS Secrets Manager for credential management
- Amazon ElastiCache for managed Redis
- Amazon MSK for managed Kafka
- CloudWatch centralized logging and monitoring
- HTTPS using AWS Certificate Manager
- WAF integration with the Application Load Balancer
- Blue/Green or Canary deployments
- Automated deployment from GitHub Actions to AWS
- Auto Scaling Groups for application instances
- Distributed tracing using OpenTelemetry

---

 ## 👨‍💻 Project Objective

 The primary objective of **Spring AWS Blueprint** is to demonstrate how a Spring Boot backend can be designed, containerized, secured, and deployed using AWS infrastructure and modern DevOps practices.

 It serves as a practical reference for understanding the integration between:

```
Spring Boot
     +
Docker
     +
AWS Networking
     +
EC2
     +
Application Load Balancer
     +
RDS PostgreSQL
     +
GitHub Actions
     +
Application Observability
```

---

 ## 📄 License

 This project is intended for educational, demonstration, and reference purposes.

```

### A couple of important fixes I made

- Your badges said **Java 21**, while the deployment/CI section said **JDK 17**. I standardized the README to **Java 17**, since that is what your Maven/CI workflow currently describes.
- I changed the architecture language to make the **public → private app → isolated DB** flow clearer.
- I added a proper **security architecture**, **local development**, **project structure**, **engineering concepts**, and **future enhancements** sections.
- I avoided calling it absolutely “production-ready,” since some components in the described setup—such as credentials/secrets management, HTTPS, WAF, automated infrastructure provisioning, and deployment automation—would normally need additional hardening before a real production deployment.
- I also made the README more suitable for a **GitHub portfolio/resume project**, rather than making it read like internal AWS documentation.
```
