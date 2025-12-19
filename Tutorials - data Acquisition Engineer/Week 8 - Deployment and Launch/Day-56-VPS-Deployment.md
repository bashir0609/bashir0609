# Day 56: The Cloud (VPS Deployment)

**Objective:** Move off localhost. Run 24/7.
**Cost:** ~$5/month.

---

## 1. The Provider
Don't use AWS EC2 (It's complex/expensive).
Use **DigitalOcean** or **Hetzner** or **Vultr**.
1.  Create Account.
2.  Create "Droplet" (Virtual Machine).
3.  OS: **Ubuntu 22.04**.
4.  Size: **2GB RAM / 1 CPU**. (Docker needs RAM).
5.  Region: Nearest to you.
6.  Auth: **SSH Key** (Upload your `id_rsa.pub`).

## 2. The Setup
Full process to go live:

```bash
# 1. Login
ssh root@123.45.67.89

# 2. Update
apt update && apt upgrade -y

# 3. Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# 4. Clone your code
git clone https://github.com/yourname/airline-scraper.git
cd airline-scraper

# 5. Config
nano .env # Paste your secrets

# 6. Launch
docker compose up -d --build
```

## 3. Persistent Data
Because we used `volumes:` in `docker-compose.yml`, if you restart the VPS, your Postgres data survives.

## 4. Homework
1.  Buy a $5 VPS.
2.  Deploy the stack.
3.  Access the Dashboard at `http://123.45.67.89:8501`.
4.  Send the link to a friend.
