# secure-nginx-web-server-module-03-assignment

# Secure Nginx Web Server with Reverse Proxy & SSL

This repository contains the configuration and setup instructions for deploying a secure, production-like web server using Nginx on an AWS EC2 Linux instance.

## 🎯 Project Overview
The goal of this project is to set up a secure web server that features:
* Static website hosting
* HTTPS utilizing a self-signed SSL certificate (OpenSSL)
* Automatic HTTP to HTTPS redirection
* Reverse proxy routing to a backend application running on port 3000

---

## 🛠️ Part 1: Basic Setup (Nginx & Static Site)

**1. Install Required Packages**
```bash
sudo apt update
sudo apt install -y nginx openssl
```

**2. Set Up the Static Website Directory**
```bash
sudo mkdir -p /var/www/secure-app
sudo chown -R ubuntu:www-data /var/www/secure-app
```

**3. Create the Static HTML Page**
```bash
echo "<h1>Secure Server Running via Nginx</h1>" | sudo tee /var/www/secure-app/index.html
```

## 🔐 Part 2: SSL Configuration

**Generate a Self-Signed SSL Certificate (Valid for 365 days)**
```bash
sudo mkdir -p /etc/nginx/ssl/

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/secure-app.key -out /etc/nginx/ssl/secure-app.crt
```
(Note: Prompts for geographic/organization information were left blank or filled with placeholder data).

**Output:**
```bash
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

## ⚙️ Part 3 & 4: Backend Setup & Nginx Configuration

**1. Simulate a Backend Application on Port 3000**
```bash
mkdir -p /tmp/backend
echo "<h1>Backend Running on Port 3000</h1>" > /tmp/backend/index.html
cd /tmp/backend
python3 -m http.server 3000 &
```

**2. Nginx Custom Configuration**
Created a new configuration file at `/etc/nginx/sites-available/secure-app`:
```nginx
server {
    # Redirect HTTP to HTTPS
    listen 80;
    server_name _; 
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name _;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/secure-app.crt;
    ssl_certificate_key /etc/nginx/ssl/secure-app.key;

    # Static Website Root
    root /var/www/secure-app;
    index index.html;

    # Serve Static Files
    location / {
        try_files $uri $uri/ =404;
    }

    # Reverse Proxy to Backend
    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
**3. Enable the Site and Reload Nginx**
```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/secure-app /etc/nginx/sites-enabled/secure-app
sudo nginx -t
sudo systemctl reload nginx
```

## ✅ Part 5: Testing & Proof of Execution
The following commands were run to verify the configuration:

- Test Nginx Syntax: `sudo nginx -t`
- Test HTTP to HTTPS Redirect: `curl -I http://localhost`
- Test HTTPS Static Site: `curl -k https://localhost`
- Test Reverse Proxy to Backend: `curl -k https://localhost/api/`

## 📸 Screenshots

1. HTTP to HTTPS Redirect Working

<img width="450" height="142" alt="Screenshot 2026-04-22 001855" src="https://github.com/user-attachments/assets/a46fc9aa-083f-492a-875d-22cab38d1a27" />

2. HTTPS Working (Static Site)

<img width="451" height="37" alt="Screenshot 2026-04-22 002029" src="https://github.com/user-attachments/assets/dfc97afd-8854-4096-93d4-24ac97c5f0b0" />

3. Backend Running via Reverse Proxy

<img width="487" height="52" alt="Screenshot 2026-04-22 002123" src="https://github.com/user-attachments/assets/03d5b648-dc0e-43d3-bc3c-a0b588676554" />





































