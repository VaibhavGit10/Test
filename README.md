# 🎯 Vaibhav Microservices Task

A Docker-Compose orchestrated set of four Node.js microservices:

- **User Service** (port 3000)  
- **Product Service** (port 3001)  
- **Order Service** (port 3002)  
- **Gateway Service** (port 3003)

---

## 📖 Overview

This project demonstrates how to containerize and orchestrate multiple Node.js microservices using Docker Compose. Each service exposes a simple REST endpoint and communicates through the Gateway.

---

## 🗂️ Repository Structure


```bash
.
├── user-service/ # User Service (Node.js + Express)
│ ├── Dockerfile
│ └── index.js
│
├── product-service/ # Product Service (Node.js + Express)
│ ├── Dockerfile
│ └── index.js
│
├── order-service/ # Order Service (Node.js + Express)
│ ├── Dockerfile
│ └── index.js
│
├── gateway-service/ # API Gateway (Node.js + Express + http-proxy-middleware)
│ ├── Dockerfile
│ └── index.js
│
├── docker-compose.yml # Compose file spinning up all four services
└── README.md 
```
