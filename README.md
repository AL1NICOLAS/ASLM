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

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  ASLM — Suivi dette de sécurité              [Entité ▼]  [Période ▼]            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────────┐  │
│  │   Stock DEV           │ │   Stock PROD          │ │   Variation T-1→T     │  │
│  │   Security Defects    │ │   Vulnerabilities     │ │                       │  │
│  │                       │ │                       │ │      ↓ -12%           │  │
│  │        247            │ │        189            │ │   (tendance)          │  │
│  └───────────────────────┘ └───────────────────────┘ └───────────────────────┘  │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Détail par criticité                                                           │
│  ┌─────────────────────────────────────┐ ┌─────────────────────────────────────┐│
│  │ DEV    High: 180   Critical: 67     │ │ PROD  High: 142   Critical: 47     ││
│  └─────────────────────────────────────┘ └─────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────────────────────┤
│  Top 5 codes apps les plus endettés                        [cliquer → détail]   │
│  ┌─────┬──────────────────────┬────────────┬────────────┐                       │
│  │  #  │ Code app             │ DEV        │ PROD       │                       │
│  ├─────┼──────────────────────┼────────────┼────────────┤                       │
│  │  1  │ APP-XXX-001          │    34      │    28      │  ← clic               │
│  │  2  │ APP-XXX-002          │    22      │    19      │                       │
│  │  3  │ APP-XXX-003          │    18      │    15      │                       │
│  │  4  │ APP-XXX-004          │    14      │    12      │                       │
│  │  5  │ APP-XXX-005          │    11      │     9      │                       │
│  └─────┴──────────────────────┴────────────┴────────────┘                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Évolution (3 / 6 / 12 mois)                                                    │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │     ╭──╮                    ╭──╮                                        │    │
│  │   ╭─╯  ╰─╮  ╭──╮         ╭─╯  ╰─╮   Stock DEV  ─ ─ ─                   │    │
│  │ ╭─╯      ╰─╯  ╰─╮     ╭─╯      ╰──  Stock PROD ─────                   │    │
│  │ T-3      T-2     T-1    T                                                │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Adoption — Couverture                                                          │
│  Scannés (DEV): 68% ████████████░░░░  │  Monitorés (PROD): 52% ████████░░░░░░  │
│  Écart : 48 apps non couvertes                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Principes de design** : vert = réduction, rouge = augmentation — Responsive : cartes en grille

---

### Vue détail — Par code app

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  ← Retour        Détail — APP-XXX-001                                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Fiche code app                                                                 │
│  ┌───────────────────────────────────────────────────────────────────────────┐  │
│  │  Code app: APP-XXX-001   │  Entité: XXX   │  Profil: P3  │  Pipelines: 4  │  │
│  │  Dernier scan: 08/02/2025                                                 │  │
│  └───────────────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Stocks & évolution                                                             │
│  ┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────────┐  │
│  │ Security Defects      │ │ Vulnerabilities       │ │ Variation             │  │
│  │ (DEV)                 │ │ (PROD)                │ │                       │  │
│  │        34             │ │        28             │ │      ↓ -8%            │  │
│  └───────────────────────┘ └───────────────────────┘ └───────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Par criticité              │  Par scanner                                       │
│  DEV:  High 24  Crit. 10    │  Fortify SAST   DEV: 18  PROD: 14                 │
│  PROD: High 20  Crit.  8    │  Sonatype SCA   DEV: 16  PROD: 14                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Historique (3 / 6 / 12 mois)                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │     ╭──╮                                                                │    │
│  │   ╭─╯  ╰─╮     Stock DEV  ─ ─ ─     Stock PROD ─────                    │    │
│  │ ╭─╯      ╰──╮                                                           │    │
│  │ T-3    T-2    T-1    T                                                    │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Liste des findings (optionnel)                                                 │
│  ┌────┬──────────┬──────────┬───────┬──────────────────┬─────────┐             │
│  │ ID │ Scanner  │ Domaine  │ CVSS  │ Description      │ Statut  │             │
│  ├────┼──────────┼──────────┼───────┼──────────────────┼─────────┤             │
│  │ 1  │ Fortify  │ DEV      │ 8.5   │ SQL injection    │ Ouvert  │             │
│  │ 2  │ Sonatype │ PROD     │ 7.2   │ CVE-2024-...     │ Ouvert  │             │
│  └────┴──────────┴──────────┴───────┴──────────────────┴─────────┘             │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Accès** : clic sur une ligne du Top 5 ou sélection via recherche / dropdown

---

## Prochaines étapes

### 1. Collecte des données (backend)

- Charger le référentiel Excel (codes apps, profils P3/P4, entité)
- Intégrer les APIs Fortify et Sonatype pour récupérer les findings
- Clarifier le mapping entre référentiel et résultats de scans (code app ↔ projet technique)

### 2. Modèle de données

- Structurer un schéma unifié : référentiel + findings DEV (Security Defects) + findings PROD (Vulnerabilities)
- Appliquer les filtres : CVSS > 7, profils P3/P4
- Prévoir les champs pour le calcul des KPIs (domaine, criticité, scanner, dates…)

### 3. Calcul des KPIs

- Développer les scripts Python pour calculer les KPIs définis
- Exporter les données agrégées dans un format exploitable par Power BI (CSV, Excel, base)

### 4. Power BI — couche de visualisation

Le dashboard sera probablement produit sous **Power BI** pour le partage et l'accessibilité.

- Créer un rapport Power BI connecté aux données préparées (fichiers, base ou API)
- Reproduire les maquettes (vue générale + vue détail)
- Configurer les filtres (Entité, Période), slicers et navigation

### Options d'architecture

| Option | Intérêt |
|--------|---------|
| **Python → CSV/Excel → Power BI** | Simple, peu d'infra |
| **Python → Base (SQLite/PostgreSQL) → Power BI** | Évolutif, historique |
| **Power BI DirectQuery vers API** | Si une API d'agrégation est exposée |

### Plan d'action proposé

1. Valider le format du fichier Excel et les clés de jointure
2. Documenter les APIs Fortify et Sonatype (endpoints, mapping code app)
3. Prototyper en Python : chargement Excel + appels API + calcul KPIs + export CSV/Excel
4. Créer un rapport Power BI pilote à partir des exports
5. Automatiser (pipeline planifié) selon la fréquence de rafraîchissement souhaitée

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
