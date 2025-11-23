# 🏗️ Architecture Infrastructure Pro-Assistante

> Vue d'ensemble complète de l'infrastructure Proxmox Hetzner
> Dernière mise à jour : 23 Novembre 2025

---

## 📊 Vue Globale

### Infrastructure Physique

```yaml
Provider: Hetzner Dedicated Server
Model: AX102
CPU: AMD Ryzen 9 7950X (16 cores / 32 threads)
RAM: 62GB DDR5 ECC
Storage:
  - System: 2x 512GB NVMe SSD (RAID 1)
  - Data: 2x 2TB NVMe SSD (ZFS)
  - Backup: Hetzner Storage Box 5TB (BX31)

Hypervisor: Proxmox VE 8.3.2

Network:
  - WAN: vmbr0 (bridge public)
  - LAN: vmbr1 (bridge privé 192.168.100.0/24)
  - VPN: WireGuard (accès admin sécurisé)
```

---

## 🗺️ Topology Réseau

```
Internet 🌐
    ↓
CT 700 - Nginx Proxy Manager (Point d'entrée unique)
    ├─→ CT 870 - FileBrowser + Authelia (files.pro-assistante.fr)
    ├─→ VM 820 - Rocket.Chat (chat.pro-assistante.fr)
    └─→ CT 760 - Guacamole (rdp.pro-assistante.fr)
            ├─→ VM 201 - Mission Alexandra (Windows 11)
            └─→ VM 202 - Mission Francia (Windows 11)

CT 860/861 - n8n Workflows
    └─→ Automation (monitoring, tracking heures)

CT 850 - Uptime Kuma
    └─→ Monitoring tous services
```

---

## 📋 Inventaire Services

### 🔴 CRITICAL - Productive Agents (Priority 1)

| ID | Type | Service | IP | RAM | CPU | Disk | URLs |
|----|------|---------|----|----|-----|------|------|
| **CT 700** | LXC | **Nginx Proxy Manager** | 192.168.100.10 | 1GB | 1 | 8GB | `*.pro-assistante.fr` |
| **CT 760** | LXC | **Guacamole RDP Gateway** | 192.168.100.20 | 2GB | 2 | 16GB | `rdp.pro-assistante.fr` |
| **CT 870** | LXC | **FileBrowser + Authelia** | 192.168.100.25 | 2GB | 2 | 30GB | `files.pro-assistante.fr`<br/>`portail-*.pro-assistante.fr` |
| **VM 201** | VM | **Mission Alexandra** | 192.168.100.101 | 8GB | 4 | 100GB | RDP via Guacamole |
| **VM 202** | VM | **Mission Francia** | 192.168.100.102 | 8GB | 4 | 100GB | RDP via Guacamole |
| **VM 820** | VM | **Rocket.Chat Pro-Assistante** | 192.168.100.120 | 4GB | 2 | 50GB | `chat.pro-assistante.fr` |

**Impact downtime** : Agents Madagascar bloqués = perte facturation  
**SLA** : < 15 min intervention  
**Backup** : Quotidien automatique (00:00 CET)

---

### 🟠 PRODUCTION - Time Tracking & Billing (Priority 2)

| ID | Type | Service | IP | RAM | CPU | Disk | Description |
|----|------|---------|----|----|-----|------|-------------|
| **CT 860** | LXC | **n8n Principal** | 192.168.100.60 | 2GB | 2 | 20GB | Workflows automation principal |
| **CT 861** | LXC | **n8n Antoine** | 192.168.100.61 | 2GB | 2 | 20GB | Workflows automation secondaire |

**Services dépendants** :
- PostgreSQL Guacamole (logs RDP connexions)
- Script `tracking-hours.py` (CT 870)
- API `hours.php` (portails clients)

**Impact downtime** : Facturation imprécise  
**SLA** : < 30 min intervention  
**Backup** : Quotidien automatique (01:00 CET)

---

### 🟡 INFRASTRUCTURE - Monitoring & Support (Priority 3)

| ID | Type | Service | IP | RAM | CPU | Disk | URLs |
|----|------|---------|----|----|-----|------|------|
| **CT 750** | LXC | **RustDesk** | 192.168.100.15 | 1GB | 1 | 10GB | `rustdesk.pro-assistante.fr` |
| **CT 850** | LXC | **Uptime Kuma** | 192.168.100.50 | 1GB | 1 | 8GB | `status.pro-assistante.fr` |
| **VM 600** | VM | **MeshCentral** | 192.168.100.110 | 2GB | 2 | 20GB | `mesh.pro-assistante.fr` |

**Impact downtime** : Confort opérationnel réduit  
**SLA** : < 1h intervention  
**Backup** : Hebdomadaire (dimanche 02:00 CET)

---

### 🟢 AUTRES SERVICES (Priority 4)

| ID | Type | Service | IP | RAM | CPU | Disk | URLs |
|----|------|---------|----|----|-----|------|------|
| **CT 800** | LXC | **Chat Phone-Help** | 192.168.100.30 | 2GB | 2 | 20GB | `chat-phonehelp.pro-assistante.fr` |
| **CT 810** | LXC | **Meetily + Scriberr** | 192.168.100.40 | 2GB | 2 | 20GB | `meet.pro-assistante.fr` |
| **VM 100** | VM | **Agenda.direct** | 192.168.100.100 | 4GB | 2 | 50GB | `agenda.direct` |
| **VM 500** | VM | **Jitsi Meet** | 192.168.100.105 | 4GB | 2 | 30GB | `jitsi.pro-assistante.fr` |
| **VM 199** | VM | **Win Office Template** | - | 6GB | 2 | 80GB | Template clonage |
| **VM 200** | VM | **Tiny11 Template** | - | 4GB | 2 | 60GB | ⚠️ Obsolète |

---

## 🔐 Sécurité & Accès

### Couches de Sécurité

```
1. Firewall Hetzner (Robot Panel)
   └─> Ports ouverts : 80, 443, 22 (admin VPN only)

2. Nginx Proxy Manager (CT 700)
   └─> Reverse proxy + SSL Let's Encrypt
   └─> Rate limiting + fail2ban

3. Authelia SSO (CT 870)
   └─> 2FA TOTP obligatoire
   └─> Single Sign-On pour tous services

4. Guacamole (CT 760)
   └─> RDP Gateway sécurisé
   └─> Multi-factor authentication
   └─> Session recording

5. Proxmox VE
   └─> Accès admin VPN WireGuard uniquement
   └─> 2FA activé
```

---

## 💾 Stratégie Backup

### Backup Automatique Quotidien (00:00 CET)

```bash
# Script : /root/scripts/backup-prod.sh
# Cron : 0 0 * * * /root/scripts/backup-prod.sh

# Services CRITICAL (Priority 1)
vzdump 700 --mode snapshot --compress zstd  # NPM
vzdump 760 --mode snapshot --compress zstd  # Guacamole
vzdump 870 --mode snapshot --compress zstd  # FileBrowser
vzdump 201 --mode snapshot --compress zstd  # Alexandra
vzdump 202 --mode snapshot --compress zstd  # Francia
vzdump 820 --mode snapshot --compress zstd  # Rocket.Chat

# Services PRODUCTION (Priority 2)
vzdump 860 --mode snapshot --compress zstd  # n8n principal
vzdump 861 --mode snapshot --compress zstd  # n8n Antoine
```

### Rétention

| Priority | Local Proxmox | Storage Box Hetzner |
|----------|---------------|---------------------|
| 🔴 CRITICAL | 7 jours | 30 jours |
| 🟠 PRODUCTION | 3 jours | 14 jours |
| 🟡 INFRA | 1 jour | 7 jours |
| 🟢 AUTRES | Best effort | 3 jours |

### Stockage Backup

```yaml
Local Proxmox:
  Path: /var/lib/vz/dump/
  Espace: 500GB (ZFS compression)
  Rotation: Automatique selon rétention

Hetzner Storage Box BX31:
  Capacity: 5TB
  Mount: /mnt/backup-hetzner (CIFS)
  Sync: rclone sync quotidien (02:00 CET)
  Encryption: AES-256
```

---

## 🔄 Workflow Client Pro-Assistante

### Architecture Flux Documentaire

```
┌──────────────────────────────────────────────────────────┐
│  CLIENT FRANÇAIS (ex: Manekineko)                        │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Portail Web Personnalisé                         │  │
│  │  https://portail-manekineko.pro-assistante.fr     │  │
│  │                                                     │  │
│  │  • 💬 Chat Rocket.Chat (channel privé)            │  │
│  │  • 📁 FileBrowser (upload/download docs)          │  │
│  │  • 📅 Planning agent                               │  │
│  │  • ⏱️ Heures travaillées (auto-calculées)         │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
                        ↕️ HTTPS
┌──────────────────────────────────────────────────────────┐
│  INFRASTRUCTURE PRO-ASSISTANTE                           │
│                                                            │
│  CT 870 - FileBrowser + Authelia                         │
│  ├─ /mnt/partage-clients/manekineko/                     │
│  │   ├─ entrants/     (client → agent)                   │
│  │   └─ sortants/     (agent → client)                   │
│  └─ Authelia SSO : 2FA TOTP obligatoire                  │
│                                                            │
│  VM 820 - Rocket.Chat                                    │
│  └─ Channel #manekineko (privé, visible Alexandra only)  │
│                                                            │
│  CT 760 - Guacamole RDP Gateway                          │
│  └─ Connexion sécurisée Alexandra → VM 201               │
└──────────────────────────────────────────────────────────┘
                        ↕️ RDP over HTTPS
┌──────────────────────────────────────────────────────────┐
│  AGENT MADAGASCAR (ex: Alexandra)                        │
│  ┌────────────────────────────────────────────────────┐  │
│  │  VM 201 - Windows 11 Pro                          │  │
│  │                                                     │  │
│  │  • Rocket.Chat Desktop (communication)            │  │
│  │  • FileBrowser (navigateur web)                   │  │
│  │  • Microsoft Office 2021                          │  │
│  │  • Outils métier client                           │  │
│  │                                                     │  │
│  │  Workflow :                                        │  │
│  │  1. Télécharge docs depuis FileBrowser/entrants/  │  │
│  │  2. Traite fichiers Excel/Word                    │  │
│  │  3. Upload résultats dans FileBrowser/sortants/   │  │
│  │  4. Notifie client via Rocket.Chat               │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
                        ↕️
┌──────────────────────────────────────────────────────────┐
│  TRACKING AUTOMATIQUE                                     │
│                                                            │
│  PostgreSQL Guacamole (CT 760)                           │
│  └─ guacamole_connection_history                         │
│      ├─ username: alexandra                              │
│      ├─ start_date: 2025-11-23 08:15:00                  │
│      └─ end_date: 2025-11-23 12:30:00                    │
│                                                            │
│  Script tracking-hours.py (CT 870)                       │
│  └─ Query PostgreSQL → Calcul heures mois                │
│  └─ Expose JSON API → Portails clients                   │
│                                                            │
│  Portail Client                                           │
│  └─ Affiche : "142.5h travaillées ce mois"               │
└──────────────────────────────────────────────────────────┘
```

### Technologies Utilisées

| Fonction | Solution | Pourquoi |
|----------|----------|----------|
| **Partage fichiers** | FileBrowser | Simple, 2GB RAM vs Nextcloud 6GB |
| **SSO / 2FA** | Authelia | Opensource, supporte TOTP/WebAuthn |
| **Communication** | Rocket.Chat | Auto-hébergé, RGPD-compliant |
| **RDP Gateway** | Guacamole | Clientless HTML5, session recording |
| **Tracking heures** | PostgreSQL + Python | Direct depuis logs Guacamole |
| **Portails web** | HTML + PHP + Nginx | Simple, duplicable en 10 min |

---

## 📊 Monitoring & Alerting

### Uptime Kuma (CT 850)

**Checks actifs** :

```yaml
Services CRITICAL (check 30s):
  - CT 700 NPM: https://pro-assistante.fr (HTTP 200)
  - CT 760 Guacamole: https://rdp.pro-assistante.fr (HTTP 200)
  - CT 870 FileBrowser: https://files.pro-assistante.fr (HTTP 200)
  - VM 820 Rocket.Chat: https://chat.pro-assistante.fr (HTTP 200)
  - VM 201 Alexandra: RDP port 3389 (TCP)
  - VM 202 Francia: RDP port 3389 (TCP)

Services PRODUCTION (check 1min):
  - CT 860 n8n: https://n8n.pro-assistante.fr (HTTP 200)
  - CT 861 n8n-antoine: https://n8n-antoine.pro-assistante.fr (HTTP 200)

Infrastructure (check 5min):
  - Proxmox VE: https://proxmox.local:8006 (HTTPS)
  - Hetzner Storage Box: ping backup.hetzner.com
```

**Alertes** :

```yaml
Notifications:
  - Discord webhook (canal #alertes-infra)
  - Email François (françois@pro-assistante.fr)
  - SMS si CRITICAL down > 5 min (Twilio)

Escalation:
  - CRITICAL down > 15 min → Téléphone François
  - PRODUCTION down > 1h → Email rapport détaillé
```

---

## 🚀 Scaling Plan 2025

### Objectifs

- **Current** : 2 agents Madagascar, ~5 clients actifs
- **Target Q2 2025** : 5 agents, 15 clients
- **Target Q4 2025** : 10 agents, 30 clients

### Infrastructure Needs

#### Court Terme (Q1 2025)

```yaml
Nouvelles VMs agents:
  - VM 203: Mission Agent#3 (8GB RAM, 4 vCPU)
  - VM 204: Mission Agent#4 (8GB RAM, 4 vCPU)
  - VM 205: Mission Agent#5 (8GB RAM, 4 vCPU)

RAM upgrade:
  - Current: 62GB utilisés / 64GB total
  - Upgrade: +64GB → 128GB total (modules DDR5)
  - Budget: ~400€

Storage:
  - Current: 4TB NVMe (2TB utilisés)
  - Monitoring: Si > 70% → Upgrade ZFS pool
```

#### Moyen Terme (Q2-Q3 2025)

```yaml
Second server Proxmox:
  - Hetzner AX102 #2
  - Cluster Proxmox HA
  - Load balancing agents VMs
  - Budget: ~100€/mois

Storage Box upgrade:
  - Current: BX31 5TB
  - Upgrade: BX41 10TB
  - Budget: +10€/mois
```

---

## 📚 Décisions Architecturales

### ADR-001 : FileBrowser vs Nextcloud (2025-11-13)

**Contexte** : Besoin partage fichiers clients/agents

**Décision** : FileBrowser ✅

**Raison** :
- Simple : 1 binaire Go vs stack Apache+PHP+Redis
- Léger : 2GB RAM vs 6GB Nextcloud
- Setup : 10 min vs 2h Nextcloud
- Use case : Upload/download Excel suffit

**Trade-offs assumés** :
- Pas de collaboration temps réel (pas besoin)
- Pas de mobile apps natives (web suffit)

---

### ADR-002 : Authelia TOTP Filesystem (2025-11-16)

**Contexte** : 2FA clients FileBrowser, migration Brevo SMTP pas prête

**Décision** : Génération TOTP CLI temporaire ✅

**Raison** :
- Fonctionnel immédiatement
- Client Manekineko débloqué
- Tests production réussis

**Trade-offs assumés** :
- Intervention manuelle admin (non scalable)
- Migration Brevo SMTP prévue Q1 2025

---

### ADR-003 : Portails HTML vs React Dashboard (2025-11-16)

**Contexte** : Interface web clients pour accès services

**Décision** : HTML + PHP API ✅

**Raison** :
- Simple : 4 liens + 1 stat heures
- Dev time : 2h par portail vs 3 jours React
- Duplication : Copy/paste template 5 min

---

## 🔗 Ressources

### Documentation Interne

- **Notion** : [Méthodologie Infrastructure 2.0](https://notion.so/2ad878e834f18118a114ce27205cac8e)
- **Claude Projects** : PROJETS CLAUDE MANAGER
- **GitHub** : [notion-infrastructure](https://github.com/fdonthewave/notion-infrastructure)

### Documentation Technique

- **Proxmox VE** : https://pve.proxmox.com/wiki/Main_Page
- **Guacamole** : https://guacamole.apache.org/doc/gug/
- **FileBrowser** : https://filebrowser.org
- **Authelia** : https://www.authelia.com/docs/
- **Rocket.Chat** : https://docs.rocket.chat
- **n8n** : https://docs.n8n.io

---

**Architecture maintenue avec ❤️ depuis Seclin, France**  
**Contact** : François Danaels (fdonthewave)  
**Version** : 1.0 - 23 Novembre 2025  
**Dernière mise à jour** : 23 Nov 2025 - 18h50 CET