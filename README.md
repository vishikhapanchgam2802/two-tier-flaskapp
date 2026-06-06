# 2-Tier Flask + MySQL App using Docker Networking

A two-tier web application built with **Flask** and **MySQL**, containerized using Docker with a focus on **Docker networking and inter-container communication**. Deployed and tested on an **AWS EC2** instance.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│        Docker Network: twotier          │
│                                         │
│  ┌──────────────┐    ┌───────────────┐  │
│  │  Flask App   │───▶│  MySQL DB     │  │
│  │  (Port 5000) │    │  (Port 3306)  │  │
│  └──────────────┘    └───────────────┘  │
│           │                             │
└───────────┼─────────────────────────────┘
            │
      EC2 Instance (Public IP)
```

- Flask and MySQL run in **separate containers**
- Both connected to a **custom Docker bridge network** (`twotier`)
- Flask connects to MySQL using the **container name as hostname** — no hardcoded IPs
- Data persisted in MySQL and retrieved by Flask across containers
- Application hosted on **AWS EC2** (ap-south-1)

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Flask + Python 3.9 | Web application framework |
| MySQL 5.7 | Relational database |
| Docker | Containerization |
| Docker Compose | Multi-container orchestration |
| AWS EC2 | Cloud deployment |

---

## 📁 Project Structure

```
two-tier-flask-app/
├── app.py                        # Flask app — routes, MySQL connection
├── requirements.txt              # Python dependencies
├── Dockerfile                    # Flask container image
├── Dockerfile-multistage         # Optimized multi-stage build
├── docker-compose.yml            # Multi-container setup
├── message.sql                   # DB init script (messages table)
├── templates/
│   └── index.html                # Frontend UI
├── eks-manifests/                # Kubernetes manifests for EKS deployment
└── k8s/                          # Local Kubernetes configs
```

---

## 🚀 Getting Started

### Prerequisites
- Docker and Docker Compose installed
- AWS EC2 instance (Ubuntu) with ports **5000** and **3306** open in Security Group

### Run with Docker Compose

```bash
# Clone the repository
git clone https://github.com/vishikhapanchgam2802/two-tier-flask-app.git
cd two-tier-flask-app

# Start both containers
docker-compose up -d

# Verify both containers are running
docker ps
```

Visit `http://<EC2-Public-IP>:5000` in your browser.

### Run Manually with Docker Commands

```bash
# Step 1: Create custom Docker network
docker network create twotier

# Step 2: Run MySQL container
docker run -d \
  --name mysql \
  --network twotier \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin \
  -v ./message.sql:/docker-entrypoint-initdb.d/message.sql \
  -p 3306:3306 \
  mysql:5.7

# Step 3: Build Flask image
docker build -t flask-app .

# Step 4: Run Flask container
docker run -d \
  --name flask-app \
  --network twotier \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  -p 5000:5000 \
  flask-app:latest
```

---

## 📸 Project Demo

### 1. Flask App live on EC2 (port 5000)
![Flask App UI](screenshots/flask-app-ui.png)
> Application accessible at `http://3.111.213.202:5000` — served from a Docker container on AWS EC2 (ap-south-1)

### 2. Messages stored and displayed from MySQL
![Messages Stored](screenshots/messages-stored.png)
> Messages submitted via the Flask frontend are persisted in MySQL and displayed on page load — proving successful inter-container communication

### 3. Data verified directly inside MySQL container
![MySQL Verification](screenshots/mysql-verification.png)
> `SELECT * FROM messages` run inside the MySQL container confirms the same data written by the Flask container — end-to-end Docker networking working correctly

---

## 🔑 Key Concepts Demonstrated

- **Docker Bridge Networking**: Custom `twotier` network allows containers to communicate by name
- **Service Discovery**: `MYSQL_HOST=mysql` — Flask resolves the MySQL container by name, not IP
- **Environment Variables**: All credentials passed via env vars, never hardcoded
- **DB Initialization**: `message.sql` auto-runs on MySQL container startup via `docker-entrypoint-initdb.d`
- **Health Checks**: Both services use Docker health checks to ensure proper startup order
- **EC2 Deployment**: Full end-to-end deployment on AWS cloud infrastructure

---

## 🧠 What I Learned

- How Docker bridge networks enable DNS-based service discovery between containers
- Why container startup order matters and how `depends_on` + health checks solve it
- Best practices for passing database credentials using environment variables
- How to initialize a MySQL schema automatically using init scripts
- Deploying and testing multi-container apps on AWS EC2

---

*Built as part of my DevOps learning journey | AWS Certified Cloud Practitioner (CLF-C02)*

<img width="1920" height="1020" alt="5 two-tier flask app" src="https://github.com/user-attachments/assets/b9d78ed2-1d97-49e2-8671-3828f5056b49" />
<img width="1920" height="1020" alt="6" src="https://github.com/user-attachments/assets/970a356a-6e2b-49ee-97ad-f19511248db5" />
<img width="1920" height="1020" alt="7" src="https://github.com/user-attachments/assets/cbee648f-de3a-4db3-b9b4-cdab0edd87dd" />
