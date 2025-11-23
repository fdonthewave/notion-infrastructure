# 🤝 Contributing to notion-infrastructure

> Guide contribution documentation infrastructure Pro-Assistante

---

## 🎯 Objectif

Ce repository est la **source de vérité technique** de l'infrastructure. Toute contribution doit maintenir :

- ✅ **Précision** : Info exacte et à jour
- ✅ **Clarté** : Compréhensible par futur soi dans 6 mois
- ✅ **Exhaustivité** : Tout pour reproduire/troubleshoot
- ✅ **Sécurité** : Pas de secrets/passwords dans git

---

## 📝 Types de Contributions

### 1. Nouveau Service

```bash
# 1. Créer branche
git checkout -b doc/nouveau-service

# 2. Copier template
cp templates/nouveau-service.md services/ct-XXX-nom-service.md

# 3. Remplir template
vim services/ct-XXX-nom-service.md

# 4. Commit
git add services/ct-XXX-nom-service.md
git commit -m "📚 doc: Ajout CT XXX - Nom Service"

# 5. Push
git push origin doc/nouveau-service
```

**Checklist** :
- [ ] Template complètement rempli
- [ ] Commandes copy-paste testées
- [ ] Screenshots ajoutés si pertinent
- [ ] Liens resources vérifiés
- [ ] Pas de passwords/secrets

---

### 2. Mise à jour Service

```bash
git checkout -b update/ct-XXX
vim services/ct-XXX-nom-service.md
git commit -m "🔄 update: CT XXX version upgrade"
git push origin update/ct-XXX
```

**Bonnes pratiques** :
- Documenter **pourquoi** la modif (pas juste quoi)
- Ajouter date de modification
- Si breaking change, créer ADR

---

### 3. Incident Report

```bash
# Après résolution incident
cp templates/incident-report.md incidents/YYYY-MM-DD-titre-court.md
vim incidents/YYYY-MM-DD-titre-court.md
git commit -m "🚨 incident: Titre court incident"
```

**Timing** : Post-mortem dans les 24h après résolution

---

### 4. Architecture Decision Record (ADR)

```bash
cp templates/adr-template.md decisions/YYYY-MM-DD-titre-decision.md
vim decisions/YYYY-MM-DD-titre-decision.md
git commit -m "🎯 adr: Titre décision"
```

**Quand créer ADR** :
- Choix technologique important (ex: FileBrowser vs Nextcloud)
- Trade-off significatif accepté
- Décision impactant architecture long-terme
- Question "pourquoi on a fait X pas Y" revenue 3x

---

### 5. Scripts

```bash
# Créer script
vim scripts/backup/nouveau-script.sh
chmod +x scripts/backup/nouveau-script.sh

# Tester
./scripts/backup/nouveau-script.sh

# Commit
git add scripts/backup/nouveau-script.sh
git commit -m "🛠️ script: Description script"
```

**Standards scripts** :
- Header avec description + author + date
- `set -euo pipefail` obligatoire
- Logging vers `/var/log/scripts/`
- Error handling propre
- Comments pour logic complexe

---

## 💎 Conventions Commit

### Format

```
<emoji> <type>: <description courte>

<body optionnel>

<footer optionnel>
```

### Types

| Emoji | Type | Usage |
|-------|------|-------|
| 📚 | doc | Documentation service |
| 🎯 | adr | Architecture Decision Record |
| 🚨 | incident | Incident report |
| 🔄 | update | Mise à jour existant |
| 🛠️ | script | Nouveau script |
| 🐛 | fix | Correction erreur doc |
| ✨ | feat | Nouvelle feature/section |
| 🗑️ | cleanup | Nettoyage/refactor |

### Exemples

```bash
# Bon
git commit -m "📚 doc: CT 870 FileBrowser deployment guide"
git commit -m "🎯 adr: Choix PostgreSQL vs MySQL pour Guacamole"
git commit -m "🚨 incident: CT 700 NPM disk full 2025-11-23"

# Mauvais
git commit -m "update" # Pas assez descriptif
git commit -m "Fixed stuff" # Pas de emoji, vague
git commit -m "YOLO commit" # Non professionnel
```

---

## 🔒 Sécurité

### ❌ NE JAMAIS COMMITER

- Passwords (même chiffrés)
- API keys
- Tokens
- Certificats SSL privés (`.key`, `.pem`)
- Backups avec données réelles
- IPs publiques (masquer avec `X.X.X.X` si besoin)
- Emails clients (remplacer par `client@example.com`)

### ✅ OK à commiter

- Exemples configs (avec placeholders)
- Scripts sans secrets
- Architecture diagrams
- Logs anonymisés (pas d'IPs/emails)
- Screenshots floutés si nécessaire

### Placeholders Standards

```bash
# Passwords
password: CHANGE_ME_STRONG_PASSWORD

# IPs
IP: 192.168.100.XXX  # ou X.X.X.X

# Emails
email: user@example.com

# Domains
domain: example.com

# API Keys
api_key: YOUR_API_KEY_HERE
```

---

## ✅ Checklist Avant Commit

### Documentation Service

- [ ] Template complètement rempli
- [ ] Commandes testées copy-paste
- [ ] Pas de passwords/secrets
- [ ] Liens resources fonctionnels
- [ ] Date mise à jour actuelle
- [ ] Version logiciel précisée

### Script

- [ ] Header avec metadata
- [ ] `set -euo pipefail`
- [ ] Error handling
- [ ] Logging vers fichier
- [ ] Testé en staging/dev
- [ ] Pas de hardcoded secrets

### ADR

- [ ] Contexte clair
- [ ] Options comparées (min 2)
- [ ] Décision justifiée
- [ ] Trade-offs explicités
- [ ] Date de review définie

---

## 🔗 Ressources

- **Methodology** : [Notion Méthodologie Infrastructure 2.0](https://notion.so/2ad878e834f18118a114ce27205cac8e)
- **Claude Projects** : PROJETS CLAUDE MANAGER
- **Architecture** : [ARCHITECTURE.md](ARCHITECTURE.md)

---

## 💬 Questions ?

**Contact** : François Danaels  
**Discord** : #infra-pro-assistante  
**Email** : françois@pro-assistante.fr

---

**Version** : 1.0 - 23 Nov 2025  
**Maintenu avec ❤️ depuis Seclin, France**