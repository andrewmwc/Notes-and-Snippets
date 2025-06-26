# Raspberry Pi Web Server Setup Guide

This guide walks you through setting up a secure PHP and Node.js web server on a Raspberry Pi from a **fresh install**.

## 📅 Prerequisites

- Fresh Raspberry Pi OS (Bookworm/Bullseye)
- Static IP: `192.168.1.174`
- Internet connection

---

## 🔧 Step 0: Update the Pi

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

---

## 🌐 Step 1: Install Apache Web Server

```bash
sudo apt install apache2 -y
```

Test:

- Visit: [http://192.168.1.174](http://192.168.1.174)
- Should display "Apache2 Debian Default Page"

---

## 🧐 Step 2: Install PHP

```bash
sudo apt install php libapache2-mod-php -y
```

Test:

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Visit: [http://192.168.1.174/info.php](http://192.168.1.174/info.php)

---

## 🔒 Step 3: Enable HTTPS (Self-Signed)

### 3.1 Enable SSL Module

```bash
sudo a2enmod ssl
```

### 3.2 Generate SSL Certificate

```bash
sudo openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/private/selfsigned.key \
-out /etc/ssl/certs/selfsigned.crt
```

Common Name: `192.168.1.174`

### 3.3 Configure SSL Virtual Host

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Update:

```apache
SSLEngine on
SSLCertificateFile /etc/ssl/certs/selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/selfsigned.key
```

### 3.4 Enable SSL Site

```bash
sudo a2ensite default-ssl
sudo systemctl reload apache2
```

Visit: [https://192.168.1.174](https://192.168.1.174) (ignore browser warning)

---

## 🔄 Step 4: Redirect HTTP to HTTPS

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Replace with:

```apache
<VirtualHost *:80>
    Redirect permanent / https://192.168.1.174/
</VirtualHost>
```

```bash
sudo systemctl restart apache2
```

---

## 🟢 Step 5: Install Node.js (LTS)

### 5.1 Add NodeSource Repository

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

### 5.2 Install Node.js and npm

```bash
sudo apt install -y nodejs
```

### 5.3 Check Installation

```bash
node -v
npm -v
```

---

## 📊 Step 6: Create and Run Node.js App

```bash
mkdir ~/nodeapp && cd ~/nodeapp
nano server.js
```

Paste:

```js
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from Node.js on Raspberry Pi!\n');
});
server.listen(3000, () => {
  console.log('Server running at http://192.168.1.174:3000/');
});
```

Run:

```bash
node server.js
```

Visit: [http://192.168.1.174:3000](http://192.168.1.174:3000)

---

## ♻️ Step 7: Run Node.js in Background (Optional)

```bash
sudo npm install -g pm2
pm2 start server.js
pm2 startup
pm2 save
```

---

## 🔄 Step 8: Reverse Proxy Node.js via Apache (Optional)

### 8.1 Enable Apache Proxy Modules

```bash
sudo a2enmod proxy proxy_http rewrite
sudo systemctl restart apache2
```

### 8.2 Configure Proxy in SSL Site

```bash
sudo nano /etc/apache2/sites-available/default-ssl.conf
```

Add inside `<VirtualHost *:443>`:

```apache
ProxyPass "/node" "http://localhost:3000/"
ProxyPassReverse "/node" "http://localhost:3000/"
```

```bash
sudo systemctl restart apache2
```

Visit: [https://192.168.1.174/node](https://192.168.1.174/node)

---

## ✅ Complete!

| Component   | Status | URL                                                              |
| ----------- | ------ | ---------------------------------------------------------------- |
| Apache      | ✅      | [https://192.168.1.174](https://192.168.1.174)                   |
| PHP         | ✅      | [https://192.168.1.174/info.php](https://192.168.1.174/info.php) |
| Node.js     | ✅      | [http://192.168.1.174:3000](http://192.168.1.174:3000)           |
| Proxy (opt) | ✅      | [https://192.168.1.174/node](https://192.168.1.174/node)         |

---
