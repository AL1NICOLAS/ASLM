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
| **PROD** (Continuous Monitoring) |release | **Vulnerabilities** |

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

### Critères de filtrage
- **CVSS > 7** (High et Critical)
- **Codes apps avec profil de sécurité P3 ou P4** uniquement

### Suivi par entité — Dette de sécurité

#### 1. Stock de dette

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Stock total Security Defects | DEV | Somme des findings CVSS>7 sur stage_release (P3/P4) |
| Stock total Vulnerabilities | PROD | Somme des findings CVSS>7 en Continuous Monitoring (P3/P4) |
| Stock par criticité | Les deux | Breakdown High vs Critical pour priorisation |

#### 2. Évolution de la dette (tendance)

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Taux de variation du stock | Les deux | (Stock T - Stock T-1) / Stock T-1 × 100 |
| Taux de réduction | Les deux | (Stock T-1 - Stock T) / Stock T-1 × 100 |
| Taux d'accroissement | Les deux | (Stock T - Stock T-1) / Stock T-1 × 100 |

#### 3. Dette par code app

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Nombre de codes apps avec dette | Les deux | Compte distinct des codes apps avec ≥1 finding CVSS>7 |
| Codes apps les plus endettés | Les deux | Top N par stock de findings |

#### 4. Maturité et adoption

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Taux de codes apps scannés | DEV | Codes apps avec scan / Scope total (P3/P4) |
| Taux de codes apps monitorés | PROD | Codes apps monitorés / Scope total (P3/P4) |
| Dette par code app scanné | DEV | Stock / Nombre de codes apps scannés |

#### 5. Temps et vélocité

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Taux de remédiation | Les deux | Findings corrigés T / Findings ouverts T-1 |
| Dette nette | Les deux | Stock T - (Nouveaux - Corrigés) |
| Temps moyen de remédiation | Les deux | Moyenne (Date correction - Date découverte) |

### KPIs de base (couverture)

| KPI | Domaine | Intention |
|-----|---------|-----------|
| Taux de couverture des scans | DEV | % du scope avec au moins un scan |
| Taux de couverture du monitoring | PROD | % du scope monitoré en prod |
| Écart scope vs activité | Les deux | Apps non couvertes |

---

## Maquette de visualisation

### Vue générale — Tableau de bord par entité

#### En-tête
- Titre : *ASLM — Suivi dette de sécurité*
- Filtres : **Entité** (dropdown BU/direction/équipe), **Période** (T-1, T-3, plage de dates)

#### Bloc 1 — KPIs principaux (3 cartes)
| Carte | Contenu |
|-------|---------|
| Stock DEV | Nombre total Security Defects (CVSS>7, P3/P4) |
| Stock PROD | Nombre total Vulnerabilities (CVSS>7, P3/P4) |
| Variation | Évolution vs T-1 (flèche ↓ verte si baisse, ↑ rouge si hausse) |

#### Bloc 2 — Détail par criticité
- DEV : High / Critical
- PROD : High / Critical

#### Bloc 3 — Top 5 codes apps les plus endettés
- Tableau : Rang | Code app | Stock DEV | Stock PROD
- **Clic sur une ligne** → ouvre la vue détail du code app

#### Bloc 4 — Graphique d'évolution
- Courbes ou barres : Stock DEV et PROD sur 3 / 6 / 12 mois

#### Bloc 5 — Adoption — Couverture
- Taux de codes apps scannés (DEV)
- Taux de codes apps monitorés (PROD)
- Nombre d'apps non couvertes (écart)

#### Principes de design
- Couleurs : vert = réduction, rouge = augmentation
- Hiérarchie visuelle : KPIs globaux en haut, détails en dessous
- Responsive : cartes en grille qui se réorganisent selon la taille d'écran

---

### Vue détail — Par code app

#### Accès
- Clic sur une ligne du **Top 5** → ouverture de la vue détail
- Ou sélection via recherche / dropdown du code app

#### Bloc 1 — Fiche code app
- Code app, Entité, Profil (P3/P4)
- Nombre de pipelines
- Date du dernier scan

#### Bloc 2 — Stocks & évolution
- Security Defects (DEV)
- Vulnerabilities (PROD)
- Tendance (variation)

#### Bloc 3 — Par criticité
- DEV : High / Critical
- PROD : High / Critical

#### Bloc 4 — Par scanner
- Fortify SAST : DEV / PROD
- Sonatype SCA : DEV / PROD

#### Bloc 5 — Historique
- Graphique d'évolution des stocks pour ce code app (3 / 6 / 12 mois)

#### Bloc 6 — Liste des findings (optionnel)
- Tableau : ID | Scanner | Domaine | CVSS | Description | Statut
- Drill-down vers les détails des findings

#### Navigation
- Bouton **← Retour vue générale** pour revenir au tableau de bord global

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
