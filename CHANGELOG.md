# Changelog

> Historique modifications repository notion-infrastructure

Format basé sur [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)

---

## [1.0.0] - 2025-11-23

### ✨ Added

#### Structure
- 📚 README.md : Vue d'ensemble repository
- 🏗️ ARCHITECTURE.md : Architecture globale infrastructure
- 🔒 .gitignore : Protection secrets
- 🤝 CONTRIBUTING.md : Guide contribution
- 📝 CHANGELOG.md : Ce fichier

#### Services Documentation (6/19 = 32%)
- 🔴 CT 700 - Nginx Proxy Manager (CRITICAL)
- 🔴 CT 760 - Guacamole RDP Gateway (CRITICAL)
- 🔴 CT 870 - FileBrowser + Authelia SSO (CRITICAL)
- 🔴 VM 201 - Mission Alexandra (CRITICAL)
- 🔴 VM 202 - Mission Francia (CRITICAL)
- 🔴 VM 820 - Rocket.Chat Pro-Assistante (CRITICAL)

#### Templates
- 📋 nouveau-service.md : Template standardisé documentation service
- 🚨 incident-report.md : Template post-mortem incidents
- 🎯 adr-template.md : Template Architecture Decision Records

#### Decisions (ADRs)
- ADR-001 : FileBrowser vs Nextcloud (2025-11-13)
- ADR-002 : Authelia TOTP Filesystem temporaire (2025-11-16)

#### Scripts
- 📝 scripts/README.md : Structure scripts maintenance
- Structure backup/ (TODO)
- Structure monitoring/ (TODO)
- Structure deployment/ (TODO)

### 📊 Stats

```yaml
Files: 17
Lines: ~15000
Documented Services: 6/19 (32%)
Templates: 3
ADRs: 2
Commits: 8
Time: ~2h
```

### 🎯 Objectifs Atteints

- ✅ Repository créé et structuré
- ✅ 6 services CRITICAL documentés
- ✅ Templates standardisés
- ✅ ADRs décisions importantes
- ✅ Guide contribution
- ✅ Sécurité (gitignore)

### 📋 TODO

#### Court terme (cette semaine)
- [ ] Documentation 13 services restants
- [ ] Scripts backup opérationnels
- [ ] Scripts monitoring opérationnels
- [ ] Tests restore backup

#### Moyen terme (ce mois)
- [ ] CI/CD GitHub Actions
- [ ] Automated testing scripts
- [ ] Integration Notion <-> GitHub
- [ ] Monitoring exécution scripts

#### Long terme (2025)
- [ ] 100% services documentés
- [ ] Infrastructure as Code (Terraform)
- [ ] Automated deployments
- [ ] Self-maintaining documentation

---

## [Unreleased]

### Planned

- 📚 Documentation services PRODUCTION (CT 800, 810, 860/861)
- 📚 Documentation services INFRASTRUCTURE (CT 750, 850, VM 600)
- 📚 Documentation services AUTRES (CT 810, VM 100, 500)
- 🛠️ Scripts backup complets
- 🛠️ Scripts monitoring complets
- 🛠️ Script create-client.sh
- 🐍 Script tracking-hours.py
- 🤖 GitHub Actions CI/CD
- 📊 Dashboard monitoring scripts

---

**Maintenu avec ❤️ depuis Seclin, France**  
**Contact** : François Danaels (fdonthewave)  
**Version** : 1.0 - 23 Nov 2025