# Google Cloud Üzerinde Docker, Nginx ve Certbot ile n8n Kurulumu

### Adım 1: Sistem Güncellemesi
```bash
sudo apt update
```

### Adım 2: Docker Kurulumu
```bash
sudo apt install docker.io
```

### Adım 3: Docker Servisini Başlatma
```bash
sudo systemctl start docker
```

### Adım 4: Docker Servisini Etkinleştirme
```bash
sudo systemctl enable docker
```

### Adım 5: Docker Volume Oluşturma
```bash
sudo docker volume create yedek
```

Bu komut, n8n verilerinizi saklamak için kullanılacak olan `yedek` adında bir Docker volume'ü oluşturacaktır. Bu volume, konteyner silinse bile verilerinizin güvenli bir şekilde saklanmasını sağlayacaktır.

### Adım 6: N8N Docker Konteynerini Çalıştırma
```bash
sudo docker run -d --restart unless-stopped \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="sizin-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://sizin-domain.com/" \
-e WEBHOOK_URL="https://sizin-domain.com/" \
-e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
-v yedek:/home/node/.n8n \
n8nio/n8n
```

### Adım 7: Nginx Kurulumu
```bash
sudo apt install nginx
```

### Adım 8: N8N için Nginx Yapılandırma Dosyası Oluşturma
```bash
sudo nano /etc/nginx/sites-available/n8n
```

Aşağıdaki içeriği ekleyin:
```nginx
server {
    listen 80;
    server_name sizin-domain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # WebSocket desteği için başlıklar
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # İstemci bilgilerini iletmek için ek başlıklar
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Kaydetmek ve çıkmak için:
```bash
CTRL+O, ENTER, CTRL+X
```

### Adım 9: Nginx Yapılandırmasını Etkinleştirme
```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
```

### Adım 10: Nginx Yapılandırmasını Test Etme
```bash
sudo nginx -t
```

### Adım 11: Nginx Servisini Yeniden Başlatma
```bash
sudo systemctl restart nginx
```

### Adım 12: SSL Sertifikaları için Certbot Kurulumu
```bash
sudo apt install certbot python3-certbot-nginx
```

### Adım 13: SSL Sertifikası Alma
```bash
sudo certbot --nginx -d sizin-domain.com
```

### Adım 14: Nginx Servisini Yeniden Başlatma
```bash
sudo systemctl restart nginx
```

************************************************************************************************************
************************************************************************************************************
************************************************************************************************************
************************************************************************************************************
************************************************************************************************************

### Adım 15: Yedekleme Kontrolü ve Konteyner Yeniden Kurulumu

Yedeklemenin başarılı olup olmadığını kontrol etmek için konteyneri silip tekrar kurma adımları:

**Dikkat:** Bu işlem mevcut çalışan n8n konteynerinizi silecektir. Ancak `yedek` volume'ünüz doğru şekilde yapılandırıldıysa, verileriniz bu volume'de saklandığı için kaybolmayacaktır.

**Alt Adım 1: Çalışan Konteynerleri Listeleme (İsteğe Bağlı Kontrol)**

```bash
sudo docker ps
```

Bu komut, çalışan konteynerlerin listesini gösterir. `n8n` adlı konteynerinizin listede olduğundan emin olun.

**Alt Adım 2: n8n Konteynerini Durdurma**

```bash
sudo docker stop n8n
```

Bu komut, çalışan `n8n` konteynerini durduracaktır.

**Alt Adım 3: n8n Konteynerini Silme**

```bash
sudo docker rm n8n
```

Bu komut, durdurulmuş olan `n8n` konteynerini sisteminizden tamamen silecektir.

**Alt Adım 4: n8n Konteynerini Tekrar Oluşturma**

```bash
sudo docker run -d --restart unless-stopped \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="sizin-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://sizin-domain.com/" \
-e WEBHOOK_URL="https://sizin-domain.com/" \
-e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
-v yedek:/home/node/.n8n \
n8nio/n8n
```

Bu komut, daha önce kullandığınız ve `yedek` volume'ünü `/home/node/.n8n` dizinine bağlayan aynı komuttur. Bu sayede n8n, volume'deki mevcut verileri kullanacaktır.

Konteyner tekrar başlatıldıktan sonra (birkaç dakika sürebilir), `https://sizin-domain.com/` adresinden n8n arayüzüne tekrar erişmeyi deneyin. Eğer yedekleme başarılı olduysa, daha önce oluşturduğunuz workflow'ları ve credential'ları görmelisiniz.
