# 🛠️ Scripts Maintenance

> Scripts automatisation et maintenance infrastructure Pro-Assistante
> Dernière mise à jour : 23 Nov 2025

---

## 📁 Structure

```
scripts/
├── README.md                 # Ce fichier
├── backup/                   # Scripts backup automatisés
│   ├── backup-prod.sh        # Backup quotidien services CRITICAL
│   ├── backup-ct870.sh       # Backup CT 870 FileBrowser
│   ├── backup-rocketchat.sh  # Backup VM 820 Rocket.Chat
│   └── sync-storagebox.sh    # Sync Hetzner Storage Box
│
├── monitoring/               # Scripts monitoring & alertes
│   ├── check-services.sh     # Vérifier tous services UP
│   ├── disk-usage-alert.sh   # Alerte si disk > 80%
│   └── ram-usage-alert.sh    # Alerte si RAM > 90%
│
└── deployment/               # Scripts déploiement
    ├── deploy-service.sh     # Template déploiement service
    ├── create-client.sh      # Créer nouveau client (dossiers + portail)
    └── tracking-hours.py     # Extraction heures travaillées
```

---

## 🔄 Scripts Backup

### backup-prod.sh

**Description** : Backup quotidien automatique tous services CRITICAL

**Cron** : `0 0 * * * /root/scripts/backup/backup-prod.sh`

**Services** :
- CT 700 (NPM)
- CT 760 (Guacamole)
- CT 870 (FileBrowser)
- VM 201 (Alexandra)
- VM 202 (Francia)
- VM 820 (Rocket.Chat)

**Localisation** : `/var/lib/vz/dump/`

**Rétention** : 7 jours local, 30 jours Storage Box

---

### backup-ct870.sh

**Description** : Backup spécifique CT 870 (configs + partage clients)

**Cron** : `0 1 * * * /root/scripts/backup/backup-ct870.sh`

**Contenu** :
- /mnt/partage-clients/ (fichiers clients)
- /opt/filebrowser/ (configs)
- /opt/authelia/ (users DB)
- /var/www/portails/ (portails HTML)

**Localisation** : `/var/backups/ct870/`

---

### sync-storagebox.sh

**Description** : Synchronisation quotidienne Hetzner Storage Box 5TB

**Cron** : `0 2 * * * /root/scripts/backup/sync-storagebox.sh`

**Méthode** : rclone sync

**Encryption** : AES-256

---

## 📊 Scripts Monitoring

### check-services.sh

**Description** : Vérifie tous services CRITICAL sont UP

**Cron** : `*/5 * * * * /root/scripts/monitoring/check-services.sh`

**Checks** :
- CT 700 NPM : HTTP 200
- CT 760 Guacamole : HTTP 200
- CT 870 FileBrowser : HTTP 200
- VM 820 Rocket.Chat : HTTP 200
- VM 201/202 : RDP port 3389 ouvert

**Alerte** : Discord webhook si service down

---

### disk-usage-alert.sh

**Description** : Alerte si disk usage > 80%

**Cron** : `0 */6 * * * /root/scripts/monitoring/disk-usage-alert.sh`

**Checks** :
- Proxmox host `/` et `/var/lib/vz/`
- Tous CTs disk usage
- Tous VMs disk usage

**Alerte** : Email + Discord si > 80%

---

## 🚀 Scripts Déploiement

### create-client.sh

**Description** : Setup complet nouveau client (1 commande)

**Usage** :
```bash
./create-client.sh <client_name>

# Exemple
./create-client.sh manekineko
```

**Actions** :
1. Crée dossier `/mnt/partage-clients/<client>/`
2. Sous-dossiers `entrants/` et `sortants/`
3. Permissions 755
4. Crée user FileBrowser
5. Duplique portail HTML depuis template
6. Config Nginx vhost
7. Restart services

**Durée** : ~2 min

---

### tracking-hours.py

**Description** : Extraction heures travaillées depuis PostgreSQL Guacamole

**Usage** :
```bash
python3 tracking-hours.py --agent alexandra --month 2025-11
```

**Output** : JSON
```json
{
  "agent": "alexandra",
  "month": "2025-11",
  "total_hours": 142.5,
  "sessions": 23
}
```

**API** : Exposé via `/api/hours.php` pour portails clients

---

## 📝 Standards

### Header Script

```bash
#!/bin/bash
# Script: [nom].sh
# Description: [description courte]
# Author: François Danaels
# Date: YYYY-MM-DD
# Version: X.X

set -euo pipefail  # Fail on error, undefined vars, pipe fails

# Variables
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/scripts/$(basename $0 .sh).log"

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1" | tee -a $LOG_FILE >&2
    exit 1
}

# Main
log "Start script"
# ... code ...
log "End script"
```

---

## ✅ TODO

### Court terme

- [ ] Créer backup-prod.sh
- [ ] Créer sync-storagebox.sh
- [ ] Créer check-services.sh
- [ ] Créer create-client.sh
- [ ] Créer tracking-hours.py

### Moyen terme

- [ ] Tests automatiques scripts
- [ ] CI/CD GitHub Actions
- [ ] Monitoring exécution scripts
- [ ] Alertes fail scripts

### Long terme

- [ ] Migration scripts → Ansible playbooks
- [ ] Infrastructure as Code (Terraform)
- [ ] Automated testing

---

**Scripts maintenus avec ❤️ par François Danaels**  
**Version** : 1.0 - 23 Nov 2025  
**Localisation** : `/root/scripts/` (Proxmox host)