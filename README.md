# 🛠️ Node.js Microservices with Docker Compose

This repository contains a set of Node.js microservices—**User Service**, **Product Service**, and **Gateway Service**—containerized using Docker and orchestrated with Docker Compose. Each service runs independently and communicates over a local network.

---

## 📁 Project Structure

.
├── user-service/ # User Service (Port 3000)
│ └── Dockerfile
│
├── product-service/ # Product Service (Port 3001)
│ └── Dockerfile
│
├── gateway-service/ # Gateway Service (Port 3003)
│ └── Dockerfile
│
├── docker-compose.yml # Multi-container orchestration file
└── README.md
