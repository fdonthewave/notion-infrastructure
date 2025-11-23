# ADR-002 : Authelia TOTP Filesystem (Temporaire)

> **Date** : 2025-11-16  
> **Statut** : ✅ Accepté (Temporaire)  
> **Auteur** : François Danaels  
> **Context** : 2FA FileBrowser sans SMTP

---

## 🎯 Contexte & Problème

CT 870 FileBrowser avec Authelia SSO est en place. Client Manekineko doit accéder avec 2FA TOTP obligatoire.

**Problème** : Authelia requiert notifier pour envoyer codes TOTP setup
- **Option classique** : SMTP email avec code TOTP
- **Situation actuelle** : Migration Brevo SMTP pas encore faite
- **Urgence** : Client attend accès aujourd'hui

**Questions clés** :
- Peut-on déployer 2FA sans SMTP ?
- Solution temporaire acceptable ?
- Quand migration SMTP ?

**Contraintes** :
- Client Manekineko bloquant business
- Pas le temps de configurer Brevo aujourd'hui (1-2h)
- Solution doit être sécurisée même si temporaire

---

## 📊 Facteurs de Décision

1. **Urgence** : Client attend aujourd'hui
2. **Sécurité** : 2FA doit rester fort
3. **Scalabilité** : Solution peux clients OK, 50 clients KO
4. **Réversibilité** : Migration vers SMTP facile ?

---

## 📋 Options Considérées

### Option A : Filesystem Notifier (TOTP CLI)

**Description** : Authelia écrit codes TOTP dans fichier local, admin lit et envoie manuellement

**Avantages ✅** :
- **Immédiat** : Fonctionne maintenant (0 config)
- **Simple** : 1 ligne config Authelia
- **Sécurisé** : TOTP reste cryptographiquement fort
- **Réversible** : Change config + restart = SMTP activé

**Inconvénients ❌** :
- **Non scalable** : Intervention manuelle par client
- **Ergonomie** : Admin doit lire fichier et envoyer
- **Temporaire** : Pas solution long-terme

**Estimation** :
```yaml
Complexity: Low (1/5)
Time: 5 min config
Scalability: 1-10 clients max
Maintenance: Manual par nouveau client
```

---

### Option B : Attendre Migration Brevo SMTP

**Description** : Configurer Brevo SMTP avant déployer 2FA

**Avantages ✅** :
- **Propre** : Solution définitive dès le départ
- **Scalable** : Emails automatiques
- **Pro** : Expérience utilisateur optimale

**Inconvénients ❌** :
- **Délai** : 1-2h config + tests Brevo
- **Client bloquant** : Manekineko attend
- **Risque** : Si Brevo fail, client toujours bloqué

**Estimation** :
```yaml
Complexity: Medium (3/5)
Time: 1-2h config Brevo
Scalability: Illimité
Delay: Client attend demain
```

---

### Option C : Pas de 2FA Temporairement

**Description** : Déployer FileBrowser sans 2FA, activer plus tard

**Avantages ✅** :
- **Rapide** : Client accède immédiatement
- **Simple** : Pas de config 2FA

**Inconvénients ❌** :
- ⚠️ **Sécurité réduite** : Password only
- **Changement workflow** : Activer 2FA plus tard = re-config clients
- **Risque** : Oubli d'activer 2FA

**Estimation** :
```yaml
Security: ⚠️ Medium risk
Complexity: Low
Time: 0 (déjà en place)
```

---

## 🎯 Décision

### ✅ Solution Choisie : **Option A - Filesystem Notifier (Temporaire)**

**Raison principale** : Client débloqué aujourd'hui avec sécurité 2FA maintenue. Migration SMTP prévue Q1 2025.

**Justification détaillée** :

1. **Business priority** : 
   - Manekineko attend accès = revenue bloquant
   - Filesystem = client opérationnel en 10 min
   - SMTP migration = peut attendre fin de semaine

2. **Sécurité maintenue** :
   - TOTP reste cryptographiquement sécurisé
   - Juste la livraison code qui change (fichier vs email)
   - Client reçoit quand même code fort 6 digits

3. **Scalabilité acceptable court-terme** :
   - 2-3 clients déployables manuellement
   - Migration SMTP avant scaling > 5 clients
   - 1 mois max pour migration

**Quote** : *"Done is better than perfect. Ship working 2FA today, optimize delivery later."*

---

## 📋 Trade-offs Acceptés

### Ce qu'on gagne ✅

- Client Manekineko opérationnel aujourd'hui
- 2FA sécurisé actif immédiatement
- Temps pour bien configurer Brevo SMTP
- Validation workflow 2FA en production

### Ce qu'on perd ⚠️

- **Intervention manuelle** → **Acceptable car** : 2-3 clients seulement
- **Pas scalable** → **Mitigation** : Migration SMTP Q1 2025
- **Expérience sous-optimale** → **Acceptable car** : Temporaire 1 mois

---

## 📊 Impact

### Business

```yaml
Clients:
  - Manekineko: Accès immédiat (vs 2 jours attente)
  - Expérience: Normale (reçoit code par autre canal)

Agents:
  - Impact: Zéro (utilisent pas FileBrowser avec 2FA)

Revenue:
  - Déblocage: 770€/mois Manekineko
  - Coût migration future: 1-2h
```

### Technique

```yaml
Services affectés:
  - CT 870 Authelia: Config filesystem notifier

Ressources:
  - RAM: 0 (même config)
  - CPU: 0
  - Disk: +1MB (fichier notification.txt)

Migration:
  - Downtime: 0 (change config + restart)
  - Rollback: Impossible (codes TOTP déjà générés)
  - Testing: 10 min
```

---

## 🛣️ Plan d'Exécution

### Phase 1 : Déploiement Filesystem (DONE)

- [x] Config Authelia filesystem notifier - 2025-11-16
- [x] Restart Authelia - 2025-11-16
- [x] Générer code TOTP Manekineko - 2025-11-16
- [x] Envoyer code client (email manuel) - 2025-11-16
- [x] Tests 2FA OK - 2025-11-16

### Phase 2 : Migration SMTP Brevo (TODO Q1 2025)

- [ ] Configurer Brevo SMTP - Décembre 2025
- [ ] Tester emails 2FA - Décembre 2025
- [ ] Migrer config Authelia - Décembre 2025
- [ ] Update doc GitHub - Décembre 2025

### Phase 3 : Validation

- [ ] Nouveaux clients reçoivent codes par email
- [ ] Procédure manuelle obsolète
- [ ] Scalable 50+ clients

---

## 🔄 Conditions de Revue

**Cette décision sera réévaluée** : **Décembre 2025** (1 mois)

**Triggers migration SMTP** :
- 5ème client demande accès (intervention manuelle trop fréquente)
- Brevo SMTP configuré pour autre usage
- Fin Q4 2024 (deadline auto)

---

## 🔗 Références

### Documentation

- [Authelia Notifiers](https://www.authelia.com/docs/configuration/notifications/introduction.html)
- [Authelia Filesystem](https://www.authelia.com/docs/configuration/notifications/filesystem.html)
- [Brevo SMTP Docs](https://developers.brevo.com/docs/send-a-transactional-email)

### Discussions

- Discord #setup-ct870 (2025-11-16)
- Notion page CT 870 decisions

---

## ✅ Validation

**Décision prise par** : François Danaels - 2025-11-16  
**Validée avec** : Claude (analyse sécurité)  
**Implémentation** : Authelia filesystem actif depuis 2025-11-16

**Résultat** : ✅ **SUCCÈS**
- Client Manekineko accès OK en 10 min
- 2FA TOTP fonctionne correctement
- Sécurité maintenue
- Migration SMTP planifiée Décembre 2025