# VM 201 - Mission Alexandra

> 🔴 **CRITICAL** - Agent Madagascar actif
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : Poste de travail virtuel Windows pour agent Madagascar Alexandra

**Technologie** : Windows 11 Pro + Microsoft Office 2021

**Impact downtime** : ⚠️ **CRITIQUE** - Agent bloqué = clients non servis

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: Virtual Machine
ID: 201
OS: Windows 11 Pro (22H2)
RAM: 8GB
CPU: 4 vCores
Disk: 100GB
IP: 192.168.100.101
Network: vmbr1 (privé)

Logiciels installés:
  - Microsoft Office 2021 (Excel, Word, Outlook)
  - Rocket.Chat Desktop
  - Google Chrome
  - Adobe Reader
  - 7-Zip
  - Outils métier clients

Accès:
  - Guacamole: https://rdp.pro-assistante.fr
  - User: alexandra
  - Password: [Stocké KeePass]
```

---

## 🚀 Déploiement Initial

### 1. Créer VM depuis Template

```bash
# Clone depuis VM 199 (Template Win Office)
qm clone 199 201 --name mission-alexandra --full

# Config réseau
qm set 201 --net0 virtio,bridge=vmbr1

# Démarrer
qm start 201
```

### 2. Personnalisation Windows

```powershell
# 1. Renommer PC
Rename-Computer -NewName "ALEXANDRA-PC" -Restart

# 2. Créer user
net user alexandra PASSWORD /add
net localgroup Administrators alexandra /add

# 3. Configurer RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# 4. Installer Rocket.Chat Desktop
# Télécharger depuis https://rocket.chat/install
# Configurer: https://chat.pro-assistante.fr
```

### 3. Clients Assignés

```yaml
Clients actifs:
  - Manekineko:
      Dossier: /mnt/partage-clients/manekineko/
      Channel: #manekineko
      Type travail: Comptabilité, facturation
      
  - Client2:
      Dossier: /mnt/partage-clients/client2/
      Channel: #client2
      Type travail: Administratif
```

---

## 📅 Workflow Quotidien

### Matin (8h00 CET)

1. Connexion Guacamole
2. Ouvrir Rocket.Chat Desktop
3. Checker messages clients
4. Ouvrir FileBrowser → Télécharger docs "entrants/"

### Journée

5. Traiter fichiers Excel/Word clients
6. Communication via Rocket.Chat si questions
7. Upload résultats dans FileBrowser "sortants/"
8. Notifier client via Rocket.Chat : "Fichier X disponible"

### Soir (18h00 CET)

9. Rapport quotidien dans #team-agents
10. Déconnexion propre Windows

---

## 🆘 Troubleshooting

### VM ne démarre pas

```bash
# Check status
qm status 201

# Logs
qm terminal 201

# Force stop + start
qm stop 201
qm start 201
```

### RDP ne répond pas

```bash
# Depuis Proxmox host
ping 192.168.100.101
nc -zv 192.168.100.101 3389

# Console Proxmox UI
# Machines > 201 > Console > Vérifier Windows UP
```

### Office activation fail

```powershell
# Réactiver Office
cd "C:\Program Files\Microsoft Office\Office16"
cscript ospp.vbs /dstatus
cscript ospp.vbs /act
```

---

## 💾 Backup

```bash
# Snapshot quotidien (Proxmox)
vzdump 201 --mode snapshot --compress zstd

# Rétention: 7 jours local, 30 jours Storage Box
```

---

## 📊 Métriques

```yaml
Utilisation:
  - Heures travail: 8h-18h CET (lun-ven)
  - Moyenne connexion: 9h/jour
  - Clients assignés: 2-3 actifs

Performance:
  - RAM usage: ~4-5GB (normal)
  - CPU usage: 20-40% (normal)
  - Latency RDP: ~150-200ms (Madagascar)
```

---

**Agent** : Alexandra  
**Contact** : alexandra@pro-assistante.fr  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Actif