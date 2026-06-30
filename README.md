# PhaLel MinIO + PostgreSQL + Nginx Setup File

Ubuntu 24.04 + Docker Compose + Nginx + SSL

Example domains:

```text
MinIO API:     https://storage.example.com
MinIO Console: https://console.example.com
```

---

# 1. Update Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 2. Install Docker

```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Check Docker:

```bash
docker --version
docker compose version
```

---

# 3. Create Project Directory

```bash
sudo mkdir -p /opt/phalel
cd /opt/phalel
```

Create folders:

```bash
mkdir -p minio-data postgres-data backup
```

---

# 4. Create `.env`

```bash
nano .env
```

Paste:

```env
################################
# MinIO
################################

MINIO_ROOT_USER=phaleladmin
MINIO_ROOT_PASSWORD=ChangeThisStrongPassword123!

################################
# PostgreSQL
################################

POSTGRES_DB=phalel_db
POSTGRES_USER=phalel
POSTGRES_PASSWORD=ChangeThisStrongPassword123!
```

---

# 5. Create `docker-compose.yml`

```bash
nano docker-compose.yml
```

Paste:

```yaml
services:

  minio:
    image: minio/minio:latest
    container_name: phalel-minio
    restart: unless-stopped

    ports:
      - "9000:9000"
      - "9001:9001"

    env_file:
      - .env

    environment:
      MINIO_SERVER_URL: "https://storage.example.com"
      MINIO_BROWSER_REDIRECT_URL: "https://console.example.com"

    volumes:
      - ./minio-data:/data

    command: server /data --console-address ":9001"

  postgres:
    image: postgres:16
    container_name: phalel-postgres
    restart: unless-stopped

    ports:
      - "5432:5432"

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

    volumes:
      - ./postgres-data:/var/lib/postgresql/data
```

Start containers:

```bash
sudo docker compose up -d
sudo docker ps
```

---

# 6. Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

Remove default Nginx site:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo rm -f /etc/nginx/sites-available/default
```

---

# 7. Create Nginx Config

```bash
sudo nano /etc/nginx/sites-available/minio
```

Paste:

```nginx
server {
    listen 80;
    server_name storage.example.com;

    client_max_body_size 5G;

    location / {
        proxy_pass http://127.0.0.1:9000;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;
    }
}

server {
    listen 80;
    server_name console.example.com;

    client_max_body_size 5G;

    location / {
        proxy_pass http://127.0.0.1:9001;

        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable Nginx config:

```bash
sudo ln -s /etc/nginx/sites-available/minio /etc/nginx/sites-enabled/minio
sudo nginx -t
sudo systemctl restart nginx
```

---

# 8. Configure Firewall

```bash
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 9000
sudo ufw allow 9001
sudo ufw allow 5432
sudo ufw enable
sudo ufw status
```

Production recommendation:

```text
Do not expose PostgreSQL 5432 publicly.
Do not expose MinIO 9000 and 9001 publicly if Nginx is already configured.
```

For production, use:

```bash
sudo ufw delete allow 5432
sudo ufw delete allow 9000
sudo ufw delete allow 9001
```

---

# 9. Install SSL Certificate

Make sure DNS A records are already pointed to your VPS IP:

```text
storage.example.com → YOUR_VPS_IP
console.example.com → YOUR_VPS_IP
```

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Create SSL certificate:

```bash
sudo certbot --nginx \
-d storage.example.com \
-d console.example.com
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

Check certificates:

```bash
sudo certbot certificates
```

---

# 10. Restart MinIO + PostgreSQL

```bash
cd /opt/phalel

sudo docker compose down
sudo docker compose up -d
sudo docker ps
```

---

# 11. Access MinIO

MinIO API:

```text
https://storage.example.com
```

MinIO Console:

```text
https://console.example.com
```

Login:

```text
Username: phaleladmin
Password: ChangeThisStrongPassword123!
```

Create buckets:

```text
phalel-videos
phalel-images
phalel-avatars
phalel-documents
phalel-backups
```

---

# 12. Connect PostgreSQL

```bash
sudo docker exec -it phalel-postgres psql -U phalel -d phalel_db
```

PostgreSQL commands:

```sql
\l
\c phalel_db
\dt
SELECT COUNT(*) FROM "Video";
\q
```

---

# 13. PostgreSQL Backup

Create backup:

```bash
sudo docker exec phalel-postgres \
pg_dump -U phalel phalel_db \
> backup/phalel_db_$(date +%F).sql
```

Restore backup:

```bash
cat backup/phalel_db_YYYY-MM-DD.sql | \
sudo docker exec -i phalel-postgres \
psql -U phalel -d phalel_db
```

---

# 14. MinIO Backup

Create backup:

```bash
sudo tar -czf backup/minio_backup_$(date +%F).tar.gz minio-data
```

Restore backup:

```bash
sudo docker compose down

sudo rm -rf minio-data

sudo tar -xzf backup/minio_backup_YYYY-MM-DD.tar.gz

sudo docker compose up -d
```

---

# 15. Next.js Environment Example

For Next.js running outside Docker:

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@YOUR_SERVER_IP:5432/phalel_db"

MINIO_ENDPOINT=storage.example.com
MINIO_PORT=443
MINIO_USE_SSL=true

MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!

MINIO_BUCKET=phalel-videos
```

For Next.js running inside Docker:

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@postgres:5432/phalel_db"

MINIO_ENDPOINT=storage.example.com
MINIO_PORT=443
MINIO_USE_SSL=true

MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!

MINIO_BUCKET=phalel-videos
```

---

# 16. Useful Commands

Check Docker containers:

```bash
sudo docker ps
```

Check MinIO logs:

```bash
sudo docker logs phalel-minio
```

Check PostgreSQL logs:

```bash
sudo docker logs phalel-postgres
```

Check Nginx config:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

Restart Docker services:

```bash
cd /opt/phalel
sudo docker compose restart
```

Stop Docker services:

```bash
cd /opt/phalel
sudo docker compose down
```

---

# Final Project Structure

```text
/opt/phalel
├── docker-compose.yml
├── .env
├── minio-data/
├── postgres-data/
└── backup/
```

---

# Services

| Service | URL / Port |
|---|---|
| MinIO API | https://storage.example.com |
| MinIO Console | https://console.example.com |
| PostgreSQL | 5432 |
| Nginx HTTP | 80 |
| Nginx HTTPS | 443 |

---

# Deployment Complete

You now have:

- Ubuntu 24.04
- Docker Engine
- Docker Compose
- MinIO Object Storage
- PostgreSQL Database
- Nginx Reverse Proxy
- SSL Certificate
- Backup Support
- Next.js Ready Configuration
