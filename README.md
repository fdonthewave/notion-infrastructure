# 📚 Infrastructure Pro-Assistante - Documentation Technique

> Source de vérité GitHub pour l'infrastructure Proxmox de Pro-Assistante.fr
> Synchronisé avec Notion pour architecture long-terme

---

## 🎯 Vue d'ensemble

Ce repository contient la **documentation technique complète** de l'infrastructure Pro-Assistante :

- 🏗️ Architecture services (19 services production)
- 📋 Guides déploiement par service
- 🔧 Scripts maintenance automatisés
- 📊 Templates nouveaux services
- 🚨 Procédures incidents

---

## 🗂️ Structure Repository

```
notion-infrastructure/
├── README.md                          # Ce fichier
├── ARCHITECTURE.md                    # Vue globale infrastructure
│
├── services/                          # 📁 Documentation par service
│   ├── ct-700-nginx-proxy-manager.md
│   ├── ct-760-guacamole.md
│   ├── ct-870-filebrowser-authelia.md
│   ├── vm-201-mission-alexandra.md
│   ├── vm-202-mission-francia.md
│   └── vm-820-rocketchat.md
│
├── scripts/                           # 🛠️ Scripts maintenance
│   ├── backup/
│   ├── monitoring/
│   └── deployment/
│
├── templates/                         # 📋 Templates standardisés
│   ├── nouveau-service.md
│   ├── incident-report.md
│   └── adr-template.md
│
└── decisions/                         # 🎯 Architecture Decision Records
    ├── 2025-11-13-filebrowser-vs-nextcloud.md
    └── 2025-11-16-authelia-totp-filesystem.md
```

---

## 🚀 Quick Start

### Pour consulter la doc

```bash
# Cloner le repo
git clone https://github.com/fdonthewave/notion-infrastructure.git
cd notion-infrastructure

# Lire l'architecture globale
cat ARCHITECTURE.md

# Consulter un service spécifique
cat services/ct-870-filebrowser-authelia.md
```

### Pour contribuer

```bash
# Créer une branche
git checkout -b doc/nouveau-service

# Éditer fichiers
vim services/nouveau-service.md

# Commit + Push
git add .
git commit -m "📚 doc: Ajout service XYZ"
git push origin doc/nouveau-service
```

---

## 📋 Services Production (19)

### 🔴 CRITICAL - Productive agents

| ID | Service | Description | Statut |
|----|---------|-------------|--------|
| CT 700 | Nginx Proxy Manager | Point d'entrée unique | ✅ Prod |
| CT 760 | Guacamole | RDP Gateway agents | ✅ Prod |
| CT 870 | FileBrowser + Authelia | Partage fichiers + SSO | ✅ Prod |
| VM 201 | Mission Alexandra | Agent Madagascar | ✅ Actif |
| VM 202 | Mission Francia | Agent Madagascar | ✅ Actif |
| VM 820 | Rocket.Chat | Communication Pro-Assistante | ✅ Prod |

### 🟠 PRODUCTION - Time tracking

| ID | Service | Description | Statut |
|----|---------|-------------|--------|
| CT 860 | n8n principal | Workflows automation | ✅ Prod |
| CT 861 | n8n secondaire | Workflows Antoine | ✅ Prod |

### 🟡 INFRASTRUCTURE - Monitoring

| ID | Service | Description | Statut |
|----|---------|-------------|--------|
| CT 750 | RustDesk | Remote support | ✅ Prod |
| CT 850 | Uptime Kuma | Monitoring services | ✅ Prod |
| VM 600 | MeshCentral | Remote management | ✅ Prod |

### 🟢 AUTRES SERVICES

| ID | Service | Description | Statut |
|----|---------|-------------|--------|
| CT 800 | Chat Phone-Help | Communication Phone-Help | ✅ Prod |
| CT 810 | Meetily + Scriberr | Meetings + transcription | ✅ Prod |
| VM 100 | Agenda.direct | Agenda en ligne | ✅ Prod |
| VM 500 | Jitsi Meet | Visioconférence | ✅ Prod |
| VM 199 | Win Office Template | Template Windows 11 | 📦 Template |
| VM 200 | Tiny11 Template | Template Tiny11 | ⚠️ Obsolète |

---

**Architecture maintenue avec ❤️ depuis Seclin, France**  
**Contact** : François Danaels (fdonthewave)  
**Version** : 1.0 - 23 Nov 2025