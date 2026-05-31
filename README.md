# 🖊️ Collaborative Code Editor

> Real time collaborative code editing powered by **Yjs CRDTs**, **Monaco Editor**, **Socket.IO**, and deployed on **AWS ECS** via Docker Image.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-18+-blue.svg)](https://reactjs.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
[![AWS ECS](https://img.shields.io/badge/AWS-ECS%20Deployed-orange.svg)](https://aws.amazon.com/ecs/)

---

## 📖 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Project Workflow](#project-workflow)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Docker Setup](#docker-setup)
- [AWS Deployment](#aws-deployment)
- [How Collaboration Works](#how-collaboration-works)
- [Screenshots](#screenshots)
- [Live Demo](#live-demo)
- [Future Improvements](#future-improvements)
- [Learning Outcomes](#learning-outcomes)
- [Author](#author)
- [License](#license)

---

## 🌟 Overview

The **Collaborative Code Editor** is a full-stack web application that allows multiple users to write and edit code simultaneously in real time, similar to Google Docs but for code. It uses **Conflict-free Replicated Data Types (CRDTs)** via Yjs to ensure all changes are merged automatically without conflicts, regardless of how many users are editing at once.

The application is containerized with **Docker** and deployed on **AWS ECS** behind an **Application Load Balancer**, making it scalable and production-ready.

---

## ✨ Features

- **Real-time collaborative editing** : multiple users edit simultaneously
- **CRDT-based synchronization** : conflict free merging using Yjs
- **Monaco Editor** : the same powerful editor that powers VS Code
- **Live active users panel** : see who's editing in real time
- **Username-based session joining** : simple URL based user identity
- ⚡ **WebSocket communication** : low latency updates via Socket.IO
- **Automatic reconnection handling** : resilient to network interruptions
- 🐳 **Dockerized application** : consistent environments across all platforms
- ☁️ **AWS cloud-native deployment** : ECS + ECR + ALB infrastructure

---

## 🛠️ Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| React | UI framework |
| Vite | Build tool & dev server |
| Monaco Editor | Code editor component |
| Tailwind CSS | Styling |
| Yjs | CRDT document model |
| y-monaco | Yjs binding for Monaco |

### Backend
| Technology | Purpose |
|---|---|
| Node.js | Runtime environment |
| Express.js | HTTP server |
| Socket.IO | WebSocket communication |
| y-socket.io | Yjs provider over Socket.IO |

### DevOps & Cloud
| Technology | Purpose |
|---|---|
| Docker | Containerization |
| AWS ECR | Container image registry |
| AWS ECS | Container orchestration |
| Application Load Balancer | Traffic routing & public access |
| AWS IAM | Access control & permissions |
| VPC & Security Groups | Network isolation & security |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENT BROWSER                      │
│                                                         │
│   ┌─────────────────┐      ┌──────────────────────┐    │
│   │  Monaco Editor  │◄────►│   Yjs CRDT Document  │    │
│   │  (Code Input)   │      │  (Local State Model) │    │
│   └────────┬────────┘      └──────────┬───────────┘    │
│            │                          │                 │
│            └──────────┬───────────────┘                 │
│                       │                                 │
│              ┌────────▼────────┐                        │
│              │  y-socket.io    │                        │
│              │  Provider       │                        │
│              └────────┬────────┘                        │
└───────────────────────┼─────────────────────────────────┘
                        │ WebSocket (Socket.IO)
┌───────────────────────▼─────────────────────────────────┐
│              APPLICATION LOAD BALANCER (AWS ALB)        │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                  AWS ECS CLUSTER                        │
│                                                         │
│   ┌─────────────────────────────────────────────────┐  │
│   │              ECS Task (Docker Container)         │  │
│   │                                                  │  │
│   │   ┌──────────────┐    ┌─────────────────────┐   │  │
│   │   │  Express.js  │    │   Socket.IO Server  │   │  │
│   │   │  HTTP Server │    │  + y-socket.io Hub  │   │  │
│   │   └──────────────┘    └─────────────────────┘   │  │
│   └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔄 Project Workflow

### End-to-End Flow — From Code to Cloud

```
Developer Machine
      │
      ▼
┌─────────────────────┐
│  Write Code Locally │
│  Frontend + Backend │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  docker build       │  ← Dockerfile bundles frontend
│  (Build Image)      │    build + Node.js backend
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  aws ecr            │  ← Tag & push image to
│  (Push to ECR)      │    Amazon Elastic Container Registry
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  AWS IAM            │  ← IAM Roles grant ECS permission
│  (Permissions)      │    to pull from ECR securely
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  AWS ECS Service    │  ← ECS pulls image, runs Task
│  (Run Container)    │    inside a Fargate/EC2 cluster
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  VPC + Security     │  ← Network isolation, inbound
│  Groups             │    rules allow port 3000/80
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Application Load   │  ← Routes public HTTP/HTTPS
│  Balancer (ALB)     │    traffic to ECS tasks
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Public Users       │  ← Access the app via
│  (Browser)          │    ALB DNS URL
└─────────────────────┘
```

### Real-Time Collaboration Flow

```
User A types code
      │
      ▼
Monaco Editor captures change
      │
      ▼
Yjs CRDT generates an "operation"
(not a full document, just the delta)
      │
      ▼
ysocket.io sends the operation
over WebSocket to the server
      │
      ▼
Socket.IO server broadcasts
the operation to all other clients
      │
      ▼
User B's Yjs instance receives
and merges the operation
      │
      ▼
y monaco applies the change
to User B's Monaco Editor
      │
      ▼
Both editors are now in sync 
(no conflicts, no overwrites)
```

---

## 📁 Project Structure

```
collaborative-code-editor/
│
├── Frontend/                    # React frontend application
│   ├── src/
│   │   ├── components/          # React components (Editor, UserPanel, etc.)
│   │   ├── App.jsx              # Root component
│   │   └── main.jsx             # Vite entry point
│   ├── public/
│   ├── index.html
│   ├── tailwind.config.js
│   ├── vite.config.js
│   └── package.json
│
├── Backend/                     # Node.js backend server
│   ├── server.js                # Express + Socket.IO + y-socket.io
│   └── package.json
│
├── Dockerfile                   # Multi-stage build (frontend + backend)
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Node.js v18+
- npm or yarn
- Docker (for containerized setup)
- AWS CLI (for cloud deployment)

### Clone the Repository

```bash
git clone https://github.com/aman12-10-12/collaborative-code-editor.git
cd collaborative-code-editor
```

### Frontend Setup

```bash
cd Frontend
npm install
npm run dev
```

Frontend runs at: `http://localhost:5173`

### Backend Setup

```bash
cd Backend
npm install
node server.js
```

Backend runs at: `http://localhost:3000`

> **Note:** Open the app in two different browser tabs/windows and enter different usernames to test real-time collaboration locally.

---

## 🐳 Docker Setup

### Build the Docker Image

```bash
docker build -t collaborative-editor .
```

### Run the Container

```bash
docker run -p 3000:3000 collaborative-editor
```

App available at: `http://localhost:3000`

---

## ☁️ AWS Deployment

This project is deployed using a fully cloud-native pipeline on AWS.

### Deployment Architecture

```
Local Docker Image
       │
       ▼
Amazon ECR (Container Registry)
       │  ← Image stored & versioned
       ▼
AWS IAM (Role-based Access)
       │  ← ECS Task Role pulls from ECR
       ▼
Amazon ECS (Fargate / EC2)
       │  ← Runs the Docker container
       ▼
VPC + Security Groups
       │  ← Controls inbound/outbound traffic
       ▼
Application Load Balancer (ALB)
       │  ← Routes public traffic to ECS tasks
       ▼
Public Internet Access
```

### Step-by-Step Deployment

**1. Authenticate Docker with ECR**
```bash
aws ecr get-login-password --region <region> | \
  docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

**2. Tag and Push Image to ECR**
```bash
docker tag collaborative-editor:latest \
  <account-id>.dkr.ecr.<region>.amazonaws.com/collaborative-editor:latest

docker push \
  <account-id>.dkr.ecr.<region>.amazonaws.com/collaborative-editor:latest
```

**3. Create ECS Cluster & Service**
- Create an ECS cluster (Fargate or EC2 launch type)
- Define a Task Definition referencing your ECR image
- Attach an IAM Task Execution Role with `AmazonECSTaskExecutionRolePolicy`
- Create an ECS Service linked to a Target Group

**4. Configure Load Balancer**
- Create an Application Load Balancer in your VPC
- Add a listener on port 80 (or 443 for HTTPS)
- Register ECS tasks as targets in the Target Group

**5. Security Groups**
- ALB Security Group: allow inbound `80/443` from `0.0.0.0/0`
- ECS Task Security Group: allow inbound from ALB Security Group only

---

## 🤝 How Collaboration Works

This project uses **Yjs** a battle tested CRDT library for real time synchronization.

### What is a CRDT?

A **Conflict-free Replicated Data Type (CRDT)** is a data structure that can be updated independently by multiple users and then merged automatically — without conflicts. Every operation is designed to be:

- **Commutative** — order doesn't matter
- **Associative** — grouping doesn't matter
- **Idempotent** — applying the same operation twice is safe

### How it works in this project

| Traditional Approach | CRDT Approach |
|---|---|
| Send full document state | Send only the delta (operation) |
| Last write wins | All writes are merged |
| Conflicts require resolution | No conflicts by design |
| Requires locking | Fully decentralized |

This is the same technology used in **Google Docs**, **Notion**, and **Figma**.

---

## 📸 Screenshots

> _Add screenshots of your editor interface here._

```md
![Editor View](./screenshots/editor.png)
![Active Users Panel](./screenshots/users.png)
```

---

## 🌐 Live Demo

```
http://your-load-balancer-url
```

---

## 🔮 Future Improvements

- [ ] Cursor position sharing with user-specific colors
- [ ] Room-based collaboration with shareable links
- [ ] Persistent document storage (PostgreSQL / DynamoDB)
- [ ] Authentication system (JWT / OAuth)
- [ ] Code execution sandbox
- [ ] Syntax support for multiple languages
- [ ] Collaborative inline chat
- [ ] Document version history
- [ ] HTTPS support on ALB with ACM SSL certificate

---

## 🎓 Learning Outcomes

Through building this project, I gained hands-on experience with:

- CRDT-based distributed synchronization using Yjs
- Real-time systems architecture with WebSockets
- Monaco Editor integration and customization
- Docker multi-stage builds for production
- AWS ECS container orchestration with Fargate
- AWS ECR for private container image management
- IAM roles and least-privilege access control
- VPC networking, Security Groups, and ALB configuration
- Production deployment strategies for cloud-native apps

---

## 👤 Author

**Aman Raj**

- GitHub: [@aman12-10-12](https://github.com/aman12-10-12)
- Repository: [collaborative-code-editor](https://github.com/aman12-10-12/collaborative-code-editor)

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details.

---

<div align="center">
  <sub>Built with ❤️ using React, Yjs, Socket.IO, Docker and AWS</sub>
</div>
