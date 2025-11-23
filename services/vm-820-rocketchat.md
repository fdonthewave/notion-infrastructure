# VM 820 - Rocket.Chat Pro-Assistante

> 🔴 **CRITICAL** - Communication clients/agents
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : Plateforme communication temps réel entre clients français et agents Madagascar

**Technologie** : Rocket.Chat (Docker) + MongoDB + Traefik

**Impact downtime** : ⚠️ **CRITIQUE** - Communication bloquée

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: Virtual Machine
ID: 820
OS: Ubuntu 22.04 LTS
RAM: 4GB
CPU: 2 vCores
Disk: 50GB
IP: 192.168.100.120
Network: vmbr1 (privé)

Services:
  Rocket.Chat:
    - Version: 6.x
    - Port: 3000
    - Users: ~10-15
  
  MongoDB:
    - Version: 6.0
    - Database: rocketchat
    - Port: 27017 (local)
```

---

## 🚀 Déploiement Quick Start (20 min)

```bash
# 1. Créer VM depuis Proxmox UI
# ID: 820, RAM: 4GB, CPU: 2, Disk: 50GB
# Network: vmbr1, IP: 192.168.100.120

# 2. Installer Docker
ssh root@192.168.100.120
apt update && apt upgrade -y
apt install -y docker.io docker-compose

# 3. Structure
mkdir -p /opt/rocketchat

# 4. Docker Compose
cat > /opt/rocketchat/docker-compose.yml << 'EOF'
version: '3.8'
services:
  mongo:
    image: mongo:6.0
    container_name: rocketchat-mongo
    restart: unless-stopped
    volumes:
      - ./data/mongo:/data/db
    command: mongod --oplogSize 128 --replSet rs0

  mongo-init-replica:
    image: mongo:6.0
    command: >
      bash -c
        "for i in `seq 1 30`; do
          mongosh mongo/rocketchat --eval '
            rs.initiate({
              _id: \"rs0\",
              members: [ { _id: 0, host: \"mongo:27017\" } ]
            })'
          s=$$?;
          if [ $$s -eq 0 ]; then
            exit 0;
          fi;
          sleep 5;
        done;"
    depends_on:
      - mongo

  rocketchat:
    image: registry.rocket.chat/rocketchat/rocket.chat:latest
    container_name: rocketchat
    restart: unless-stopped
    depends_on:
      - mongo
    ports:
      - "3000:3000"
    environment:
      MONGO_URL: mongodb://mongo:27017/rocketchat?replicaSet=rs0
      MONGO_OPLOG_URL: mongodb://mongo:27017/local?replicaSet=rs0
      ROOT_URL: https://chat.pro-assistante.fr
      PORT: 3000
      DEPLOY_METHOD: docker
    volumes:
      - ./data/uploads:/app/uploads
EOF

# 5. Démarrer
cd /opt/rocketchat
docker-compose up -d

# 6. Setup initial
# URL: https://chat.pro-assistante.fr (via NPM CT 700)
# Créer admin, configurer SMTP, etc.
```

---

## 👥 Channels Configurés

### Clients

```yaml
#manekineko:
  Type: Private
  Members: 
    - alexandra (agent)
    - contact@manekineko.fr (client)
  Purpose: Communication client Manekineko

#general:
  Type: Public
  Purpose: Annonces générales
```

### Agents

```yaml
#team-agents:
  Type: Private
  Members: 
    - alexandra
    - francia
    - françois (admin)
  Purpose: Coordination équipe
```

---

## 💾 Backup

```bash
#!/bin/bash
# /root/scripts/backup-rocketchat.sh

BACKUP_DIR="/var/backups/rocketchat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup MongoDB
docker exec rocketchat-mongo mongodump --archive=/tmp/mongo-$DATE.archive
docker cp rocketchat-mongo:/tmp/mongo-$DATE.archive $BACKUP_DIR/

# Backup uploads
tar -czf $BACKUP_DIR/uploads-$DATE.tar.gz /opt/rocketchat/data/uploads

# Rétention 7 jours
find $BACKUP_DIR -name "*" -mtime +7 -delete
```

---

**Service maintenu avec ❤️ par François Danaels**  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Production