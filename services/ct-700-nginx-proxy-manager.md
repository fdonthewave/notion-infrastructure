# CT 700 - Nginx Proxy Manager

> 🔴 **CRITICAL** - Point d'entrée unique de toute l'infrastructure
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : Reverse proxy central pour tous les services Pro-Assistante

**Technologie** : Nginx Proxy Manager (Docker)

**Impact downtime** : ⚠️ **CRITIQUE** - Tous services inaccessibles

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: LXC Container
ID: 700
OS: Ubuntu 24.04 LTS
RAM: 1GB
CPU: 1 vCore
Disk: 8GB
IP: 192.168.100.10
Network: vmbr1 (privé) + vmbr0 (WAN via NAT)

Docker:
  - nginx-proxy-manager:latest
  - Port 80 (HTTP)
  - Port 443 (HTTPS)
  - Port 81 (Admin UI)

Données:
  - /opt/npm/data (configs, certs)
  - /opt/npm/letsencrypt (SSL certificates)
```

---

## 🌐 Proxy Hosts Configurés

### Services Production

| Domain | Backend | SSL | 2FA |
|--------|---------|-----|-----|
| `files.pro-assistante.fr` | CT 870:8080 | ✅ Let's Encrypt | ✅ Authelia |
| `chat.pro-assistante.fr` | VM 820:3000 | ✅ Let's Encrypt | ❌ |
| `rdp.pro-assistante.fr` | CT 760:8080 | ✅ Let's Encrypt | ✅ Guacamole |
| `n8n.pro-assistante.fr` | CT 860:5678 | ✅ Let's Encrypt | ❌ |
| `status.pro-assistante.fr` | CT 850:3001 | ✅ Let's Encrypt | ❌ |

### Portails Clients

| Domain | Backend | SSL | 2FA |
|--------|---------|-----|-----|
| `portail-manekineko.pro-assistante.fr` | CT 870:80 | ✅ Let's Encrypt | ✅ Authelia |

---

## 🚀 Déploiement

### Quick Start (10 min)

```bash
# 1. Créer CT depuis Proxmox UI
pct create 700 local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst \
  --hostname npm \
  --memory 1024 \
  --cores 1 \
  --rootfs local-zfs:8 \
  --net0 name=eth0,bridge=vmbr1,ip=192.168.100.10/24,gw=192.168.100.1 \
  --nameserver 8.8.8.8 \
  --features nesting=1

# 2. Démarrer CT
pct start 700

# 3. Entrer dans CT
pct enter 700

# 4. Installer Docker
apt update && apt upgrade -y
apt install -y docker.io docker-compose
systemctl enable --now docker

# 5. Créer structure
mkdir -p /opt/npm/{data,letsencrypt}

# 6. Docker Compose
cat > /opt/npm/docker-compose.yml << 'EOF'
version: '3.8'
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    environment:
      - DISABLE_IPV6=true
EOF

# 7. Démarrer NPM
cd /opt/npm
docker-compose up -d

# 8. Vérifier
docker logs -f nginx-proxy-manager
```

### Accès Admin Initial

```
URL: http://192.168.100.10:81
Email: admin@example.com
Password: changeme

⚠️ CHANGER IMMÉDIATEMENT après premier login !
```

---

## 🔐 Configuration Sécurité

### SSL Let's Encrypt

```yaml
Settings:
  - Email: françois@pro-assistante.fr
  - Use DNS Challenge: Non (HTTP-01)
  - Auto-renew: Oui (30 jours avant expiration)
```

### Rate Limiting

```nginx
# Par IP
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req zone=general burst=20 nodelay;
```

### Fail2Ban (optionnel)

```bash
# Installer fail2ban
apt install -y fail2ban

# Config NPM
cat > /etc/fail2ban/filter.d/npm.conf << 'EOF'
[Definition]
failregex = ^.* 401 .* HTTP/.*" .*$
            ^.* 403 .* HTTP/.*" .*$
ignoreregex =
EOF

# Jail
cat > /etc/fail2ban/jail.d/npm.conf << 'EOF'
[npm]
enabled = true
port = http,https
filter = npm
logpath = /opt/npm/data/logs/*.log
maxretry = 5
bantime = 3600
findtime = 600
EOF

systemctl restart fail2ban
```

---

## 📊 Monitoring

### Health Checks

```bash
# Status service
docker ps | grep nginx-proxy-manager

# Logs temps réel
docker logs -f --tail 100 nginx-proxy-manager

# Vérifier SSL
curl -I https://files.pro-assistante.fr
```

### Uptime Kuma

```yaml
Check:
  - Type: HTTP(s)
  - URL: https://pro-assistante.fr
  - Interval: 30s
  - Retry: 3
  - Alert: Discord + Email si down > 2 min
```

---

## 🆘 Troubleshooting

### NPM ne démarre pas

```bash
# Vérifier logs
docker logs nginx-proxy-manager

# Problème courant : port 80/443 déjà utilisé
netstat -tulpn | grep ':80\|:443'

# Solution : kill process ou changer ports NPM

# Restart
cd /opt/npm
docker-compose restart
```

### SSL ne se renouvelle pas

```bash
# Vérifier certs
ls -lh /opt/npm/letsencrypt/live/

# Forcer renouvellement manuel
docker exec nginx-proxy-manager certbot renew --force-renewal

# Vérifier logs Let's Encrypt
docker exec nginx-proxy-manager cat /data/logs/letsencrypt-requests.log
```

### 502 Bad Gateway

```bash
# Vérifier backend accessible
curl http://192.168.100.25:8080  # Exemple CT 870

# Vérifier DNS résout
nslookup files.pro-assistante.fr

# Vérifier config proxy host dans NPM UI
# Settings > Proxy Hosts > Edit
```

---

## 💾 Backup

### Script Backup

```bash
#!/bin/bash
# /root/scripts/backup-npm.sh

BACKUP_DIR="/var/backups/npm"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup configs + certs
tar -czf $BACKUP_DIR/npm-$DATE.tar.gz \
  /opt/npm/data \
  /opt/npm/letsencrypt

# Rétention 7 jours
find $BACKUP_DIR -name "npm-*.tar.gz" -mtime +7 -delete

echo "✅ Backup NPM completed: npm-$DATE.tar.gz"
```

### Restore

```bash
# Stop NPM
cd /opt/npm
docker-compose down

# Restore depuis backup
tar -xzf /var/backups/npm/npm-YYYYMMDD_HHMMSS.tar.gz -C /

# Restart
docker-compose up -d
```

---

## 📝 Maintenance

### Mise à jour NPM

```bash
# 1. Backup AVANT upgrade
/root/scripts/backup-npm.sh

# 2. Pull latest image
cd /opt/npm
docker-compose pull

# 3. Restart avec nouvelle image
docker-compose down
docker-compose up -d

# 4. Vérifier logs
docker logs -f nginx-proxy-manager

# 5. Tester tous services
curl -I https://files.pro-assistante.fr
curl -I https://chat.pro-assistante.fr
# etc.
```

### Nettoyage Logs

```bash
# Logs NPM peuvent grossir
find /opt/npm/data/logs -name "*.log" -mtime +30 -delete

# Rotation automatique
cat > /etc/logrotate.d/npm << 'EOF'
/opt/npm/data/logs/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
EOF
```

---

## 🎯 Checklist Production

### Avant mise en production

- [ ] Docker + Docker Compose installés
- [ ] NPM démarre correctement
- [ ] Admin UI accessible (port 81)
- [ ] Password admin changé
- [ ] SSL Let's Encrypt configuré
- [ ] Tous proxy hosts créés
- [ ] Tests curl tous services OK
- [ ] Monitoring Uptime Kuma actif
- [ ] Backup automatique configuré
- [ ] Fail2ban activé (optionnel)

### Tests réguliers

- [ ] Tous domaines HTTPS valides
- [ ] Certificats SSL < 30 jours expiration
- [ ] Backend services répondent
- [ ] Logs pas d'erreurs critiques
- [ ] Disk usage CT 700 < 70%

---

## 🔗 Ressources

### Documentation

- **NPM Official** : https://nginxproxymanager.com/guide/
- **Docker Hub** : https://hub.docker.com/r/jc21/nginx-proxy-manager
- **GitHub** : https://github.com/NginxProxyManager/nginx-proxy-manager

### Support

- **GitHub Issues** : https://github.com/NginxProxyManager/nginx-proxy-manager/issues
- **Reddit r/selfhosted** : https://reddit.com/r/selfhosted

---

## 📈 Métriques

```yaml
Performance:
  - Latency proxy: < 50ms
  - SSL handshake: < 200ms
  - Requests/sec: ~100 (normal)

Ressources:
  - RAM: ~300MB (Docker + NPM)
  - CPU: < 10% normal
  - Disk: ~2GB (configs + logs)

Uptime:
  - Target: > 99.9%
  - Current: 99.95% (Q4 2024)
```

---

**Service maintenu avec ❤️ par François Danaels**  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Production