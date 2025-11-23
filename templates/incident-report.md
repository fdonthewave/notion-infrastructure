# 🚨 Incident Report - [TITRE COURT]

> Date incident : [YYYY-MM-DD HH:MM]
> Durée : [X] minutes
> Severité : 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low

---

## 📋 Résumé Exécutif

[Résumé en 2-3 phrases de ce qui s'est passé et de l'impact]

---

## 🕛 Timeline

| Heure | Événement |
|-------|------------|
| HH:MM | 🔴 **Détection** : [Description] |
| HH:MM | 🔍 Investigation : [Action] |
| HH:MM | 🔧 Intervention : [Action] |
| HH:MM | 📊 Validation : [Tests] |
| HH:MM | ✅ **Résolution** : Service restauré |

**Durée totale downtime** : [X] minutes

---

## 📊 Impact

### Services Affectés

- 🔴 **[Service 1]** : Complètement inaccessible
- 🟡 **[Service 2]** : Partiellement dégradé
- 🟢 **[Service 3]** : Non affecté

### Utilisateurs Impactés

```yaml
Agents:
  - Alexandra: Bloqué [X] minutes
  - Francia: Bloqué [X] minutes

Clients:
  - Client1: Impact [description]
  - Client2: Pas d'impact

Business:
  - Heures perdues: [X]h
  - Facturation impact: [X]€
```

---

## 🔍 Root Cause Analysis

### Cause Immédiate

[Description technique de ce qui a directement causé l'incident]

**Exemple** :
- Disque CT 870 plein (100% usage)
- Docker logs non rotationnés depuis 6 mois
- FileBrowser crash OOM

### Cause Racine

[Description de la cause profonde, souvent organisationnelle ou process]

**Exemple** :
- Pas de monitoring disk usage
- Pas de rotation logs automatique
- Pas d'alertes proactives

### Facteurs Contributeurs

- [Facteur 1]
- [Facteur 2]
- [Facteur 3]

---

## 🔧 Actions Prises

### Fix Immédiat (Emergency)

```bash
# 1. Libérer espace disque
cd /opt/filebrowser/data/logs
find . -name "*.log" -mtime +7 -delete

# 2. Restart service
docker-compose restart

# 3. Vérifier
curl -I https://files.pro-assistante.fr
```

### Fix Temporaire

[Si fix d'urgence pas optimal, décrire solution temporaire]

### Fix Permanent

[Solution définitive mise en place]

---

## 🔄 Actions Préventives

### Court Terme (cette semaine)

- [ ] **Monitoring** : Ajouter alerte disk usage > 80% (Uptime Kuma)
- [ ] **Logs rotation** : Configurer logrotate pour tous services Docker
- [ ] **Documentation** : Update runbook avec procédure "disk full"

### Moyen Terme (ce mois)

- [ ] **Automation** : Script cleanup logs quotidien (cron)
- [ ] **Alerting** : Notifications Discord #alertes-infra
- [ ] **Capacity planning** : Review disk usage tous services

### Long Terme (trimestre)

- [ ] **Architecture** : Centralized logging (Loki/Grafana)
- [ ] **Testing** : Chaos engineering disk full scenarios
- [ ] **Training** : Runbook incidents pour équipe

---

## 📚 Leçons Apprises

### Ce qui a bien fonctionné ✅

- [Point positif 1]
- [Point positif 2]

### Ce qui peut être amélioré 🔧

- [Point amélioration 1]
- [Point amélioration 2]

### Actions à prendre 🎯

1. **[Action 1]** - Responsable: [Nom] - Deadline: [Date]
2. **[Action 2]** - Responsable: [Nom] - Deadline: [Date]
3. **[Action 3]** - Responsable: [Nom] - Deadline: [Date]

---

## 📎 Annexes

### Logs Pertinents

```
[Coller logs clés]
```

### Screenshots

- [Lien screenshot 1]
- [Lien screenshot 2]

### Communications

**Discord #alertes-infra** :
```
[HH:MM] 🚨 INCIDENT CT 870 - FileBrowser down
[HH:MM] Investigation en cours...
[HH:MM] ✅ Service restauré
```

**Email clients** :
```
[Si clients ont été notifiés, copier email]
```

---

## ✅ Sign-off

**Incident Owner** : [Nom]
**Date Post-Mortem** : [YYYY-MM-DD]
**Reviewé par** : [Nom(s)]
**Approuvé** : [Oui/Non]

**Actions suivies** : [Lien Notion/GitHub Issues]

---

**Template v1.0 - 23 Nov 2025**  
**Adapté de Google SRE Workbook**