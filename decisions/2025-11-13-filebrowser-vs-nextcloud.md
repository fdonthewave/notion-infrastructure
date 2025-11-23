# ADR-001 : FileBrowser vs Nextcloud

> **Date** : 2025-11-13  
> **Statut** : ✅ Accepté  
> **Auteur** : François Danaels  
> **Context** : Setup partage fichiers Pro-Assistante

---

## 🎯 Contexte & Problème

Les agents Madagascar (Alexandra, Francia) ont besoin de partager des fichiers avec les clients français :
- Clients uploadent docs à traiter (Excel, Word, PDF)
- Agents téléchargent, traitent, puis re-uploadent résultats
- Besoin d'organisation par dossier client
- Accès sécurisé avec authentification

**Questions clés** :
- Quelle solution pour partage fichiers simple et efficace ?
- Faut-il une suite collaborative complète ?
- Quel niveau de complexité acceptable ?

**Contraintes** :
- Budget RAM limité (62GB total Proxmox)
- Setup rapide nécessaire (client Manekineko attend)
- Maintenance minimaliste (1 personne)
- Agents ont déjà Office sur VMs Windows

---

## 📊 Facteurs de Décision

1. **Complexité setup** : < 1h vs > 2h
2. **RAM usage** : Infrastructure limitée
3. **Maintenance** : Effort ongoing patches/updates
4. **Use case** : Upload/download vs collaboration temps réel
5. **Scalabilité** : 5 clients today, 30 clients 2026

---

## 📋 Options Considérées

### Option A : FileBrowser

**Description** : Application Go minimaliste, single binary, interface web simple

**Avantages ✅** :
- **Ultra simple** : 1 binaire Go, config minimal
- **Léger** : 512MB-1GB RAM suffisant
- **Setup rapide** : 10 min Docker run
- **Maintenance faible** : Auto-update simple
- **Suffisant** : Upload/download/organize = besoin couvert

**Inconvénients ❌** :
- Pas de collaboration temps réel
- Pas d'édition online documents
- Pas de mobile apps natives
- Features limitées vs Nextcloud

**Estimation** :
```yaml
Complexity: Low (1/5)
Time: 10 min setup
RAM: 512MB-1GB
Cost: 0€ (opensource)
Maintenance: Low (pull image mensuel)
```

---

### Option B : Nextcloud

**Description** : Suite collaborative complète (files, calendar, contacts, office online)

**Avantages ✅** :
- **Features riches** : Calendrier, contacts, tâches
- **Collaboration** : Édition online documents (OnlyOffice/Collabora)
- **Mobile apps** : iOS/Android natives
- **Maturité** : Large communauté, bien documenté
- **Extensible** : 200+ apps disponibles

**Inconvénients ❌** :
- **Complexe** : Stack Apache+PHP+Redis+DB
- **Lourd** : 4-6GB RAM minimum
- **Setup long** : 1-2h config + optimisation
- **Maintenance** : Patches fréquents, breaking changes
- **Overkill** : 90% features inutiles pour notre use case

**Estimation** :
```yaml
Complexity: High (4/5)
Time: 2h setup + 1h optimisation
RAM: 4-6GB
Cost: 0€ (opensource)
Maintenance: Medium (updates complexes)
```

---

### Option C : Seafile

**Description** : Alternative Nextcloud, focus performance

**Avantages ✅** :
- Plus rapide que Nextcloud
- Meilleure performance sync

**Inconvénients ❌** :
- Toujours complexe (client+server)
- 2-3GB RAM minimum
- Communauté plus petite
- Setup 1h+

**Estimation** :
```yaml
Complexity: Medium-High (3/5)
Time: 1h setup
RAM: 2-3GB
Cost: 0€ (opensource)
Maintenance: Medium
```

---

## 🎯 Décision

### ✅ Solution Choisie : **FileBrowser (Option A)**

**Raison principale** : Simple besoin = solution simple. Nextcloud apporte calendrier/contacts/office online = inutiles car agents ont déjà Office sur VMs Windows.

**Justification détaillée** :

1. **Use case analysis** : 
   - Clients uploadent Excel → FileBrowser OK
   - Agents téléchargent → FileBrowser OK
   - Agents traitent dans Office VM → Pas besoin office online
   - Agents re-uploadent → FileBrowser OK
   - ➡️ **Conclusion** : 100% use case couvert par FileBrowser

2. **RAM critique** : 
   - Proxmox 62GB total
   - Déjà alloué : ~50GB (VMs agents + services)
   - Restant : ~12GB buffer
   - FileBrowser 1GB vs Nextcloud 6GB = 🎯 5GB économisés

3. **Maintenance réaliste** : 
   - 1 personne (François) maintient infra
   - FileBrowser : `docker-compose pull && restart` = 2 min
   - Nextcloud : Lire changelog + tester + backup + migrate = 1h+

**Quote** : *"Perfect is the enemy of good. Ship simple solutions that work."*

---

## 📋 Trade-offs Acceptés

### Ce qu'on gagne ✅

- Setup en 10 min au lieu de 2h
- 5GB RAM économisés pour futures VMs agents
- Maintenance minimaliste
- Stabilité (moins de dépendances = moins de risques)

### Ce qu'on perd ⚠️

- **Pas de calendrier partagé** → **Mitigation** : Google Calendar gratuit suffit
- **Pas d'édition online** → **Acceptable car** : Agents ont Office sur VMs
- **Pas de mobile apps** → **Acceptable car** : Web responsive suffit
- **Moins de features** → **Acceptable car** : On en a pas besoin

---

## 📊 Impact

### Business

```yaml
Clients:
  - Impact: Accès fichiers immédiat (vs 2 jours attente Nextcloud)
  - UX: Simple, pas de courbe apprentissage

Agents:
  - Workflow: Identique (upload/download)
  - Training: 0 (interface intuitive)

Revenue:
  - Cost: 0€ (opensource)
  - ROI: Immédiat (client Manekineko débloqué)
```

### Technique

```yaml
Services affectés:
  - CT 870: Nouveau (FileBrowser)
  - CT 700: Config NPM proxy

Ressources:
  - RAM: +1GB (CT 870)
  - CPU: +1 vCore
  - Disk: +20GB

Migration:
  - Downtime: 0 (nouveau service)
  - Rollback: Facile (stop container)
  - Testing: 30 min
```

---

## 🛣️ Plan d'Exécution

### Phase 1 : Déploiement (DONE)

- [x] Créer CT 870 - François - 2025-11-13
- [x] Installer FileBrowser Docker - François - 2025-11-13
- [x] Config Authelia 2FA - François - 2025-11-16
- [x] Setup Nginx reverse proxy - François - 2025-11-16

### Phase 2 : Tests (DONE)

- [x] Tests upload/download
- [x] Tests 2FA TOTP
- [x] Tests client Manekineko
- [x] Performance OK

### Phase 3 : Production (DONE)

- [x] Config NPM SSL Let's Encrypt
- [x] Monitoring Uptime Kuma
- [x] Documentation GitHub
- [x] Backup automatique

---

## 🔄 Conditions de Revue

**Cette décision sera réévaluée si** :

- **Scaling > 30 clients** : Si volume fichiers nécessite Nextcloud performance
- **Besoin collaboration temps réel** : Si clients demandent édition online
- **Budget RAM > 128GB** : Si contrainte RAM disparait

**Review date** : 2026-05-01 (6 mois)

---

## 🔗 Références

### Documentation

- [FileBrowser Official](https://filebrowser.org)
- [Nextcloud Docs](https://docs.nextcloud.com)
- [Reddit r/selfhosted - FileBrowser vs Nextcloud](https://reddit.com/r/selfhosted)

### Benchmarks

- FileBrowser RAM usage: 300-800MB
- Nextcloud RAM usage: 4-6GB
- Setup time measured: 10 min vs 2h

---

## ✅ Validation

**Décision prise par** : François Danaels - 2025-11-13  
**Validée avec** : Claude (analyse systémique)  
**Implémentation** : CT 870 en production depuis 2025-11-16

**Résultat** : ✅ **SUCCÈS**
- Client Manekineko opérationnel
- RAM usage : 650MB (prévu 1GB)
- Stabilité : 99.9% uptime
- Satisfaction : Positive (simple, rapide)