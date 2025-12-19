# Day 53: Going Live (VPS Deployment)

**Objective:** Move your code from `localhost` to `The Cloud`.
**Cost:** ~$5/month (DigitalOcean Droplet or Hetzner Cloud).

---

## 1. Renting the Server
1.  Go to DigitalOcean / Hetzner / Linode.
2.  Create a **Droplet** (VM).
3.  OS: **Ubuntu 22.04 LTS** (Standard).
4.  Specs: 2GB RAM (minimum for Browsers/Docker).
5.  Region: New York (or closest to your target).
6.  SSH Key: Upload your public key so you can login.

## 2. Setting up the Server
Open your terminal:
```bash
ssh root@YOUR_SERVER_IP
```

Inside the server, install Docker (One-liner):
```bash
curl -fsSL https://get.docker.com | sh
```

## 3. Deploying Code (The Manual Way)
1.  **Git Clone:**
    ```bash
    git clone https://github.com/yourname/scraper-api.git
    cd scraper-api
    ```
2.  **Run:**
    ```bash
    docker compose up -d --build
    ```
    (`-d` means Detached mode. It runs in the background).

## 4. Keeping it Alive
Docker handles restarts automatically if you added `restart: always` to your `docker-compose.yml`.
If the server reboots, your API comes back up.

## 5. Exposing to the World
BY default, `uvicorn` is on port 8000.
You want `http://api.yoursite.com`.

**Install Nginx (Reverse Proxy):**
```bash
apt install nginx
```

**Config:** `/etc/nginx/sites-available/default`
```nginx
server {
    listen 80;
    server_name api.yoursite.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Restart Nginx: `systemctl restart nginx`.

## 6. Homework
1.  Spend $5. Buy a VPS.
2.  Get your API running on a public IP address.
3.  Send the URL to a friend.
