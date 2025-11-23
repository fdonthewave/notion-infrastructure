# CT 870 - FileBrowser + Authelia SSO

> 🔴 **CRITICAL** - Partage fichiers clients/agents + Single Sign-On
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : 
- Partage fichiers sécurisé entre clients français et agents Madagascar
- SSO avec 2FA TOTP pour tous services
- Portails web personnalisés par client

**Technologies** :
- FileBrowser (Docker)
- Authelia (SSO / 2FA)
- Nginx (reverse proxy local + portails HTML)
- PHP 8.2 (API tracking heures)

**Impact downtime** : ⚠️ **CRITIQUE** - Agents bloqués, pas d'accès docs

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: LXC Container
ID: 870
OS: Ubuntu 24.04 LTS
RAM: 2GB
CPU: 2 vCores
Disk: 30GB
IP: 192.168.100.25
Network: vmbr1 (privé)

Services:
  FileBrowser:
    - Port: 8080
    - Users: clients + agents
    - Storage: /mnt/partage-clients/
  
  Authelia:
    - Port: 9091
    - 2FA: TOTP filesystem (temporaire)
    - Backend: file (users_database.yml)
  
  Nginx:
    - Port: 80 (portails HTML clients)
    - Port: 8080 (proxy FileBrowser)
  
  PHP:
    - Version: 8.2-fpm
    - Script: /var/www/portails/api/hours.php
```

---

## 📁 Architecture Fichiers

```
CT 870
├── /mnt/partage-clients/          # Partage clients
│   ├── manekineko/
│   │   ├── entrants/              # Client → Agent
│   │   └── sortants/              # Agent → Client
│   ├── client2/
│   └── template/                  # Template nouveau client
│
├── /opt/filebrowser/              # FileBrowser Docker
│   ├── docker-compose.yml
│   ├── filebrowser.db             # Users + settings
│   └── config.json
│
├── /opt/authelia/                 # Authelia SSO
│   ├── docker-compose.yml
│   ├── configuration.yml
│   ├── users_database.yml         # Users + passwords
│   └── notification.txt           # TOTP codes (temp)
│
├── /var/www/portails/             # Portails clients HTML
│   ├── manekineko/
│   │   └── index.html             # Portail personnalisé
│   ├── api/
│   │   └── hours.php              # API heures travaillées
│   └── template/
│       └── index.html             # Template portail
│
└── /opt/scripts/
    └── tracking-hours.py          # Script tracking heures
```

---

## 🚀 Déploiement FileBrowser

### Quick Start (10 min)

```bash
# 1. Créer CT 870
pct create 870 local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst \
  --hostname filebrowser \
  --memory 2048 \
  --cores 2 \
  --rootfs local-zfs:30 \
  --net0 name=eth0,bridge=vmbr1,ip=192.168.100.25/24,gw=192.168.100.1 \
  --features nesting=1

pct start 870
pct enter 870

# 2. Installer Docker
apt update && apt upgrade -y
apt install -y docker.io docker-compose

# 3. Créer structure
mkdir -p /mnt/partage-clients/{manekineko/{entrants,sortants},template}
mkdir -p /opt/filebrowser

# 4. Docker Compose FileBrowser
cat > /opt/filebrowser/docker-compose.yml << 'EOF'
version: '3.8'
services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - /mnt/partage-clients:/srv
      - ./filebrowser.db:/database.db
      - ./config.json:/config/config.json
    environment:
      - FB_DATABASE=/database.db
      - FB_CONFIG=/config/config.json
EOF

# 5. Config FileBrowser
cat > /opt/filebrowser/config.json << 'EOF'
{
  "port": 80,
  "baseURL": "",
  "address": "0.0.0.0",
  "log": "stdout",
  "database": "/database.db",
  "root": "/srv"
}
EOF

# 6. Démarrer FileBrowser
cd /opt/filebrowser
docker-compose up -d

# 7. Accès initial
# URL: http://192.168.100.25:8080
# User: admin
# Pass: admin
# ⚠️ CHANGER IMMÉDIATEMENT !
```

---

## 🔐 Configuration Authelia

### Installation

```bash
# 1. Créer structure
mkdir -p /opt/authelia

# 2. Docker Compose
cat > /opt/authelia/docker-compose.yml << 'EOF'
version: '3.8'
services:
  authelia:
    image: authelia/authelia:latest
    container_name: authelia
    restart: unless-stopped
    ports:
      - "9091:9091"
    volumes:
      - ./configuration.yml:/config/configuration.yml
      - ./users_database.yml:/config/users_database.yml
      - ./notification.txt:/config/notification.txt
    environment:
      - TZ=Europe/Paris
EOF

# 3. Configuration principale
cat > /opt/authelia/configuration.yml << 'EOF'
server:
  host: 0.0.0.0
  port: 9091

log:
  level: info

authentication_backend:
  file:
    path: /config/users_database.yml
    password:
      algorithm: argon2id
      iterations: 1
      salt_length: 16
      parallelism: 8
      memory: 64

access_control:
  default_policy: deny
  rules:
    - domain: files.pro-assistante.fr
      policy: two_factor
    - domain: portail-*.pro-assistante.fr
      policy: two_factor

session:
  name: authelia_session
  secret: CHANGE_ME_SECRET_KEY_32_CHARS
  expiration: 3600
  inactivity: 300
  domain: pro-assistante.fr

storage:
  local:
    path: /config/db.sqlite3

notifier:
  filesystem:
    filename: /config/notification.txt
EOF

# 4. Créer users database
cat > /opt/authelia/users_database.yml << 'EOF'
users:
  manekineko:
    displayname: "Client Manekineko"
    password: "$argon2id$v=19$m=65536,t=3,p=4$HASH"  # Générer avec authelia hash-password
    email: contact@manekineko.fr
    groups:
      - clients
EOF

# 5. Démarrer Authelia
cd /opt/authelia
docker-compose up -d
```

### Générer Mot de Passe Authelia

```bash
docker run --rm authelia/authelia:latest authelia hash-password 'MotDePasseClient123'
# Copier le hash dans users_database.yml
```

### Générer Code TOTP (Temporaire)

```bash
# Lire le fichier notification.txt
docker exec authelia cat /config/notification.txt

# Copier le lien TOTP et ouvrir dans Google Authenticator
# Format: otpauth://totp/...
```

---

## 🌐 Portails Clients HTML

### Template Portail

```html
<!-- /var/www/portails/template/index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portail Client - Pro-Assistante</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        .container {
            background: white;
            border-radius: 20px;
            padding: 40px;
            max-width: 600px;
            width: 100%;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        h1 {
            color: #667eea;
            margin-bottom: 10px;
            font-size: 32px;
        }
        .subtitle {
            color: #666;
            margin-bottom: 30px;
            font-size: 14px;
        }
        .stats {
            background: #f7f9fc;
            border-radius: 10px;
            padding: 20px;
            margin-bottom: 30px;
            text-align: center;
        }
        .stats h2 {
            color: #667eea;
            font-size: 48px;
            margin-bottom: 5px;
        }
        .stats p {
            color: #666;
            font-size: 14px;
        }
        .links {
            display: grid;
            gap: 15px;
        }
        .link {
            display: flex;
            align-items: center;
            padding: 20px;
            background: #f7f9fc;
            border-radius: 10px;
            text-decoration: none;
            color: #333;
            transition: all 0.3s;
            border: 2px solid transparent;
        }
        .link:hover {
            background: #667eea;
            color: white;
            border-color: #667eea;
            transform: translateX(5px);
        }
        .link-icon {
            font-size: 32px;
            margin-right: 20px;
        }
        .link-content h3 {
            font-size: 18px;
            margin-bottom: 5px;
        }
        .link-content p {
            font-size: 14px;
            opacity: 0.7;
        }
        .footer {
            margin-top: 30px;
            text-align: center;
            color: #999;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎯 Portail CLIENT_NAME</h1>
        <p class="subtitle">Bienvenue sur votre espace Pro-Assistante</p>
        
        <div class="stats">
            <h2 id="hours">--.--h</h2>
            <p>Heures travaillées ce mois</p>
        </div>

        <div class="links">
            <a href="https://files.pro-assistante.fr" class="link" target="_blank">
                <span class="link-icon">📁</span>
                <div class="link-content">
                    <h3>Partage Fichiers</h3>
                    <p>Upload/download documents</p>
                </div>
            </a>

            <a href="https://chat.pro-assistante.fr/channel/CLIENT_CHANNEL" class="link" target="_blank">
                <span class="link-icon">💬</span>
                <div class="link-content">
                    <h3>Chat Direct</h3>
                    <p>Communication avec votre agent</p>
                </div>
            </a>

            <a href="#" class="link" onclick="alert('Planning disponible prochainement')">
                <span class="link-icon">📅</span>
                <div class="link-content">
                    <h3>Planning Agent</h3>
                    <p>Disponibilités et absences</p>
                </div>
            </a>

            <a href="https://status.pro-assistante.fr" class="link" target="_blank">
                <span class="link-icon">🟢</span>
                <div class="link-content">
                    <h3>Statut Services</h3>
                    <p>Monitoring infrastructure</p>
                </div>
            </a>
        </div>

        <div class="footer">
            <p>Pro-Assistante © 2025 - Support: françois@pro-assistante.fr</p>
        </div>
    </div>

    <script>
        // Fetch heures travaillées depuis API
        fetch('/api/hours.php?client=CLIENT_ID')
            .then(res => res.json())
            .then(data => {
                document.getElementById('hours').textContent = data.hours + 'h';
            })
            .catch(err => {
                console.error('Error loading hours:', err);
                document.getElementById('hours').textContent = '--h';
            });
    </script>
</body>
</html>
```

### Dupliquer Portail Client

```bash
# Créer portail Manekineko
cp -r /var/www/portails/template /var/www/portails/manekineko

# Personnaliser
sed -i 's/CLIENT_NAME/Manekineko/g' /var/www/portails/manekineko/index.html
sed -i 's/CLIENT_CHANNEL/manekineko/g' /var/www/portails/manekineko/index.html
sed -i 's/CLIENT_ID/manekineko/g' /var/www/portails/manekineko/index.html

# Configurer vhost Nginx (dans CT 700 NPM)
# Domain: portail-manekineko.pro-assistante.fr
# Forward: http://192.168.100.25:80
```

---

## 💾 Backup

```bash
#!/bin/bash
# /root/scripts/backup-ct870.sh

BACKUP_DIR="/var/backups/ct870"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup partage clients
tar -czf $BACKUP_DIR/clients-$DATE.tar.gz /mnt/partage-clients/

# Backup configs
tar -czf $BACKUP_DIR/configs-$DATE.tar.gz \
  /opt/filebrowser \
  /opt/authelia \
  /var/www/portails

# Rétention 7 jours
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

---

**Service maintenu avec ❤️ par François Danaels**  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Production