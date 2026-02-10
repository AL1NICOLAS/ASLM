# ASLM

Projet Python ASLM — KPIs pour piloter l'adoption AppSec sur un périmètre de codes applications.

---

## Contexte et objectifs

### Périmètre
- **~500 codes applications** à couvrir
- Chaque code app contient plusieurs pipelines (projets techniques)
- **Contexte** : début de programme AppSec, équipes peu matures, tendance à éviter l'AppSec

### Objectif
Produire des KPIs pour piloter l'adoption et mesurer la couverture des scans de sécurité (SAST + SCA) sur le périmètre.

---

## Sources de données

| Source | Type | Contenu |
|--------|------|---------|
| **Fichiers Excel** | Entrée | Référentiel du scope à couvrir (clé : code app) |
| **API Fortify** | API | Rapports SAST — Security Defects et Vulnerabilities |
| **API Sonatype Lifecycle** | API | Rapports SCA — Security Defects et Vulnerabilities |

---

## Distinction DEV / PROD

Les mêmes scanners (Fortify SAST et Sonatype SCA) sont utilisés dans les deux domaines. Seule la **branche** et le **domaine** changent :

| Domaine | Branche | Dénomination des findings |
|---------|---------|---------------------------|
| **DEV** (SSDLC Build) | `stage_release` | **Security Defects** |
| **PROD** (Continuous Monitoring) | branche prod/monitoring | **Vulnerabilities** |

> La terminologie dépend du **domaine** (DEV vs PROD), pas du type de scanner.

---

## Modèle de données (schéma)

```
Référentiel Excel (scope global)
         │
         ├──→ API Fortify + Sonatype (stage_release)    → Security Defects (DEV)
         │
         └──→ API Fortify + Sonatype (Continuous Mon.)  → Vulnerabilities (PROD)
```

---

## KPIs cibles

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Taux de couverture des scans | DEV | % du scope avec au moins un scan |
| Stock de Security Defects | DEV | Charge à traiter côté build |
| Taux de couverture du monitoring | PROD | % du scope monitoré en prod |
| Stock de Vulnerabilities | PROD | Exposition en production |
| Écart scope vs activité | Les deux | Apps non couvertes |

---

## Installation

```bash
python -m venv venv
source venv/bin/activate  # Linux/macOS
# ou: venv\Scripts\activate  # Windows

pip install -r requirements.txt
```

## Utilisation

```bash
python main.py
```
