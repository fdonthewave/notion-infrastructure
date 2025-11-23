# CT 760 - Guacamole RDP Gateway

> 🔴 **CRITICAL** - Gateway RDP pour accès agents Madagascar
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : Passerelle RDP HTML5 clientless pour accès VMs agents depuis navigateur

**Technologie** : Apache Guacamole (Docker) + PostgreSQL

**Impact downtime** : ⚠️ **CRITIQUE** - Agents ne peuvent plus se connecter = arrêt total activité

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: LXC Container
ID: 760
OS: Ubuntu 24.04 LTS
RAM: 2GB
CPU: 2 vCores
Disk: 16GB
IP: 192.168.100.20
Network: vmbr1 (privé)

Services:
  Guacamole:
    - Port: 8080
    - Version: 1.5.x
    - Auth: PostgreSQL
  
  PostgreSQL:
    - Version: 15
    - Database: guacamole_db
    - Port: 5432 (local only)
```

---

## 🚀 Déploiement Quick Start (15 min)

```bash
# 1. Créer CT
pct create 760 local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst \
  --hostname guacamole \
  --memory 2048 \
  --cores 2 \
  --rootfs local-zfs:16 \
  --net0 name=eth0,bridge=vmbr1,ip=192.168.100.20/24,gw=192.168.100.1 \
  --features nesting=1

pct start 760
pct enter 760

# 2. Installer Docker + PostgreSQL
apt update && apt upgrade -y
apt install -y docker.io docker-compose postgresql-client

# 3. Structure
mkdir -p /opt/guacamole/{init,data}

# 4. Docker Compose
cat > /opt/guacamole/docker-compose.yml << 'EOF'
version: '3.8'
services:
  guacd:
    image: guacamole/guacd:latest
    container_name: guacd
    restart: unless-stopped
    volumes:
      - ./data/drive:/drive
      - ./data/record:/record

  postgres:
    image: postgres:15
    container_name: guacamole-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: guacamole_db
      POSTGRES_USER: guacamole_user
      POSTGRES_PASSWORD: CHANGE_ME_STRONG_PASSWORD
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d

  guacamole:
    image: guacamole/guacamole:latest
    container_name: guacamole
    restart: unless-stopped
    depends_on:
      - guacd
      - postgres
    ports:
      - "8080:8080"
    environment:
      GUACD_HOSTNAME: guacd
      POSTGRES_HOSTNAME: postgres
      POSTGRES_DATABASE: guacamole_db
      POSTGRES_USER: guacamole_user
      POSTGRES_PASSWORD: CHANGE_ME_STRONG_PASSWORD
EOF

# 5. Init SQL schema
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgres > /opt/guacamole/init/initdb.sql

# 6. Démarrer
cd /opt/guacamole
docker-compose up -d

# 7. Accès initial
# URL: http://192.168.100.20:8080/guacamole
# User: guacadmin
# Pass: guacadmin
# ⚠️ CHANGER IMMÉDIATEMENT !
```

---

## 👥 Connexions RDP Configurées

### VM 201 - Mission Alexandra

```yaml
Name: Alexandra - Windows 11
Protocol: RDP
Hostname: 192.168.100.101
Port: 3389
Username: alexandra
Password: [Stocké sécurisé]
Security: any
Ignore server certificate: true
Enable audio: false  # ⚠️ Limitation Proxmox VM
Enable printing: true
Enable drive: true
Drive path: /drive/alexandra
```

### VM 202 - Mission Francia

```yaml
Name: Francia - Windows 11
Protocol: RDP
Hostname: 192.168.100.102
Port: 3389
Username: francia
Password: [Stocké sécurisé]
Security: any
Ignore server certificate: true
Enable audio: false  # ⚠️ Limitation Proxmox VM
Enable printing: true
Enable drive: true
Drive path: /drive/francia
```

---

## 📊 Tracking Heures (PostgreSQL)

### Requête Heures Travaillées

```sql
-- Heures mois en cours par agent
SELECT 
  username,
  COUNT(*) as nb_sessions,
  SUM(EXTRACT(EPOCH FROM (end_date - start_date))/3600) as total_hours
FROM guacamole_connection_history
WHERE username = 'alexandra'
  AND start_date >= DATE_TRUNC('month', CURRENT_DATE)
  AND end_date IS NOT NULL
GROUP BY username;
```

### Script Python Extraction

```python
#!/usr/bin/env python3
# /opt/scripts/extract-hours.py

import psycopg2
import json
from datetime import datetime

conn = psycopg2.connect(
    host="192.168.100.20",
    database="guacamole_db",
    user="guacamole_user",
    password="PASSWORD"
)

cur = conn.cursor()
cur.execute("""
    SELECT username,
           SUM(EXTRACT(EPOCH FROM (end_date - start_date))/3600) as hours
    FROM guacamole_connection_history
    WHERE start_date >= DATE_TRUNC('month', CURRENT_DATE)
      AND end_date IS NOT NULL
    GROUP BY username
""")

results = {}
for row in cur.fetchall():
    results[row[0]] = round(row[1], 2)

print(json.dumps(results, indent=2))

cur.close()
conn.close()
```

---

## 🆘 Troubleshooting

### Connexion RDP fail

```bash
# 1. Vérifier VM agent accessible
ping 192.168.100.101

# 2. Tester RDP direct
nc -zv 192.168.100.101 3389

# 3. Logs Guacamole
docker logs guacamole --tail 100
docker logs guacd --tail 100

# 4. Vérifier credentials dans Guacamole UI
# Settings > Connections > Edit > Parameters
```

### Audio ne fonctionne pas

⚠️ **LIMITATION CONNUE** : Audio RDP impossible dans Guacamole sur VMs Proxmox

**Raison** : Proxmox VMs n'ont pas de périphérique audio virtuel compatible

**Solutions alternatives** :
1. Utiliser Rocket.Chat pour appels vocaux
2. Migration future Azure Virtual Desktop (Q4 2025)

---

**Service maintenu avec ❤️ par François Danaels**  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Production