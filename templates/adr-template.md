# ADR-[XXX] : [Titre Décision]

> **Date** : YYYY-MM-DD  
> **Statut** : 🟡 Proposé / ✅ Accepté / 🚫 Rejeté / 📋 Superseded  
> **Auteur** : [Nom]  
> **Reviewers** : [Nom(s)]

---

## 🎯 Contexte & Problème

[Décrire la situation actuelle et le problème à résoudre]

**Questions clés** :
- [Question 1 ?]
- [Question 2 ?]
- [Question 3 ?]

**Contraintes** :
- [Contrainte technique 1]
- [Contrainte business 2]
- [Contrainte ressources 3]

---

## 📊 Facteurs de Décision

- **Complexité** : Simple vs Complexe
- **Coût** : RAM, CPU, Licensing, Maintenance
- **Temps déploiement** : Setup + Config + Tests
- **Scalabilité** : 5 clients vs 50 clients
- **Maintenance** : Effort ongoing
- **Communauté** : Support, doc, maturité
- **Sécurité** : Risques, compliance

---

## 📋 Options Considérées

### Option A : [Nom Solution A]

**Description** : [Description courte]

**Avantages ✅** :
- [Avantage 1]
- [Avantage 2]
- [Avantage 3]

**Inconvénients ❌** :
- [Inconvénient 1]
- [Inconvénient 2]

**Estimation** :
```yaml
Complexity: Low / Medium / High
Time: [X]h setup
RAM: [X]GB
Cost: [X]€/mois
Maintenance: Low / Medium / High
```

---

### Option B : [Nom Solution B]

**Description** : [Description courte]

**Avantages ✅** :
- [Avantage 1]
- [Avantage 2]

**Inconvénients ❌** :
- [Inconvénient 1]
- [Inconvénient 2]

**Estimation** :
```yaml
Complexity: Low / Medium / High
Time: [X]h setup
RAM: [X]GB
Cost: [X]€/mois
Maintenance: Low / Medium / High
```

---

### Option C : [Nom Solution C]

[Même structure que A et B]

---

## 🎯 Décision

### ✅ Solution Choisie : **[Option X]**

**Raison principale** : [Pourquoi X > Y/Z en 1-2 phrases]

**Justification détaillée** :

1. **[Facteur 1]** : [Explication]
2. **[Facteur 2]** : [Explication]
3. **[Facteur 3]** : [Explication]

**Quote** : *"[Citation clé ou principe appliqué]"*

---

## 📋 Trade-offs Acceptés

### Ce qu'on gagne ✅

- [Bénéfice 1]
- [Bénéfice 2]
- [Bénéfice 3]

### Ce qu'on perd ⚠️

- [Limitation 1] → **Mitigation** : [Comment on compense]
- [Limitation 2] → **Mitigation** : [Comment on compense]
- [Limitation 3] → **Acceptable car** : [Raison]

---

## 📊 Impact

### Business

```yaml
Clients:
  - Impact: [Description]
  - Timeline: [Quand visible]

Agents:
  - Workflow change: [Oui/Non]
  - Training needed: [Oui/Non]

Revenue:
  - Cost: [X]€ one-time + [X]€/mois
  - ROI: [X] mois
```

### Technique

```yaml
Services affectés:
  - [Service 1]: [Impact]
  - [Service 2]: [Impact]

Ressources:
  - RAM: +[X]GB
  - CPU: +[X] vCores
  - Disk: +[X]GB

Migration:
  - Downtime: [X] min
  - Rollback: [Possible/Impossible]
  - Testing: [X]h
```

---

## 🛣️ Plan d'Exécution

### Phase 1 : Préparation

- [ ] [Task 1] - [Responsable] - [Deadline]
- [ ] [Task 2] - [Responsable] - [Deadline]

### Phase 2 : Exécution

- [ ] [Task 1] - [Responsable] - [Deadline]
- [ ] [Task 2] - [Responsable] - [Deadline]

### Phase 3 : Validation

- [ ] Tests fonctionnels OK
- [ ] Performance acceptable
- [ ] Monitoring configuré
- [ ] Documentation à jour

---

## 🔄 Conditions de Revue

**Cette décision sera réévaluée si** :

- [Condition 1, ex: Scaling > 50 clients]
- [Condition 2, ex: Nouvelle techno mature]
- [Condition 3, ex: Budget > 1000€/mois]

**Review date** : [YYYY-MM-DD] (6 mois / 1 an)

---

## 🔗 Références

### Documentation

- [Lien 1 : Source officielle]
- [Lien 2 : Article technique]
- [Lien 3 : Benchmark]

### Discussions

- [Lien chat Discord/Slack]
- [Lien issue GitHub]
- [Lien page Notion]

### Standards Appliqués

- [Standard 1, ex: KISS principle]
- [Standard 2, ex: 12-factor app]

---

## ✅ Validation

**Décision prise par** : [Nom] - [Date]  
**Validée par** : [Nom(s)] - [Date]  
**Implémentation trackée** : [Lien GitHub Issue/Notion]

---

**Template ADR v1.0 - 23 Nov 2025**  
**Inspiré de Michael Nygard's ADR format**