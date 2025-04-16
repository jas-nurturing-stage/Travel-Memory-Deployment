# ✈️ TravelMemory – Fullstack Deployment on AWS

This project deploys a MERN stack travel memory application using AWS services like EC2, ALB, ASG, Target Groups, and Cloudflare for security and DNS.

---

## 📁 Folder Structure

```
TravelMemory/
│
├── backend/              # Node.js Express backend
├── frontend/             # Vite + React frontend
├── setup-backend.sh      # Backend provisioning script (Auto Scaling)
└── setup-frontend.sh     # Frontend EC2 static deployment script
```

---

## ⚙️ Prerequisites

Before deploying this application, ensure the following:

- ✅ A valid AWS account with permissions to create EC2, ALB, ASG, and Target Groups
- ✅ Domain access via Cloudflare (e.g., `example.com` and subdomain `api.example.com`)
- ✅ MongoDB Atlas cluster (URI needed in backend `.env`)
- ✅ Security group rules allowing HTTP (80), HTTPS (443), and backend port (3000)
- ✅ Ubuntu-based EC2 AMI with internet access for installing dependencies

---

## 🛠 Installation Instructions

### 📦 Required Services

Ensure the following packages are installed on your EC2 instances:

- **Nginx** – Reverse proxy for both backend and frontend
- **Node.js v22** – JavaScript runtime environment for backend and Vite frontend build
- **PM2** – Process manager for running the backend as a service

### 🧪 PM2 Common Commands

```bash
# Start your backend application using PM2
pm2 start index.js --name "travel-memory-backend"

# View all running PM2 processes
pm2 list

# View logs for a specific process
pm2 logs travel-memory-backend

# Restart a process by name
pm2 restart travel-memory-backend

# Stop a process
pm2 stop travel-memory-backend

# Delete a process from PM2
pm2 delete travel-memory-backend

# Save process list to start on reboot
pm2 save

# Re-generate startup script after reboot
pm2 startup
```

### 🔧 Backend Setup Script

```bash
#!/bin/bash

# === System Update ===
apt-get update -y

# === Install Node.js v22 ===
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
apt-get install -y nodejs

# === Install Nginx ===
apt-get install -y nginx git

# === Clone Backend Repo to /home/ubuntu ===
cd /home/ubuntu
chmod -R 775 /home/ubuntu/
git clone -b backend https://github.com/your-repo/Travel-Memory.git
cd Travel-Memory

# === Create .env File ===
echo "PORT=3000" > .env
echo "MONGO_URI='your mongo url'" >> .env

# === Install App Dependencies & PM2 ===
npm install -y
npm install -g pm2 -y
pm2 start index.js --name "travel-memory-backend"
pm2 startup
pm2 save

# === Configure Nginx ===
rm -f /etc/nginx/sites-enabled/default

bash -c 'cat > /etc/nginx/sites-available/travel-memory << EOF
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF'

ln -s /etc/nginx/sites-available/travel-memory /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

echo "✅ TravelMemory backend is live at api.example.com"
```

### 🧩 Frontend Setup Script

```bash
#!/bin/bash

# === System Update ===
apt-get update -y

# === Install Node.js v22 ===
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
apt-get install -y nodejs

# === Install Nginx & Git ===
apt-get install -y nginx git

# === Clone Frontend Repo to /home/ubuntu ===
cd /home/ubuntu
chmod -R 775 /home/ubuntu/
git clone -b frontend https://github.com/your-repo/Travel-Memory.git
cd Travel-Memory

# === Create .env for Vite ===
echo "VITE_API_BASE_URL=https://api.example.com" > .env

# === Install Dependencies & Build ===
npm install -y
npm run build

# === Copy Build Output to /var/www/html ===
rm -rf /var/www/html/*
cp -r dist/* /var/www/html/

# === Configure Nginx for Static Site ===
rm -f /etc/nginx/sites-enabled/default

bash -c 'cat > /etc/nginx/sites-available/travel-memory-ui << EOF
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files \$uri \$uri/ /index.html;
    }
}
EOF'

ln -s /etc/nginx/sites-available/travel-memory-ui /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

echo "✅ TravelMemory frontend is deployed at: http://example.com"
```

---

## 🧱 Deployment Architecture Diagram

<p align="center">
  <img src="diagram_travelmemory_resized.png" alt="TravelMemory AWS Architecture" width="800">
</p>

---

---

## ⚙️ Backend Deployment Flow (AWS)

### 1. Launch Templates

![travel-memory-nginx](https://github.com/user-attachments/assets/1c524725-f34b-471e-9ec8-e5bae74ebf32)
![Travel-memory ubuntu](https://github.com/user-attachments/assets/26e41e69-c930-4504-8bc3-5df2f3107e58)
![Travel-memory](https://github.com/user-attachments/assets/ad15ac35-81c7-4ab3-adf3-8a02a3ad3b41)


### 2. Target Groups

![TG1](https://github.com/user-attachments/assets/5e7b2c59-81b4-41a6-9a64-bd795a8af1f4)
![TG2](https://github.com/user-attachments/assets/c02e6b6e-463a-4e7b-b7ec-db417ce139aa)


### 3. Application Load Balancer (ALB)

![LB-working](https://github.com/user-attachments/assets/d8fd369c-281c-4d5d-8fc2-866f1d31877f)
![ALB4](https://github.com/user-attachments/assets/dde7e960-f93a-4cc0-925e-d6a0f0fdb2e4)
![ALB1](https://github.com/user-attachments/assets/af4a378c-3c09-4660-9b6e-7e629afce92f)
![ALB 3](https://github.com/user-attachments/assets/fbb2a30e-4e93-432c-bd4a-cfafb476f800)
![ALB2](https://github.com/user-attachments/assets/afea4fec-a55a-4732-bc77-4f5a2b592913)


### 4. Auto Scaling Groups (ASG)

![ASG 1](https://github.com/user-attachments/assets/7df6da13-f8ec-483c-b27f-ecfeb41cff38)
![ASG 2](https://github.com/user-attachments/assets/c64c7c0e-3f7e-41f5-97f2-128f9fd52e45)
![ASG 3](https://github.com/user-attachments/assets/6705cefd-191e-4d1e-b806-db44b73e5341)
![ASG 4](https://github.com/user-attachments/assets/f4236cba-9e6a-4d3b-be9a-955e8a71f5a6)
![ASG 5](https://github.com/user-attachments/assets/0e594521-da53-4196-bf32-b2ca459e7138)


### 5. Cloudflare – Add CNAME for api.example.com

![CNAME record](https://github.com/user-attachments/assets/74300353-d8f8-4a12-a1a8-f5f7ee7826b9)


---

## 🌐 Frontend Deployment Flow

### 1. EC2 Instance
![Frontend-launch](https://github.com/user-attachments/assets/2d5d7969-4469-45ad-bc0c-163d35780c12)

![Frontend-script](https://github.com/user-attachments/assets/85c16e06-29f9-40fb-bd45-bdfe8d9714e1)


### 2. Cloudflare – Add A Record

![A record](https://github.com/user-attachments/assets/b218fe41-0294-43b1-ad9d-111bbc588b65)


### 3. Application Running → example.com

![APP running](https://github.com/user-attachments/assets/5a0d1795-b491-4dc1-8925-26a1a9475678)


---

## 📜 License

This project is open-source and available for **educational, practice, and real-world development guidance**. You are free to use, modify, and distribute this project for non-commercial and commercial purposes.

---

## 🙏 Thank You

We appreciate your interest in the TravelMemory project.

---
