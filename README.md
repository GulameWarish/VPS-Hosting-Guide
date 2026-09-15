# 🚀 Deploying a MERN Stack Project on Hostinger VPS

This guide explains **how to deploy a MERN stack application (MongoDB, Express, React, Node.js)** on a **Hostinger VPS** using **Nginx, PM2, and SSL certificates**.

It covers everything from **server preparation to production deployment with HTTPS**.

---

# 🏗️ Architecture Overview

A typical production MERN deployment looks like this:

```
Internet
   │
   ▼
Domain (yourdomain.com)
   │
   ▼
Nginx (Reverse Proxy + SSL)
   │
   ├── React Frontend (Static Build)
   │
   └── Node.js API (Express Server via PM2)
            │
            ▼
        MongoDB Database
```

---

# 1️⃣ Preparing the VPS Environment

Before deploying applications, the server must be prepared with required tools.

---

## 1.1 Connect to Your VPS

Use SSH to log in to your server.

```bash
ssh root@your_vps_ip
```

Example:

```bash
ssh root@187.124.98.219
```

SSH allows you to **remotely control your server from your terminal**.

---

## 1.2 Update and Upgrade System

Always update the system before installing packages.

```bash
sudo apt update
sudo apt upgrade -y
```

Why this is important:

- Installs latest security patches
- Fixes vulnerabilities
- Updates installed packages

---

## 1.3 Install Node.js Using NVM

NVM (Node Version Manager) allows managing multiple Node versions.

Install NVM:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Load NVM:

```bash
\. "$HOME/.nvm/nvm.sh"
```

Install Node.js:

```bash
nvm install 22
```

Verify installation:

```bash
node -v
npm -v
```

---

## 1.4 Install Git

Git is needed to pull your project from GitHub.

```bash
sudo apt install -y git
```

Check version:

```bash
git --version
```

---

# 2️⃣ Setting Up MongoDB Database

Your MERN stack needs MongoDB.

You have two options:

---

## Option 1 — MongoDB Atlas (Recommended)

Advantages:

- No server management
- Automatic backups
- Scalable

Example connection string:

```
mongodb+srv://username:password@cluster.mongodb.net/database
```

---

## Option 2 — Install MongoDB on VPS

Install MongoDB:

```bash
sudo apt install mongodb
```

Start service:

```bash
sudo systemctl start mongodb
```

Enable auto start:

```bash
sudo systemctl enable mongodb
```

Check status:

```bash
sudo systemctl status mongodb
```

---

# 3️⃣ Deploying the Express + Node.js Backend

Now deploy the backend API server.

---

## 3.1 Create Project Directory

```bash
mkdir /var/www
cd /var/www
```

`/var/www` is the standard directory used to host web applications.

---

## 3.2 Clone Your Backend Repository

```bash
git clone https://github.com/yourusername/your-repo.git
```

Navigate to backend folder:

```bash
cd your-repo/backend
```

---

## 3.3 Install Dependencies

```bash
npm install
```

This installs all dependencies listed in `package.json`.

---

## 3.4 Create Environment Variables

Create `.env` file:

```bash
nano .env
```

Example configuration:

```
PORT=4000
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/app
JWT_SECRET=your_secret
FRONTEND_URL=https://yourdomain.com
```

Save file:

```
CTRL + X
Y
ENTER
```

---

## 3.5 Install PM2 (Process Manager)

PM2 keeps your Node server running in production.

Install globally:

```bash
npm install -g pm2
```

Start backend server:

```bash
pm2 start server.js --name project-backend
```

Check running processes:

```bash
pm2 list
```

---

## 3.6 Enable Auto Start

Start PM2 on server reboot.

```bash
pm2 startup
pm2 save
```

---

## 3.7 Configure Firewall

Check firewall status:

```bash
sudo ufw status
```

Enable firewall:

```bash
sudo ufw enable
```

Allow SSH:

```bash
sudo ufw allow OpenSSH
```

Allow backend port:

```bash
sudo ufw allow 4000
```

---

# 4️⃣ Deploying React Frontend

React apps must be **built into static files** before deployment.

---

## 4.1 Install Dependencies

Navigate to frontend:

```bash
cd /var/www/your-repo/frontend
```

Install packages:

```bash
npm install
```

---

## 4.2 Configure Environment Variables

If needed:

```bash
nano .env
```

Example:

```
VITE_SERVER_URL=https://api.yourdomain.com
```

---

## 4.3 Build React Application

```bash
npm run build
```

This creates the **dist folder** containing static files.

Example structure:

```
dist
 ├── index.html
 ├── assets
 └── css/js files
```

---

# 5️⃣ Installing and Configuring Nginx

Nginx acts as:

- Static file server
- Reverse proxy
- SSL termination

---

## 5.1 Install Nginx

```bash
sudo apt install -y nginx
```

Check status:

```bash
systemctl status nginx
```

---

## 5.2 Allow Nginx in Firewall

```bash
sudo ufw allow 'Nginx Full'
```

---

# 6️⃣ Configure Nginx for React Frontend

Create configuration file:

```bash
nano /etc/nginx/sites-available/yourdomain.com.conf
```

Example configuration:

```
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        root /var/www/your-repo/frontend/dist;
        try_files $uri /index.html;
    }
}
```

Enable site:

```bash
ln -s /etc/nginx/sites-available/yourdomain.com.conf /etc/nginx/sites-enabled/
```

Test configuration:

```bash
nginx -t
```

Restart nginx:

```bash
systemctl restart nginx
```

---

# 7️⃣ Configure Nginx Reverse Proxy for Backend

Create API config:

```bash
nano /etc/nginx/sites-available/api.yourdomain.com.conf
```

Configuration:

```
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable API site:

```bash
ln -s /etc/nginx/sites-available/api.yourdomain.com.conf /etc/nginx/sites-enabled/
```

Restart nginx:

```bash
systemctl restart nginx
```

---

# 8️⃣ Connect Domain to VPS

Go to your **domain DNS manager**.

Add A records:

| Type | Name | Value |
|------|------|------|
| A | @ | VPS_IP |
| A | www | VPS_IP |
| A | api | VPS_IP |

Example:

```
@ → 187.124.98.219
api → 187.124.98.219
```

DNS propagation may take **5–30 minutes**.

---

# 9️⃣ Setting Up SSL (HTTPS)

Install Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Generate SSL certificates:

```bash
certbot --nginx -d yourdomain.com -d www.yourdomain.com -d api.yourdomain.com
```

Certbot automatically:

- installs SSL certificate
- configures nginx
- enables HTTPS redirect

---

# 🔟 Verify SSL Auto-Renewal

SSL certificates expire every **90 days**.

Test renewal:

```bash
certbot renew --dry-run
```

---

# 🎯 Final Production Setup

Frontend

```
https://yourdomain.com
```

Backend API

```
https://api.yourdomain.com
```

Server

```
Hostinger VPS
```

Process Manager

```
PM2
```

Reverse Proxy

```
Nginx
```

Database

```
MongoDB
```

---

# 🛠️ Useful Production Commands

Restart backend

```bash
pm2 restart project-backend
```

Check logs

```bash
pm2 logs
```

Restart nginx

```bash
systemctl restart nginx
```

Check nginx config

```bash
nginx -t
```

---

# 📌 Conclusion

After completing this guide you will have:

- A **production-ready MERN application**
- **React frontend served via Nginx**
- **Node.js backend managed by PM2**
- **MongoDB database**
- **Secure HTTPS with SSL**
- **Domain and subdomain routing**

This setup is widely used for **real-world production deployments of MERN applications**.
