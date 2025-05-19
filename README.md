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


## 🚀 Prerequisites

- **Docker** (v20.10+)  
- **Docker Compose** (v1.29+)

Verify your installation:

```bash
docker --version
docker-compose --version
```

## ⚙️ Quick Start

### 1.Clone the repo

```bash
Copy
Edit
git clone https://github.com/VaibhavGit10/-Vaibhav_Microservices-Task.git
cd -Vaibhav_Microservices-Task
```

### 2.Build & start services

```bash
Copy
Edit
docker-compose up --build
```

### 3.Verify containers are running

```bash
Copy
Edit
docker-compose ps
```

## Each service should now be accessible on localhost under its designated port.

## 🧩 Service Endpoints

| **Service**       | **Port** | **Base URL**                  | **Sample Endpoint**                      |
|-------------------|----------|-------------------------------|------------------------------------------|
| User Service      | 3000     | `http://localhost:3000`       | `/users`                                 |
| Product Service   | 3001     | `http://localhost:3001`       | `/products`                              |
| Order Service     | 3002     | `http://localhost:3002`       | `/orders`                                |
| Gateway Service   | 3003     | `http://localhost:3003/api`   | `/users`, `/products`, `/orders`         |


## 🧪 Testing

You can use `curl`, **Postman**, or your browser to hit these endpoints:

```bash
# 🧍 Users
curl http://localhost:3000/users

# 📦 Products
curl http://localhost:3001/products

# 📑 Orders
curl http://localhost:3002/orders

# 🌐 Gateway
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
```

## A successful response will return a JSON array (or placeholder message) from each service.

## 🛠️ Troubleshooting

| **Issue**                                | **Solution**                                                                 |
|------------------------------------------|------------------------------------------------------------------------------|
| Ports 3000–3003 already in use           | Free up or change the port mappings in `docker-compose.yml`.               |
| Containers crash on start                | Inspect logs: `docker-compose logs <service-name>`                         |
| Code changes not reflected               | Rebuild images: `docker-compose up --


## 🤝 Contributing

1. Fork this repository  
2. Create a feature branch (`git checkout -b feature/YourFeature`)  
3. Commit your changes (`git commit -m "Add awesome feature"`)  
4. Push to the branch (`git push origin feature/YourFeature`)  
5. Open a Pull Request  

---

## 📄 License

Distributed under the **Apache-2.0** License. See [`LICENSE`](./LICENSE) for details.


