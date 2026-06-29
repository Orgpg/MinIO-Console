# PhaLel MinIO + PostgreSQL Setup (Ubuntu 24.04 + Docker Compose)

ဒီ Setup က **One VPS** ပေါ်မှာ **MinIO + PostgreSQL** ကို တစ်ခါတည်း Run ဖို့ ပြင်ထားတဲ့ Version ဖြစ်ပါတယ်။

---

## 1. Update Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Install Docker

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

## 3. Create Project Folder

```bash
sudo mkdir -p /opt/phalel
cd /opt/phalel
```

Create data folders:

```bash
sudo mkdir -p minio-data postgres-data backup
```

---

## 4. Create `.env`

```bash
sudo nano .env
```

Paste:

```env
# MinIO
MINIO_ROOT_USER=phaleladmin
MINIO_ROOT_PASSWORD=ChangeThisStrongPassword123!

# PostgreSQL
POSTGRES_DB=phalel_db
POSTGRES_USER=phalel
POSTGRES_PASSWORD=ChangeThisStrongPassword123!
```

> Password ကို Production မှာ Strong Password ပြောင်းပါ။

---

## 5. Create `docker-compose.yml`

```bash
sudo nano docker-compose.yml
```

Paste:

```yaml
services:
  minio:
    image: minio/minio:latest
    container_name: phalel-minio
    restart: always
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
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
```

---

## 6. Start MinIO + PostgreSQL

```bash
sudo docker compose up -d
```

Check containers:

```bash
sudo docker ps
```

You should see:

```text
phalel-minio
phalel-postgres
```

---

## 7. Open Firewall

Testing အတွက်:

```bash
sudo ufw allow 9000
sudo ufw allow 9001
sudo ufw allow 5432
sudo ufw enable
```

> Production မှာ PostgreSQL port `5432` ကို Public မဖွင့်သင့်ပါ။ Backend နဲ့ Docker Network အတွင်းမှာပဲ သုံးတာ ပိုကောင်းပါတယ်။

---

## 8. Open MinIO Dashboard

Browser မှာဖွင့်ပါ:

```text
http://YOUR_SERVER_IP:9001
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

## 9. Connect PostgreSQL

Correct command:

```bash
sudo docker exec -it phalel-postgres psql -U phalel -d phalel_db
```

Database list ကြည့်ရန်:

```sql
\l
```

Current database ထဲက tables ကြည့်ရန်:

```sql
\dt
```

Exit:

```sql
\q
```

> Error မဖြစ်အောင် `-d phalel_db` ထည့်ဖို့လိုပါတယ်။ `psql -U phalel` ပဲရေးရင် `phalel` ဆိုတဲ့ database ကိုရှာပြီး မရှိရင် error တက်နိုင်ပါတယ်။

---

## 10. PostgreSQL Backup

Create backup:

```bash
sudo docker exec phalel-postgres pg_dump -U phalel phalel_db > backup/phalel_db_$(date +%F).sql
```

Restore backup:

```bash
cat backup/phalel_db_YYYY-MM-DD.sql | sudo docker exec -i phalel-postgres psql -U phalel -d phalel_db
```

---

## 11. MinIO Backup

Backup MinIO data folder:

```bash
sudo tar -czf backup/minio_backup_$(date +%F).tar.gz minio-data
```

Restore MinIO backup:

```bash
sudo docker compose down
sudo rm -rf minio-data
sudo tar -xzf backup/minio_backup_YYYY-MM-DD.tar.gz
sudo docker compose up -d
```

---

## 12. Useful Commands

Stop services:

```bash
sudo docker compose down
```

Restart services:

```bash
sudo docker compose restart
```

View logs:

```bash
sudo docker logs phalel-minio
sudo docker logs phalel-postgres
```

Remove and restart:

```bash
sudo docker compose down
sudo docker compose up -d
```

---

## 13. Next.js Connection Example

`.env.local`

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@YOUR_SERVER_IP:5432/phalel_db"

MINIO_ENDPOINT=YOUR_SERVER_IP
MINIO_PORT=9000
MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!
MINIO_BUCKET=phalel-videos
```

If Next.js API is running inside Docker, use service names:

```env
DATABASE_URL="postgresql://phalel:ChangeThisStrongPassword123!@postgres:5432/phalel_db"

MINIO_ENDPOINT=minio
MINIO_PORT=9000
MINIO_ACCESS_KEY=phaleladmin
MINIO_SECRET_KEY=ChangeThisStrongPassword123!
MINIO_BUCKET=phalel-videos
```

---

## Final Structure

```text
/opt/phalel
├── docker-compose.yml
├── .env
├── minio-data/
├── postgres-data/
└── backup/
```

✅ MinIO + PostgreSQL setup completed.
