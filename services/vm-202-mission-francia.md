# VM 202 - Mission Francia

> 🔴 **CRITICAL** - Agent Madagascar actif
> Dernière mise à jour : 23 Nov 2025

---

## 📋 Vue d'ensemble

**Rôle** : Poste de travail virtuel Windows pour agent Madagascar Francia

**Technologie** : Windows 11 Pro + Microsoft Office 2021

**Impact downtime** : ⚠️ **CRITIQUE** - Agent bloqué = clients non servis

**SLA** : < 15 min intervention

---

## 🔧 Spécifications Techniques

```yaml
Type: Virtual Machine
ID: 202
OS: Windows 11 Pro (22H2)
RAM: 8GB
CPU: 4 vCores
Disk: 100GB
IP: 192.168.100.102
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
  - User: francia
  - Password: [Stocké KeePass]
```

---

## 🚀 Déploiement Initial

### 1. Créer VM depuis Template

```bash
# Clone depuis VM 199 (Template Win Office)
qm clone 199 202 --name mission-francia --full

# Config réseau
qm set 202 --net0 virtio,bridge=vmbr1

# Démarrer
qm start 202
```

### 2. Personnalisation Windows

```powershell
# 1. Renommer PC
Rename-Computer -NewName "FRANCIA-PC" -Restart

# 2. Créer user
net user francia PASSWORD /add
net localgroup Administrators francia /add

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
  - Client3:
      Dossier: /mnt/partage-clients/client3/
      Channel: #client3
      Type travail: Secrétariat médical
      
  - Client4:
      Dossier: /mnt/partage-clients/client4/
      Channel: #client4
      Type travail: Gestion RH
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
qm status 202

# Logs
qm terminal 202

# Force stop + start
qm stop 202
qm start 202
```

### RDP ne répond pas

```bash
# Depuis Proxmox host
ping 192.168.100.102
nc -zv 192.168.100.102 3389

# Console Proxmox UI
# Machines > 202 > Console > Vérifier Windows UP
```

---

## 💾 Backup

```bash
# Snapshot quotidien (Proxmox)
vzdump 202 --mode snapshot --compress zstd

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

**Agent** : Francia  
**Contact** : francia@pro-assistante.fr  
**Version** : 1.0 - 23 Nov 2025  
**Statut** : ✅ Actif