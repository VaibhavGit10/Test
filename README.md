# 🌐 B10_Vaibhav_Pawar_Assignment_04
*✈️ Travel Memory — End-to-End MERN Stack Deployment on AWS*

---
## 🧭 Overview ##
Welcome to the deployment guide for Travel Memory, a full-stack MERN application designed to capture and share travel experiences. This repository is a forked version of the original TravelMemory project, enhanced with structured DevOps practices and cloud deployment workflows.
---

## 📂 Repository Structure

This repository is already **forked** from the original source (https://github.com/UnpredictablePrashant/TravelMemory) into the main branch and contains the following additional branches:

- `Frontend` — React.js code for a responsive and interactive user interface
- `Backend` — Node.js + Express server logic and MongoDB connectivity

### 🚀 Project Objective
The primary aim of this assignment is to deliver a hands-on, end-to-end DevOps deployment of a modern web application using real-world tools and cloud infrastructure. You'll gain practical experience in deploying a scalable and production-ready MERN stack application on AWS, while mastering essential DevOps concepts.

By following this guide, you will:
- 🔁 Fork and clone the Travel Memory repository for personalized deployment.
- 🖥️ Set up AWS EC2 instances to host both frontend and backend services.
- ⚖️ Configure an Application Load Balancer (ALB) for intelligent traffic routing and high availability.
- 📈 Enable Auto Scaling Groups (ASG) to ensure fault tolerance and seamless horizontal scaling.
- 🧩 Deploy Nginx as a reverse proxy to efficiently route requests to your React frontend and Node.js backend.
- 🌐 Integrate Cloudflare DNS for enhanced domain management, performance, and security.
- 📋 Document the entire deployment pipeline, from provisioning to scaling, including useful tips and troubleshooting steps.

---

## 🏗️ Architecture Diagram

<p align="center">
  <img src="https://github.com/user-attachments/assets/9bf15ee4-2393-45ed-bb90-bda86da28f6f" width="70%" />
</p>

---

## 🧩 Common Configuration

### ✅ Step 1: Create a Security Group (For both Frontend & Backend)

1. Go to AWS Console > EC2 > Security Groups > Create Security Group.
2. Name it: `travel-memory-sg` (or as per your choice).
3. Add **Inbound Rules**:
   - **Type**: SSH | **Protocol**: TCP | **Port**: 22 | **Source**: My IP
   - **Type**: HTTP | **Protocol**: TCP | **Port**: 80 | **Source**: 0.0.0.0/0, ::/0
   - **Type**: HTTPS | **Protocol**: TCP | **Port**: 443 | **Source**: 0.0.0.0/0, ::/0
   - **Type**: Custom TCP | **Port**: 3000 | **Source**: 0.0.0.0/0, ::/0
   - **Type**: Custom TCP | **Port**: 3001 | **Source**: 0.0.0.0/0, ::/0

#### 📸 Output Screenshots

<p align="center">
  <img src="https://github.com/user-attachments/assets/df2efa5b-5375-41a3-aac7-26f1b03fcd4c" width="45%" />
  <img src="https://github.com/user-attachments/assets/5db3bd74-56d1-4307-9949-c0de7a6ba851" width="45%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/b926593d-c503-4cfc-80e1-5941e3fc2790" width="45%" />
</p>

---

## 🛠️ Backend Configuration

### ✅ Step 1: Create Launch Template

1. Go to AWS Console > EC2 > Launch Templates > Create Launch Template.
2. **Name**: `backend-launch-template` (or as per your choice).
3. Select **Ubuntu 22.04** as the AMI.
4. Choose **Instance Type**: `t2.micro` (or as per your choice).
5. Select your **Key Pair**.
6. Select the **Security Group** created earlier.

7. **Edit `MERN-BACKEND.sh` script:**
   - Replace:
     ```bash
     echo "MONGO_URI=mongodb+srv://<USERNAME>:<PASSWORD>@<CLUSTER_NAME>/<COLLECTION_NAME>" >> .env
     ```
     👉 with your MongoDB Atlas connection string.

   - Replace:
     ```bash
     server_name backendsubdomainname.yourdomainname.com;
     ```
     👉 with your own backend subdomain:
     ```bash
     server_name yourownbackendsubdomainname.yourdomainname.com;
     ```

   - *(Optional)* Update success message:
     ```bash
     echo "✅ TravelMemory backend is live at yourownbackendsubdomainname.yourdomainname.com"
     ```

8. Paste the updated `MERN-BACKEND.sh` script into the **Advanced > User Data** section.
9. Click **Create Template**.

#### 📸 Output Screenshots

<p align="center">
  <img src="https://github.com/user-attachments/assets/cac53227-7f01-4a6e-96d4-ba3064ab307f" width="45%" />
  <img src="https://github.com/user-attachments/assets/c513a8d5-678e-4baa-826a-22d539f47d68" width="45%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/057fc65c-3aab-4794-b89a-36c99503b015" width="45%" />
  <img src="https://github.com/user-attachments/assets/c323a506-fb16-4979-a572-489e74b4bd11" width="45%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/231864fd-2935-4ef7-9254-2a66dc8c29ba" width="45%" />
</p>


---

### ✅ Step 2: Create Target Group

1. Go to EC2 > Target Groups > Create Target Group.
2. **Target Type**: Instances
3. **Name**: `backend-target-group` (or as per your choice).
4. **Protocol**: HTTP | **Port**: 80
5. **VPC**: Select your VPC
6. **Protocol Version**: HTTP1

#### 📸 Output Screenshots

<p align="center">
  <img src="https://github.com/user-attachments/assets/a471ef9a-12ac-495b-b130-6cc5da16f6da" width="45%" />
  <img src="https://github.com/user-attachments/assets/b3c8d185-2fee-408c-88a5-552d7c1e952d" width="45%" />
</p>

---

### ✅ Step 3: Create Application Load Balancer

1. Go to EC2 > Load Balancers > Create Load Balancer.
2. **Name**: `backend-load-balancer` (or as per your choice).
3. **Scheme**: Internet-facing
4. **IP address type**: IPv4
5. **Listeners**: HTTP port 80
6. **Availability Zones**: Select All (or as per your choice).
7. **Security Group**: Select the Security Group created earlier.
8. **Routing**: Select the Target Group created earlier.

#### 📸 Output Screenshots

<p align="center">
  <img src="https://github.com/user-attachments/assets/5a3e0d3c-3184-49ea-acfb-3d1c54f8b33c" width="48%" />
  <img src="https://github.com/user-attachments/assets/c90a106d-a21d-4ba7-93ec-a8a0d4710d9f" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/2bff2141-2571-4186-b7f1-4dd0384798a2" width="48%" />
</p>



---

### ✅ Step 4: Create Auto Scaling Group

1. Go to EC2 > Auto Scaling Groups > Create Auto Scaling Group.
2. **Name**: `backend-asg` (or as per your choice).
3. Use the **Launch Template** created earlier.
4. In the **VPC and Availability Zones** section, Select All (or as per your choice).
5. **Instance Distribution**: Best balance effort.
6. Attach the existing **Load Balancer** and select the earlier created **Target Group**.
7. **Tick the Option**: Elastic Load Balancing health checks.
8. **Health Check Grace Period**: 300 seconds.
9. **Group Size**:
   - Desired: 2
   - Min: 1
   - Max: 3
10. **Scaling policies**: No scaling policies.
11. Review and Click **Create Auto Scaling Group**.
12. Wait for instances to become Healthy.

#### 📸 Output Screenshots 

<p align="center">
  <img src="https://github.com/user-attachments/assets/001d2d67-e300-49ab-8174-d600617eb068" width="48%" />
  <img src="https://github.com/user-attachments/assets/ea0f8c06-90a4-43e0-a2fc-f637924d3f7b" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/f2d2e057-cc00-4ab9-b5bb-c05127209026" width="48%" />
  <img src="https://github.com/user-attachments/assets/3c863023-ae08-4367-b32e-ff67aadfd8d2" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/8f61dbda-43e9-4809-b956-2465a5b46677" width="48%" />
  <img src="https://github.com/user-attachments/assets/8b63a0c2-29f2-402c-9731-f2df070e0bd1" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/fed7885b-fbf9-4f52-966f-1ab5d986663e" width="48%" />
  <img src="https://github.com/user-attachments/assets/568131e8-5f14-494e-9563-3485c46561bd" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/ba6add95-d32d-4aa7-b00f-9fd437dfeca1" width="48%" />
  <img src="https://github.com/user-attachments/assets/3d401a05-1f53-430e-a930-81ae4ce16ffa" width="48%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/f9c36bc2-0abd-4891-b65a-202d5d327857" width="48%" />
  <img src="https://github.com/user


---

### ✅ Backend Cloudflare Configuration

1. Get the **DNS name** of your Load Balancer.
2. Go to **Cloudflare** > DNS Settings.
3. Add a new record:
   - **Type**: CNAME
   - **Name**: `backendsubdomainname` (choose your own, used in scripts)
   - **Target**: DNS name of your Load Balancer
   - **Proxy Status**: DNS only

4. Test:
   - Visit: `http://backendsubdomainname.yourdomainname.com` or `http://backendsubdomainname.yourdomainname.com/trip`
   - ✅ If you get a response, backend is successfully configured!
  
#### 📸 Output Screenshots 


<p align="center">
  <img src="https://github.com/user-attachments/assets/9aac9230-a23d-46ff-8fbf-721764d618ff" width="48%" />
  <img src="https://github.com/user-attachments/assets/5f71b1b2-55e8-43ea-9c86-349e9a1c0d4c" width="48%" />
</p>



---

## 🌐 Frontend Configuration

### ✅ Step 1: Create EC2 Instance

1. Go to EC2 > Instances > Launch Instance.
2. **Name**: `frontend-instance` (or as per your choice).
3. **AMI**: Ubuntu 22.04
4. **Instance Type**: t2.micro (or as per your choice).
5. **Key Pair**: Select your existing key pair.
6. **Security Group**: Select the Security Group created earlier.
7. **Edit `MERN-FRONTEND.sh` script:**
   - Replace:
     ```bash
     VITE_API_BASE_URL=http://backendsubdomainname.yourdomainname.com
     ```
     👉 with:
     ```bash
     VITE_API_BASE_URL=http://yourownbackendsubdomainname.yourdomainname.com
     ```

   - Replace:
     ```bash
     server_name frontendsubdomainname.yourdomainname.com;
     ```
     👉 with:
     ```bash
     server_name yourownfrontendsubdomainname.yourdomainname.com;
     ```

   - *(Optional)* Update success message:
     ```bash
     echo "✅ TravelMemory frontend is deployed at: yourownfrontendsubdomainname.yourdomainname.com"
     ```

8. Paste the updated `MERN-FRONTEND.sh` script into **Advanced > User Data**.
9. Click **Launch Instance**.

#### 📸 Output Screenshots 

<p align="center">
  <img src="https://github.com/user-attachments/assets/f0006406-a585-4665-bf13-8e3ebc59c659" width="45%" />
  <img src="https://github.com/user-attachments/assets/c5906a50-45a5-4083-9129-3927639e8520" width="45%" />
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/468c0c35-6c75-4a73-9dae-6f7dc77381a5" width="45%" />
  <img src="https://github.com/user-attachments/assets/a0d678cc-9252-473e-bcf4-f14cf135d27f" width="45%" />
</p>


---

### ✅ Frontend Cloudflare Configuration

1. Get the **Public IPv4 address** of your EC2 frontend instance.
2. Go to **Cloudflare** > DNS Settings.
3. Add a new record:
   - **Type**: A
   - **Name**: `frontendsubdomainname` (choose your own, used in scripts)
   - **Target**: Public IPv4 of the EC2 instance
   - **Proxy Status**: DNS only

4. Test:
   - Visit: `http://frontendsubdomainname.yourdomainname.com`
   - ✅ If frontend loads and you are able to create a trip record, your setup is successful!

#### 📸 Output Screenshots 

<p align="center">
  <img src="https://github.com/user-attachments/assets/d76673eb-d6ec-4c67-9252-f47fe24e33d5" width="45%" />
  <img src="https://github.com/user-attachments/assets/faadff31-5838-421c-83ae-d63957bab95a" width="45%" />
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/f873107a-4200-40bc-a8b8-9fef14339424" width="45%" />
</p>


---

## 🚀 Enhancement Points for this Version 1 Release

1. Deploy this project for **HTTPS** (SSL/TLS) for better Security.
2. Use **Elastic IP** for the Frontend Instance to ensure a Static IP.

---

## 🎉 Congratulations!

You have successfully deployed the **Travel Memory MERN Stack Application** with:
- Auto Scaling Backend
- Load Balancer for Backend EC2 Instances
- Standalone Frontend EC2 Instance
- Cloudflare-managed DNS

## Additional Information
POST Request URL: http://your-server-url/api/trip/

Request Body: 
```json
{
    "tripName": "Incredible India",
    "startDateOfJourney": "19-03-2022",
    "endDateOfJourney": "27-03-2022",
    "nameOfHotels":"Hotel Namaste, Backpackers Club",
    "placesVisited":"Delhi, Kolkata, Chennai, Mumbai",
    "totalCost": 800000,
    "tripType": "leisure",
    "experience": "Lorem Ipsum, Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum, ",
    "image": "https://t3.ftcdn.net/jpg/03/04/85/26/360_F_304852693_nSOn9KvUgafgvZ6wM0CNaULYUa7xXBkA.jpg",
    "shortDescription":"India is a wonderful country with rich culture and good people.",
    "featured": true
}
```

## 📜 License
This project is licensed under the MIT License. Feel free to use, modify, and distribute it as per the terms of the license.

## 🤝 Contributions Welcome
Got ideas to improve this project? Found a bug? Feel free to fork the repository, make your changes, and open a pull request. Contributions of all kinds are welcome!
If this project helped you, consider giving it a ⭐ — it really motivates and supports continued development. 🙌

## 📬 Contact & Support
For issues, feature requests, or general questions, please open an issue on GitHub.
Let’s build better software together!

## 🚀 Happy Deploying!
Wishing you smooth deployments and zero downtime. Keep building amazing things! 💻✨

---
