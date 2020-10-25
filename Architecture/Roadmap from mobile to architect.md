# Roadmap from Mobile Development to Solution Architect


## 1. Backend Developer

**Spring Framework:**
- Beans
- Spring Boot vs Spring MVC
- Application Architectures (layered)

**REST:**
- HTTP (Status codes, methods, standard headers)
- 5 levels of rest
- Authentication (Basic, JWT, OAuth 2.0, RBAC, Scopes)
- [I18n + Localization](http://apiux.com/2013/03/20/5-laws-api-dates-and-times/)

**Databases:**
- Types (SQL, Document Store, Key-Value store)
- Spring JPA
- How to select a database (CAP theorem, PACELC theorem)

**Docker:**
- Difference between physical servers, VM, Container
- Containerise a web application
- Deploy containers to EC2
- Docker-compose

**Message Brokers:**
- Why are message brokers needed
- try out a message broker: RabbitMQ/Kafka/ActiveMQ
- Message Patterns (covered in Microservice Patterns book)
- how to select a message broker (covered in Microservice Patterns book)

## 2. Architecture

**Microservices:** _(all of this is in Microservice patterns)_
- How to split services (bounded-context pattern)
- Domain Driven Development
- Sagas + Transaction logging (important)
- Event Sourcing (important)
- CQRS (important)
- API Gateways
- Testing services

**Cloud:**
- Pick a cloud platform
- Understand how to build a highly available, scalable, redundant architecture _(AWS Solution Architecture does a good job with this)_
- Kubernetes
- Infrastructure as Code: One of Cloudformation/Terraform
- Monitoring: Prometheus, Grafana, AlertManager