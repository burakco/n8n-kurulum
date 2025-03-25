# n8n-kurulum

# Google Cloud Setup  Docker and Nginx & Cerbot for n8n

### Step 1: Update the system
```bash
sudo apt update
```

### Step 2: Install Docker
```bash
sudo apt install docker.io
```

### Step 3: Start Docker service
```bash
sudo systemctl start docker
```

### Step 4: Enable Docker service
```bash
sudo systemctl enable docker
```

### Step 5: Run N8N Docker Container
```bash
sudo docker run -d --restart unless-stopped \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.burakco.net" \
-e WEBHOOK_TUNNEL_URL="https://n8n.burakco.net/" \
-e WEBHOOK_URL="https://n8n.burakco.net/" \
-e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
-v yedek:/home/node/.n8n \
n8nio/n8n
```

### Step 6: Install Nginx
```bash
sudo apt install nginx
```

### Step 7: Create an Nginx configuration file for N8N
```bash
sudo nano /etc/nginx/sites-available/n8n
```
Add the following content:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit using:
```bash
CTRL+O, ENTER, CTRL+X
```

### Step 8: Enable the Nginx configuration
```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
```

### Step 9: Test Nginx configuration
```bash
sudo nginx -t
```

### Step 10: Restart Nginx service
```bash
sudo systemctl restart nginx
```

### Step 11: Install Certbot for SSL Certificates
```bash
sudo apt install certbot python3-certbot-nginx
```

### Step 12: Obtain SSL Certificate
```bash
sudo certbot --nginx -d your-domain.com
```

### Step 14: Restart Nginx service
```bash
sudo systemctl restart nginx
```

### Step 15: Yedekleme Kontrolü ve Konteyner Yeniden Kurulumu

Elbette, yedeklemenin başarılı olup olmadığını görmek için konteyneri silip tekrar kurma adımlarını ve kodlarını aşağıda bulabilirsiniz:

**Dikkat:** Bu işlem mevcut çalışan n8n konteynerinizi silecektir. Ancak `yedek` volume'ünüz doğru şekilde yapılandırıldıysa, verileriniz bu volume'de saklandığı için kaybolmayacaktır.

**Adım 1: Çalışan Konteynerleri Listeleme (isteğe bağlı, ancak kontrol etmek iyi olabilir):**

```bash
sudo docker ps
```

Bu komut, çalışan konteynerlerin listesini gösterir. `n8n` adlı konteynerinizin listede olduğundan emin olun.

**Adım 2: n8n Konteynerini Durdurma:**

```bash
sudo docker stop n8n
```

Bu komut, çalışan `n8n` konteynerini durduracaktır.

**Adım 3: n8n Konteynerini Silme:**

```bash
sudo docker rm n8n
```

Bu komut, durdurulmuş olan `n8n` konteynerini sisteminizden tamamen silecektir.

**Adım 4: n8n Konteynerini Tekrar Oluşturma (Aynı `docker run` komutuyla):**

```bash
sudo docker run -d --restart unless-stopped \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.burakco.net" \
-e WEBHOOK_TUNNEL_URL="https://n8n.burakco.net/" \
-e WEBHOOK_URL="https://n8n.burakco.net/" \
-e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
-v yedek:/home/node/.n8n \
n8nio/n8n
```

Bu komut, daha önce kullandığınız ve `yedek` volume'ünü `/home/node/.n8n` dizinine bağlayan aynı komuttur. Bu sayede n8n, volume'deki mevcut verileri kullanacaktır.

**Adım 5: Verilerin Kontrol Edilmesi:**

Konteyner tekrar başlatıldıktan sonra (birkaç dakika sürebilir), `https://n8n.burakco.net/` adresinden n8n arayüzüne tekrar erişmeyi deneyin. Eğer yedekleme başarılı olduysa, daha önce oluşturduğunuz workflow'ları ve credential'ları görmelisiniz.

Lütfen bu adımları sırasıyla uygulayın ve sonucu benimle paylaşın. Eğer bir sorun yaşarsanız veya workflow ve credential'larınız görünmezse, lütfen bana bildirin.

