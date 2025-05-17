# Vue 3 + Vite

This template should help get you started developing with Vue 3 in Vite. The template uses Vue 3 `<script setup>` SFCs, check out the [script setup docs](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) to learn more.

Learn more about IDE Support for Vue in the [Vue Docs Scaling up Guide](https://vuejs.org/guide/scaling-up/tooling.html#ide-support).

## Misal mau nginx
Berikut solusi lengkap dari awal hingga deployment dengan HTTPS yang benar:

### 1. **Penyebab Error Let's Encrypt**
Let's Encrypt **tidak mengizinkan** sertifikat untuk domain AWS (*.amazonaws.com). Solusi:
```bash
# Solusi wajib:
- Beli domain murah (contoh: travel-appku.site ~$2/tahun)
- Atau gunakan subdomain gratis (misal: eu.5v.pl)

# Alternatif (tidak direkomendasikan):
- Ganti provider sertifikat: Cloudflare Free SSL
- Atau pakai self-signed certificate (akan ada peringatan di browser)
```

### 2. **Arsitektur Baru dengan Domain**
```
GitHub Actions (CI/CD)
│
├── Build & Push Docker Image → Docker Hub
│
└── Deploy to EC2:
     ├── Pull Image
     ├── Setup Nginx + SSL
     └── Renew Certificate
```

### 3. **Persiapan Domain & DNS**
1. Beli domain di Namecheap/GoDaddy
2. Buat DNS A Record:
   ```
   Type  Name  Value                 TTL
   A     @     54.164.49.129         300
   A     *     54.164.49.129         300
   ```

### 4. **Modifikasi Dockerfile & Nginx Config**
```dockerfile
# Dockerfile
FROM node:20 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
COPY ssl /etc/nginx/ssl  # Folder untuk sertifikat
```

```nginx
# nginx.conf (versi HTTPS)
events {}

http {
    include /etc/nginx/mime.types;
    
    server {
        listen 80;
        server_name travel-appku.site www.travel-appku.site;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name travel-appku.site www.travel-appku.site;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri $uri/ /index.html;
        }
    }
}
```

### 5. **Setup CI/CD dengan Auto SSL**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build & Push Docker
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKER_USER }}/travel-ui:latest

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - name: SSH & Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            # Setup awal (cukup 1x)
            sudo apt update
            sudo apt install -y certbot docker-compose
            sudo usermod -aG docker ubuntu
            
            # Generate certificate (jalan pertama kali)
            sudo certbot certonly --standalone -d travel-appku.site --non-interactive --agree-tos -m admin@travel-appku.site
            
            # Buat folder SSL di container
            sudo mkdir -p /home/ubuntu/ssl
            sudo cp /etc/letsencrypt/live/travel-appku.site/* /home/ubuntu/ssl/
            
            # Deploy dengan docker-compose
            docker-compose down || true
            docker-compose up -d
```

### 6. **docker-compose.yml di EC2**
```yaml
# /home/ubuntu/docker-compose.yml
version: '3.8'

services:
  web:
    image: $DOCKER_USER/travel-ui:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./ssl:/etc/nginx/ssl
    restart: unless-stopped
```

### 7. **Auto Renewal Certificate**
```bash
# Di EC2, buat cron job:
echo "0 3 * * * certbot renew --quiet && cp /etc/letsencrypt/live/travel-appku.site/* /home/ubuntu/ssl/ && docker restart travel-ui-web-1" | sudo tee -a /etc/cron.d/renew-cert
```

### 8. **Verifikasi**
```bash
# Cek SSL
curl -I https://travel-appku.site
openssl s_client -connect travel-appku.site:443 -servername travel-appku.site

# Cek container
docker ps
docker logs travel-ui-web-1
```

### Troubleshooting Tips:
1. **Certificate Permission Error**:
   ```bash
   sudo chmod 644 /home/ubuntu/ssl/*
   docker restart travel-ui-web-1
   ```

2. **DNS Not Propagated**:
   ```bash
   dig travel-appku.site +short
   # Jika belum muncul, tunggu 1-24 jam
   ```

3. **Nginx Config Error**:
   ```bash
   docker exec travel-ui-web-1 nginx -t
   ```

Dengan setup ini, setiap push ke branch main akan:
1. Build image baru
2. Deploy ke EC2
3. Otomatis renew certificate tiap 3 bulan
4. Aplikasi bisa diakses via HTTPS dengan domain proper