# PhaLel MinIO + PostgreSQL Setup (Ubuntu 24.04 + Docker Compose)

This guide explains how to deploy **MinIO** and **PostgreSQL** on a single Ubuntu 24.04 VPS using **Docker Compose**.

---

# 1. Update the Server

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

Verify the installation:

```bash
docker --version
docker compose version
```

---

# 3. Create the Project Directory

```bash
sudo mkdir -p /opt/phalel
cd /opt/phalel
```

Create the required data directories:

```bash
mkdir -p minio-data postgres-data backup
```

Project structure:

```text
/opt/phalel
├── docker-compose.yml
├── .env
├── minio-data/
├── postgres-data/
└── backup/
```

---

# 4. Create the Environment File

Create a new environment file:

```bash
nano .env
```

Paste the following configuration:

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

> Replace all default passwords before deploying to production.

---

# 5. Create docker-compose.yml

Create the Docker Compose file:

```bash
nano docker-compose.yml
```

Paste the following configuration:

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

Save the file.

---

# 6. Start the Containers

```bash
sudo docker compose up -d
```

Check running containers:

```bash
sudo docker ps
```

Expected output:

```text
phalel-minio
phalel-postgres
```

---

# 7. Configure the Firewall

Allow the required ports:

```bash
sudo ufw allow 9000
sudo ufw allow 9001
sudo ufw allow 5432

sudo ufw enable
```

Verify firewall rules:

```bash
sudo ufw status
```

> For production deployments, do **not** expose PostgreSQL (5432) to the public internet. Only allow internal access.

---

# 8. Access the MinIO Dashboard

Open your browser:

```text
http://YOUR_SERVER_IP:9001
```

Login credentials:

```text
Username:
phaleladmin

Password:
ChangeThisStrongPassword123!
```

Create the following buckets:

```text
phalel-videos
phalel-images
phalel-avatars
phalel-documents
phalel-backups
```

---

# 9. Connect to PostgreSQL

Open the PostgreSQL shell:

```bash
sudo docker exec -it phalel-postgres psql -U phalel -d phalel_db
```

List databases:

```sql
\l
```

Connect to the database:

```sql
\c phalel_db
```

List tables:

```sql
\dt
```

Show table structure:

```sql
\d "Video"
```

Display records:

```sql
SELECT * FROM "Video";
```

Count records:

```sql
SELECT COUNT(*) FROM "Video";
```

Exit PostgreSQL:

```sql
\q
```

---

# 10. PostgreSQL Backup

Create a backup:

```bash
sudo docker exec phalel-postgres \
pg_dump -U phalel phalel_db \
> backup/phalel_db_$(date +%F).sql
```

Restore a backup:

```bash
cat backup/phalel_db_YYYY-MM-DD.sql | \
sudo docker exec -i phalel-postgres \
psql -U phalel -d phalel_db
```

---

# 11. MinIO Backup

Create a backup:

```bash
sudo tar -czf backup/minio_backup_$(date +%F).tar.gz minio-data
```

Restore the backup:

```bash
sudo docker compose down

sudo rm -rf minio-data

sudo tar -xzf backup/minio_backup_YYYY-MM-DD.tar.gz

sudo docker compose up -d
```

---

# 12. Docker Management Commands

View running containers:

```bash
sudo docker ps
```

View logs:

```bash
sudo docker logs phalel-minio

sudo docker logs phalel-postgres
```

Restart all services:

```bash
sudo docker compose restart
```

Stop all services:

```bash
sudo docker compose down
```

Restart from scratch:

```bash
sudo docker compose down

sudo docker compose up -d
```

---

# 13. Next.js Environment Configuration

## Running Next.js Outside Docker

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@YOUR_SERVER_IP:5432/phalel_db"

MINIO_ENDPOINT=YOUR_SERVER_IP
MINIO_PORT=9000
MINIO_USE_SSL=false

MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!

MINIO_BUCKET=phalel-videos
```

---

## Running Next.js Inside Docker

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@postgres:5432/phalel_db"

MINIO_ENDPOINT=minio
MINIO_PORT=9000
MINIO_USE_SSL=false

MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!

MINIO_BUCKET=phalel-videos
```

---

# Final Project Structure

```text
/opt/phalel
│
├── docker-compose.yml
├── .env
│
├── minio-data/
│
├── postgres-data/
│
└── backup/
    ├── phalel_db_2026-06-29.sql
    └── minio_backup_2026-06-29.tar.gz
```

---

# Services

| Service       | Port |
| ------------- | ---- |
| MinIO API     | 9000 |
| MinIO Console | 9001 |
| PostgreSQL    | 5432 |

---

# Deployment Complete

You now have a Docker-based environment running:

* ✅ Ubuntu 24.04
* ✅ Docker Engine
* ✅ Docker Compose
* ✅ MinIO Object Storage
* ✅ PostgreSQL Database
* ✅ Persistent Data Volumes
* ✅ Backup Support
* ✅ Ready for Next.js Integration
