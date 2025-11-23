# [ID] - [Nom Service]

> 🔴/🟠/🟡/🟢 **PRIORITY** - [Description courte]
> Dernière mise à jour : [DATE]

---

## 📋 Vue d'ensemble

**Rôle** : [Description détaillée du service]

**Technologie** : [Stack technique principale]

**Impact downtime** : [Décrire impact]

**SLA** : < [X] min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: LXC Container / Virtual Machine
ID: [XXX]
OS: Ubuntu 24.04 LTS / Windows 11
RAM: [X]GB
CPU: [X] vCores
Disk: [X]GB
IP: 192.168.100.[X]
Network: vmbr1 (privé) / vmbr0 (public)

Services:
  ServiceName:
    - Port: [XXXX]
    - Version: [X.X.X]
    - Config: [...]
```

---

## 🚀 Déploiement

### Quick Start ([X] min)

```bash
# 1. Créer CT/VM
[Commandes Proxmox]

# 2. Installer dépendances
[Commandes apt/yum]

# 3. Configurer service
[Commandes config]

# 4. Démarrer
[Commandes start]

# 5. Vérifier
[Commandes check]
```

### Configuration Détaillée

[Détails configs, fichiers, etc.]

---

## 🔐 Sécurité

### Accès

```yaml
Users:
  - admin: [Role]
  - user1: [Role]

Authentication:
  - Type: [Password / 2FA / SSO]
  - Backend: [File / LDAP / OAuth]
```

### Firewall

```bash
# Ports ouverts
[Liste ports + règles]
```

---

## 📊 Monitoring

### Health Checks

```yaml
Uptime Kuma:
  - Type: HTTP(s) / TCP / Ping
  - URL/IP: [URL]
  - Interval: [X]s
  - Alert: [Discord / Email / SMS]
```

### Métriques

```yaml
Performance:
  - CPU: < [X]% normal
  - RAM: < [X]GB normal
  - Disk: < [X]% usage
  - Latency: < [X]ms
```

---

## 🆘 Troubleshooting

### Problème 1: [Description]

```bash
# Diagnostic
[Commandes debug]

# Solution
[Commandes fix]
```

### Problème 2: [Description]

```bash
# Diagnostic
[Commandes debug]

# Solution
[Commandes fix]
```

---

## 💾 Backup

### Script Backup

```bash
#!/bin/bash
# /root/scripts/backup-[service].sh

BACKUP_DIR="/var/backups/[service]"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup data
[Commandes backup]

# Rétention [X] jours
find $BACKUP_DIR -name "*" -mtime +[X] -delete
```

### Restore

```bash
# Restore depuis backup
[Commandes restore]
```

---

## 📝 Maintenance

### Mise à jour

```bash
# 1. Backup AVANT upgrade
[Script backup]

# 2. Update
[Commandes update]

# 3. Restart
[Commandes restart]

# 4. Vérifier
[Commandes check]
```

### Nettoyage

```bash
# Logs rotation
[Commandes rotation]

# Temp files cleanup
[Commandes cleanup]
```

---

## 🔗 Ressources

### Documentation

- **Official Docs** : [URL]
- **GitHub** : [URL]
- **Docker Hub** : [URL]

### Support

- **Community Forum** : [URL]
- **GitHub Issues** : [URL]

---

## ✅ Checklist Production

### Avant mise en production

- [ ] Service installé et démarre
- [ ] Config validée
- [ ] Sécurité OK (firewall, auth)
- [ ] Monitoring configuré
- [ ] Backup automatisé
- [ ] Tests fonctionnels OK
- [ ] Documentation à jour

### Tests réguliers

- [ ] Service répond
- [ ] Logs pas d'erreurs
- [ ] Ressources normales
- [ ] Backup récent existe

---

**Service maintenu avec ❤️ par François Danaels**  
**Version** : [X.X] - [DATE]  
**Statut** : ✅ Production / 🛠️ Dev / 📍 Staging