# 📋 Decision Log - Architecture & Technical Decisions

## Overview

Ce document enregistre toutes les décisions techniques importantes prises lors de la construction de cette application FastAPI avec authentification et pipeline CI/CD.

---

## ✅ Décisions Principales

### 1. Framework : FastAPI ⚡

**Décision** : Utiliser FastAPI au lieu de Flask ou Django

**Justification** :
- ⚡ Performance supérieure (async/await natif)
- 📚 Documentation automatique (OpenAPI/Swagger)
- ✅ Validation des données intégrée (Pydantic)
- 🔒 Sécurité par défaut
- 🧪 Facile à tester
- 👥 Communauté croissante et active

**Alternatives considérées** :
- Flask : Minimaliste mais moins de features intégrées
- Django : Trop heavy pour cette use case
- Starlette : Trop bas niveau, FastAPI l'utilise déjà

---

### 2. Base de Données : PostgreSQL ✅

**Décision** : PostgreSQL au lieu de MongoDB ou MySQL

**Justification** :
- 🔒 ACID compliance garantie
- 📊 Support natif JSON
- 🎯 Optimisé pour les relationnels complexes
- 💪 Très fiable en production
- 🛠️ Support excellent pour les migrations
- 📈 Performance scalable

**Configuration** :
- Version : PostgreSQL 15-alpine (image Docker légère)
- ORM : SQLAlchemy 2.0
- Health checks : Service actif avant démarrage API

---

### 3. Authentification : Tokens + Bcrypt 🔐

**Décision** : Tokens simples + hachage Bcrypt

**Justification pour tokens simples** :
- ✨ Simple à implémenter et comprendre
- 🎯 Suffisant pour une démo/MVP
- 🔒 Tokens URL-safe aléatoires (32+ chars)
- 📝 Facile à tester et déboguer

**Justification pour Bcrypt** :
- 🛡️ Hash cryptographique robuste
- 🔀 Salting automatique
- ⏰ Adaptatif à la puissance de calcul

**Production** :
- TODO : Migrer vers JWT (JSON Web Tokens)
- TODO : Ajouter expiration de tokens
- TODO : Implémenter refresh tokens

---

### 4. Tests : Pytest + TestClient 🧪

**Décision** : Pytest avec TestClient FastAPI

**Justification** :
- ✅ Syntax simple et lisible
- 🚀 Plugins riches (coverage, mocking, etc.)
- 📝 Cohabitation facile avec FastAPI
- 🧪 TestClient permet tester sans serveur réel

**Couverture** :
- ✅ Tests unitaires pour les cas principaux
- ✅ Tests d'intégration (DB + API)
- ✅ 7+ tests couvrant les critères

**À améliorer** :
- Ajouter des tests de charge
- Ajouter des tests de sécurité (injection SQL)
- Ajouter des mocks pour les dépendances externes

---

### 5. Conteneurisation : Docker + Compose 🐳

**Décision** : Docker pour conteneurisation, Docker Compose pour orchestration locale

**Justification Docker** :
- 📦 Reproductibilité garantie
- 🚀 Déploiement rapide et prévisible
- 🔒 Isolation des processus
- 📈 Scalabilité facilitée

**Justification Docker Compose** :
- 🎯 Multi-conteneurs orchestrés localement
- 🔗 Networking automatique
- 💾 Volumes persistants
- 🏥 Health checks intégrés

**Configuration** :
- Image de base : python:3.10-slim (efficace)
- Port API : 8000 (FastAPI par défaut)
- Port DB : 5432 (PostgreSQL standard)
- Variables d'environnement : Pas de secrets en dur

---

### 6. Pipeline CI/CD : GitHub Actions 🔄

**Décision** : GitHub Actions au lieu de Jenkins/GitLab CI/CircleCI

**Justification** :
- ✨ Intégration native avec GitHub
- 🆓 Gratuit pour repos publics
- 🚀 Facile à configurer
- 📝 Syntaxe YAML simple
- 🔐 Secrets management intégré

**Architecture du pipeline** :

```
┌─────────────────┐
│  Push to main   │
└────────┬────────┘
         │
         ▼
┌──────────────────────────────────────┐
│  Job 1: Tests Unitaires              │
│  - Setup Python 3.10                 │
│  - Install dependencies              │
│  - Run pytest                         │
│  - PostgreSQL service container      │
└────────┬─────────────────────────────┘
         │ (approuver si succès)
         ▼
┌──────────────────────────────────────┐
│  Job 2: Build Docker Image           │
│  - Build Dockerfile                  │
│  - Push à ghcr.io (si main)          │
│  - Cache Docker layers               │
└────────┬─────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│  Job 3: Test Docker Compose          │
│  - docker-compose up                 │
│  - Health checks                     │
│  - Clean up                          │
└──────────────────────────────────────┘
```

**Étapes principales** :

1. **Test Job** :
   - Installe les dépendances
   - Lance PostgreSQL en service container
   - Exécute pytest
   - Échoue si tests échouent

2. **Build Job** :
   - Dépend du succès de Test
   - Build l'image Docker
   - Pousse sur GitHub Container Registry (main seulement)
   - Utilise cache Docker pour vitesse

3. **Docker Compose Test** :
   - Teste la stack complète
   - Vérifie health check API
   - Nettoie les ressources

---

### 7. Secrets & Sécurité 🔒

**Décision** : Aucun secret en dur, tout en variables d'environnement

**Justification** :
- 🔐 Secrets jamais dans le code/git
- 🔄 Configuration par environnement
- ✨ Flexible production/staging/dev

**Pipeline CI/CD** :
- Utilise `secrets.GITHUB_TOKEN` automatique
- Pas d'autres secrets requis
- Registry push optionnel (PRs n'en ont pas besoin)

**Production** :
- TODO : Utiliser Azure Key Vault / AWS Secrets Manager
- TODO : Rotation automatique de secrets
- TODO : Audit logging des accès

---

### 8. Versions & Dépendances 📦

**Python** : 3.10+
- Raison : Version LTS, async/await stable, type hints matures

**FastAPI** : 2.104.1
- Raison : Dernière version stable

**PostgreSQL** : 15-alpine
- Raison : Version récente, image légère Alpine

**SQLAlchemy** : 2.0.23
- Raison : Version 2.0+ pour syntaxe moderne

**Autres dépendances** :
```
- uvicorn : Serveur ASGI
- pydantic : Validation données
- passlib : Hachage mots de passe
- psycopg2 : Driver PostgreSQL
- pytest : Testing framework
- httpx : HTTP client tests
```

---

### 9. Structure du Projet 📁

**Décision** : Séparer app, docker, tests, docs

```
app/                 # Code métier
├── main.py         # Application FastAPI
├── requirements.txt # Dépendances
└── tests/          # Tests unitaires

docker/             # Configuration conteneurs
├── Dockerfile      # Image FastAPI
└── docker-compose.yml # Orchestration

.github/workflows/  # CI/CD
└── ci.yml         # Pipeline GitHub Actions

docs/              # Documentation
└── decision-log.md # Ce fichier
```

**Justification** :
- ✨ Clear separation of concerns
- 🔍 Facile à naviguer
- 📈 Scalable pour futurs modules

---

### 10. Health Checks 🏥

**Décision** : Health check endpoint + PostgreSQL health check

**Endpoints** :
- `GET /health` → Vérifier que l'API répond
- `GET /protected` → Vérifier authentification

**PostgreSQL** :
- Service container avec health check
- API attend que PostgreSQL soit healthy
- Évite les erreurs de connexion

---

## 🚀 Améliorations Futures

### Court terme (Phase 2)
- [ ] JWT avec expiration de tokens
- [ ] Refresh tokens
- [ ] Email verification
- [ ] Password reset flow
- [ ] CORS configuration
- [ ] Rate limiting

### Moyen terme (Phase 3)
- [ ] Role-based access control (RBAC)
- [ ] API versioning
- [ ] Database migrations (Alembic)
- [ ] Logging centralisé
- [ ] Monitoring & alerts
- [ ] Kubernetes deployment

### Long terme (Phase 4+)
- [ ] Microservices architecture
- [ ] Event-driven messaging (Kafka)
- [ ] GraphQL API
- [ ] gRPC services
- [ ] Multi-region deployment
- [ ] Advanced security (2FA, OAuth2)

---

## 📊 Comparaison Alternatives Considérées

### Framework Web

| Aspect | FastAPI | Flask | Django |
|--------|---------|-------|--------|
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Async | ✅ Native | ⚠️ Via extensions | ⚠️ Via extensions |
| Validation | ✅ Built-in | ❌ Externe | ✅ Built-in |
| Docs | ✅ Auto | ❌ Manuel | ⚠️ Limited |
| Learning curve | ⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |

### Base de Données

| Aspect | PostgreSQL | MySQL | MongoDB |
|--------|-----------|--------|---------|
| ACID | ✅ Full | ✅ InnoDB | ⚠️ Transactions |
| Scalabilité | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Relationnels | ✅ Excellent | ✅ Bon | ❌ Non |
| JSON | ✅ Built-in | ⚠️ Limited | ✅ Native |
| Maintenance | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

### CI/CD

| Aspect | GitHub Actions | GitLab CI | Jenkins |
|--------|----------------|-----------|---------|
| Setup | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Intégration | ✅ Native | ⭐⭐⭐ | ⚠️ Plugin |
| Coût | 🆓 (public) | 🆓 (public) | 🆓 Auto-hosted |
| Communauté | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Scalabilité | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 🔍 Lessons Learned

### ✅ Ce qui a bien fonctionné

1. **FastAPI + SQLAlchemy** : Combinaison puissante et productive
2. **Docker Compose** : Parfait pour développement local
3. **GitHub Actions** : Pipeline simple et efficace
4. **Type hints** : Python 3.10 avec types = code plus robuste
5. **Pytest** : Testing framework excellent

### ⚠️ Défis rencontrés & solutions

| Défi | Solution |
|-----|----------|
| PostgreSQL port conflict | Utiliser variables d'env |
| Docker layer caching | Multi-stage builds |
| Test database isolation | SQLite in-memory for tests |
| Health check timing | Service dependencies |

---

## 📚 Ressources & Références

- [FastAPI Best Practices](https://fastapi.tiangolo.com/deployment/concepts/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [OWASP Top 10 API Security](https://owasp.org/www-project-api-security/)

---

**Auteur** : Anselme Gildas Boutchouang Tchassem  
**Statut** : 🟢 Production-ready
