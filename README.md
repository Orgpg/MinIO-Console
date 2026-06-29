# PhaLel MinIO Server Setup (Ubuntu 24.04)

## 1. Update Server

``` bash
sudo apt update && sudo apt upgrade -y
```

------------------------------------------------------------------------

## 2. Install Docker

``` bash
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

------------------------------------------------------------------------

## 3. Create Project Folder

``` bash
sudo mkdir -p /opt/phalel
cd /opt/phalel
```

------------------------------------------------------------------------

## 4. Create `.env`

``` bash
sudo nano .env
```

Paste:

``` env
MINIO_ROOT_USER=phaleladmin
MINIO_ROOT_PASSWORD=ChangeThisStrongPassword123!
```

------------------------------------------------------------------------

## 5. Create `docker-compose.yml`

``` bash
sudo nano docker-compose.yml
```

Paste:

``` yaml
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
```

------------------------------------------------------------------------

## 6. Start MinIO

``` bash
sudo docker compose up -d
```

Check running containers:

``` bash
sudo docker ps
```

------------------------------------------------------------------------

## 7. Open Firewall

``` bash
sudo ufw allow 9000
sudo ufw allow 9001
sudo ufw enable
```

------------------------------------------------------------------------

## 8. Open Dashboard

    http://YOUR_SERVER_IP:9001

Login:

``` text
Username: phaleladmin
Password: ChangeThisStrongPassword123!
```

------------------------------------------------------------------------

## 9. Create Buckets

Create the following buckets:

``` text
phalel-videos
phalel-images
phalel-avatars
phalel-documents
phalel-backups
```

------------------------------------------------------------------------

## 10. Test Upload

Open **phalel-videos** bucket and upload a sample video.

✅ MinIO installation completed successfully.
