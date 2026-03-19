# VPS Deployment for Node.js Apps

A streamlined guide for deploying Node.js applications to a VPS (DigitalOcean Droplet, AWS EC2, Hetzner etc.). This setup is optimized for performance, security, and automatic process management and Docker-less.

## Overview

This repository provides a standard workflow for moving a Node.js application from a local environment to a production-ready Linux VPS (Ubuntu/Debian). It covers:

* **Process Management:** Keeping the app alive with PM2.
* **Reverse Proxy:** Using Nginx to handle incoming traffic and port forwarding.
* **Security:** SSL/TLS encryption via Let's Encrypt.
* **Firewall:** Basic UFW configuration to protect your server.

---

## Prerequisites

* A VPS running **Ubuntu 20.04+** or **Debian**.
* A domain name pointing to your VPS IP address (A Record).
* SSH access to your server.
* Node.js and npm/pnpm/yarn installed on the server.

---

## Installation

### 1. Server Preparation
Update your system and install essential dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx git -y
```

### 2. Install PM2
PM2 is a production process manager that ensures your app restarts if it crashes or the server reboots.
```bash
npm install pm2 -g
```

### 3. Clone and deploy application
Clone your repository and install dependencies:
```bash
cd /var/www
git clone [https://github.com/your-username/your-repo.git](https://github.com/your-username/your-repo.git)
cd your-repo
npm install
npm run build # If using TypeScript or a framework like Next.js
```

Start the application with PM2:
```bash
pm2 start npm --name "my-app" -- start
pm2 save
pm2 startup
```

### 4. Configure Nginx (Reverse Proxy)

Create a new Nginx configuration file:
```bash
sudo nano /etc/nginx/sites-available/my-app
```

Add the following configuration (replace `yourdomain.com` and port `3000` with your actual values):
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 5. Secure with SSL
Use Certbot to automatically fetch and configure an SSL certificate:
```
sudo apt install python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```

## Contributing
Contributions are welcome! If you have suggestions for better Nginx configs or automation scripts, please open an issue or a PR.

## License
This project is licensed under the MIT License.
