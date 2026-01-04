# n8n

Below is a clear, step-by-step guide to configure n8n on AWS EC2 using Docker (recommended, stable, production-ready).

I’ll assume:

EC2 OS: Ubuntu 22.04

You want web access to n8n

Later you can add domain + HTTPS

🧠 Architecture (simple)
Internet
   ↓
AWS EC2 (Ubuntu)
   ↓
Docker
   ↓
n8n container

✅ STEP 1: Launch EC2 Instance

Go to AWS EC2 → Launch Instance

Choose:

AMI: Ubuntu Server 22.04

Instance type: t2.micro (ok for testing)

Key pair: create/download .pem

Security Group:

Allow SSH (22) from your IP

Allow HTTP (80) (or custom port 5678)

Launch instance

🔐 STEP 2: Connect to EC2
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP


Update system:

sudo apt update && sudo apt upgrade -y

🐳 STEP 3: Install Docker
sudo apt install docker.io -y


Enable Docker:

sudo systemctl start docker
sudo systemctl enable docker


Allow non-root Docker usage:

sudo usermod -aG docker ubuntu
newgrp docker


Verify:

docker --version

🧩 STEP 4: Install Docker Compose
sudo apt install docker-compose-plugin -y


Verify:

docker compose version

📁 STEP 5: Create n8n Project Directory
mkdir ~/n8n
cd ~/n8n


Create volume folder:

mkdir n8n_data

📄 STEP 6: Create docker-compose.yml
nano docker-compose.yml


Paste this 👇
```
version: "3.8"

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=strongpassword
      - N8N_HOST=EC2_PUBLIC_IP
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
    volumes:
      - ./n8n_data:/home/node/.n8n
    restart: unless-stopped
```

⚠️ Replace:

EC2_PUBLIC_IP

strongpassword

Save:

CTRL + O → Enter → CTRL + X

▶️ STEP 7: Start n8n
docker compose up -d


Check:

docker ps

🌍 STEP 8: Open AWS Security Group Port

In EC2 Security Group, add:

Type	Port
Custom TCP	5678
Source	0.0.0.0/0
🌐 STEP 9: Access n8n

Open browser:

http://EC2_PUBLIC_IP:5678


Login with:

Username: admin

Password: strongpassword

🎉 n8n is running!

🔐 STEP 10 (Recommended): Use Nginx + Domain + HTTPS

Instead of:

:5678


Use:

https://n8n.yourdomain.com


High-level steps:

Point domain → EC2 IP

Install Nginx

Use Certbot for SSL

Reverse proxy to n8n

👉 I can give exact Nginx + SSL config if you want.

🛠 Useful Commands

Stop:

docker compose down


Logs:

docker compose logs -f


Update n8n:

docker compose pull
docker compose up -d

🧠 Best Practices (Important)

✅ Always use Docker
✅ Enable auth (already done)
✅ Use HTTPS in production
✅ Backup n8n_data folder
✅ Use environment variables for secrets

🔚 Summary

Launch EC2

Install Docker + Compose

Create docker-compose file

Open port

Access n8n

If you want next:

🔐 Nginx + SSL

☁️ RDS PostgreSQL with n8n

🧩 Webhook configuration

🔄 Auto-start + backups
