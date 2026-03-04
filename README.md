# 🚀 Production Deployment Guide (Node.js API + React Frontend with Nginx)


# 1️⃣ Server Requirements

* Ubuntu 20.04 / 22.04
* Nginx installed
* Node.js installed
* PM2 installed (recommended)

Install dependencies:

```bash
sudo apt update
sudo apt install nginx
sudo apt install nodejs npm
sudo npm install -g pm2
```

---

# 2️⃣ Deploy Node.js API

Assume:

* App runs on port `5000`
* Path: `/var/www/api`

Start API using PM2:

```bash
cd /var/www/api
pm2 start app.js --name api
pm2 startup
pm2 save
```

Confirm running:

```bash
pm2 status
```

---

# 3️⃣ Nginx Config – API (HTTP Only)

Create:

```bash
sudo nano /etc/nginx/sites-available/api.yourdomain.com
```

Paste:

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    server_tokens off;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;

        # WebSocket support (Socket.io ready)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 60s;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
```

Enable:

```bash
sudo ln -s /etc/nginx/sites-available/api.yourdomain.com /etc/nginx/sites-enabled/
```

---

# 4️⃣ Deploy React Frontend

Build project:

```bash
npm run build
```

Move build to server:

```bash
sudo mkdir -p /var/www/frontend
sudo cp -r build/* /var/www/frontend/
```

---

# 5️⃣ Nginx Config – React (HTTP Only)

Create:

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/frontend;
    index index.html;

    server_tokens off;

    # Cache static assets (1 year)
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # React Router support
    location / {
        try_files $uri /index.html;
    }

    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;
}
```

Enable:

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

---

# 6️⃣ Test & Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# 7️⃣ Firewall Setup

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

Make sure:

* DNS A record points to server IP
* Port 80 open in cloud firewall

---

# ✅ Now Your Setup Is Production Ready (HTTP)

✔ Reverse proxy
✔ WebSocket ready
✔ React router working
✔ Static caching enabled
✔ Gzip enabled
✔ PM2 auto restart
✔ Security headers minimal

---

# 🔐 8️⃣ Later: Add SSL (Production Upgrade)

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

Run for frontend:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Run for API:

```bash
sudo certbot --nginx -d api.yourdomain.com
```

Certbot will:

* Add `listen 443 ssl`
* Insert certificate paths
* Add HTTP → HTTPS redirect
* Reload Nginx
* Setup auto-renew

Test renewal:

```bash
sudo certbot renew --dry-run
```

---

# 🔥 Optional Production Hardening (Recommended)

Edit main config:

```bash
sudo nano /etc/nginx/nginx.conf
```

Inside `http {}` add:

```nginx
client_max_body_size 20M;
server_tokens off;
```

Reload:

```bash
sudo systemctl reload nginx
```

---

# 🎯 Architecture Overview

```
User → Nginx (80)
        ├── yourdomain.com → React Static
        └── api.yourdomain.com → Node.js (5000 via PM2)
```

After SSL:

```
User → Nginx (443 HTTPS)
        ├── Frontend
        └── API
```
