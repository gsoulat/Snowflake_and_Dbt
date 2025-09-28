# Pipeline NYC Taxi : Snowflake + DBT
## Guide Complet de Transformation de Données Volumineuses

---

## 📋 Vue d'Ensemble du Projet

### 🎯 Objectif
Créer un pipeline de données robuste pour analyser **200+ GB** de données réelles de taxis NYC avec Snowflake et DBT.

### 📊 Dataset : NYC Taxi Trip Data
- **Volume** : 3+ milliards de trajets depuis 2009
- **Taille** : 200+ GB compressés, 750+ GB décompressés
- **Source** : [NYC Taxi & Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- **Format** : Fichiers Parquet mensuels

### 🏗️ Architecture
```
RAW DATA → STAGING → INTERMEDIATE → MARTS
   ↓          ↓           ↓          ↓
Parquet → Nettoyage → Calculs → Analyses
```

---

## 🚀 Étapes de Mise en Œuvre

### 1. **Préparation** (30 min)
```bash
# Créer compte Snowflake (essai gratuit)
# Installer DBT
pip install dbt-snowflake

# Vérifier installation
dbt --version
```

### 2. **Configuration Snowflake** (15 min)
```sql
-- Configuration initiale
USE ROLE ACCOUNTADMIN;

CREATE WAREHOUSE NYC_TAXI_WH WITH
  WAREHOUSE_SIZE = 'LARGE'
  AUTO_SUSPEND = 60;

CREATE DATABASE NYC_TAXI_DB;
CREATE SCHEMA NYC_TAXI_DB.RAW_DATA;
CREATE SCHEMA NYC_TAXI_DB.STAGING; 
CREATE SCHEMA NYC_TAXI_DB.MARTS;
```

### 3. **Téléchargement des Données** (2-4h)
```python
# Script de téléchargement automatisé
import requests
from tqdm import tqdm

def download_nyc_taxi_data(years=[2023, 2024]):
    base_url = "https://d37ci6vzurychx.cloudfront.net/trip-data/"
    # Télécharge ~50GB pour 2 années
```

### 4. **Chargement dans Snowflake** (30 min)
```sql
-- Créer table de destination
CREATE TABLE yellow_taxi_trips_raw (
    vendorid INTEGER,
    tpep_pickup_datetime TIMESTAMP,
    tpep_dropoff_datetime TIMESTAMP,
    -- ... autres colonnes
);

-- Charger via stage
COPY INTO yellow_taxi_trips_raw 
FROM @nyc_taxi_stage
FILE_FORMAT = (TYPE = 'PARQUET');
```

---

## 🔄 Pipeline de Transformation DBT

### 📁 Structure du Projet
```
models/
├── staging/
│   └── stg_yellow_taxi_trips.sql      # Nettoyage
├── intermediate/
│   └── int_trip_metrics.sql           # Calculs
├── marts/
│   ├── core/
│   │   └── fact_trips.sql             # Table de faits
│   └── analytics/
│       ├── daily_trip_summary.sql     # Résumés quotidiens
│       ├── hourly_demand_patterns.sql # Patterns horaires
│       ├── location_analysis.sql      # Analyse géographique
│       └── revenue_analysis.sql       # Analyse financière
```

---

## 🧹 ÉTAPE 1 : Staging - Nettoyage des Données

### `staging/stg_yellow_taxi_trips.sql`

#### 🚨 Problèmes à Corriger

| **Colonne** | **Problème** | **Solution** |
|-------------|--------------|--------------|
| `passenger_count` | 0, NULL, >6 | NULL/0 → 1, >6 → 6 |
| `trip_distance` | Négatif, >100 miles | <0 → 0, >100 → NULL |
| `pickup/dropoff_datetime` | pickup > dropoff | Exclure le trajet |
| `fare_amount, tip_amount` | Valeurs négatives | <0 → 0 |
| `PULocationID, DOLocationID` | NULL | Exclure le trajet |

#### ➕ Colonnes Dérivées Ajoutées
```sql
-- Dimensions temporelles
pickup_date = DATE(tpep_pickup_datetime)
pickup_hour = EXTRACT(HOUR FROM tpep_pickup_datetime)
pickup_day_of_week = EXTRACT(DOW FROM tpep_pickup_datetime)
pickup_month = EXTRACT(MONTH FROM tpep_pickup_datetime)
pickup_year = EXTRACT(YEAR FROM tpep_pickup_datetime)

-- Métriques calculées
trip_duration_minutes = DATEDIFF(MINUTE, pickup, dropoff)
avg_speed_mph = (trip_distance / trip_duration_minutes) * 60
```

#### 🔍 Filtres de Qualité
```sql
WHERE tpep_pickup_datetime IS NOT NULL
  AND tpep_dropoff_datetime IS NOT NULL
  AND tpep_pickup_datetime < tpep_dropoff_datetime
  AND tpep_pickup_datetime >= '2009-01-01'
  AND pulocationid IS NOT NULL
  AND dolocationid IS NOT NULL
```

---

## ⚙️ ÉTAPE 2 : Intermediate - Calculs et Catégorisations

### `intermediate/int_trip_metrics.sql`

#### 📊 Catégories Métier Créées

**Distance du Trajet**
```sql
distance_category = CASE 
    WHEN trip_distance <= 1 THEN 'Court (≤1 mile)'
    WHEN trip_distance <= 3 THEN 'Moyen (1-3 miles)'
    WHEN trip_distance <= 10 THEN 'Long (3-10 miles)'
    ELSE 'Très long (>10 miles)'
END
```

**Durée du Trajet**
```sql
duration_category = CASE 
    WHEN trip_duration_minutes <= 10 THEN 'Rapide (≤10 min)'
    WHEN trip_duration_minutes <= 30 THEN 'Normal (10-30 min)'
    WHEN trip_duration_minutes <= 60 THEN 'Long (30-60 min)'
    ELSE 'Très long (>60 min)'
END
```

**Période de la Journée**
```sql
time_period = CASE 
    WHEN pickup_hour BETWEEN 6 AND 9 THEN 'Rush Matinal'
    WHEN pickup_hour BETWEEN 10 AND 15 THEN 'Journée'
    WHEN pickup_hour BETWEEN 16 AND 19 THEN 'Rush Soir'
    WHEN pickup_hour BETWEEN 20 AND 23 THEN 'Soirée'
    ELSE 'Nuit'
END
```

**Type de Jour**
```sql
day_name = CASE pickup_day_of_week
    WHEN 0 THEN 'Dimanche'
    WHEN 1 THEN 'Lundi'
    -- ... etc
END

is_weekend = CASE 
    WHEN pickup_day_of_week IN (0, 6) THEN 'Weekend'
    ELSE 'Semaine'
END
```

**Métrique Financière**
```sql
tip_rate_percent = CASE 
    WHEN fare_amount > 0 
    THEN (tip_amount / fare_amount) * 100
    ELSE 0
END
```

---

## 🎯 ÉTAPE 3 : Marts Core - Table de Faits Finale

### `marts/core/fact_trips.sql`

#### 🏗️ Structure de la Table de Faits

**🔑 Clés et Identifiants**
```sql
trip_key = generate_surrogate_key([pickup_datetime, pulocationid, dolocationid, fare_amount])
```

**📅 Dimensions Temporelles**
```sql
pickup_date, pickup_hour, pickup_day_of_week, day_name, 
is_weekend, time_period, pickup_month, pickup_year
```

**🗺️ Dimensions Géographiques**
```sql
pulocationid, dolocationid
```

**🚗 Dimensions de Voyage**
```sql
passenger_count, distance_category, duration_category, 
payment_type, vendorid
```

**📊 Métriques de Performance**
```sql
-- Distance et Temps
trip_distance, trip_duration_minutes, avg_speed_mph

-- Financier
fare_amount, tip_amount, tip_rate_percent, extra, mta_tax,
tolls_amount, improvement_surcharge, congestion_surcharge, 
airport_fee, total_amount
```

**🔧 Optimisations Techniques**
```sql
-- Index pour performance
indexes=[
  {'columns': ['pickup_date'], 'type': 'btree'},
  {'columns': ['pulocationid', 'dolocationid'], 'type': 'btree'}
]
```

---

## 📈 ÉTAPE 4 : Marts Analytics - Tables d'Analyse

### 1. `daily_trip_summary.sql` - Résumé Quotidien

#### 📊 Métriques de Volume
```sql
total_trips = COUNT(*)
unique_pickup_zones = COUNT(DISTINCT pulocationid)
unique_dropoff_zones = COUNT(DISTINCT dolocationid)
```

#### 🚀 Métriques de Performance
```sql
avg_trip_distance = AVG(trip_distance)
median_trip_distance = MEDIAN(trip_distance)
avg_trip_duration = AVG(trip_duration_minutes)
avg_speed = AVG(avg_speed_mph)
```

#### 💰 Métriques Financières
```sql
total_revenue = SUM(total_amount)
avg_trip_value = AVG(total_amount)
total_tips = SUM(tip_amount)
avg_tip_rate = AVG(tip_rate_percent)
```

#### 🕐 Répartition Temporelle
```sql
morning_rush_trips = SUM(CASE WHEN time_period = 'Rush Matinal' THEN 1 ELSE 0 END)
evening_rush_trips = SUM(CASE WHEN time_period = 'Rush Soir' THEN 1 ELSE 0 END)
night_trips = SUM(CASE WHEN time_period = 'Nuit' THEN 1 ELSE 0 END)
```

### 2. `hourly_demand_patterns.sql` - Patterns Horaires

#### 🔍 Analyses par Heure
```sql
-- Volume et revenus par heure de la journée
trips_per_hour, revenue_per_hour, avg_speed_by_hour

-- Identification des pics de demande
peak_hours, demand_intensity, surge_indicators

-- Patterns weekend vs semaine
weekend_vs_weekday_patterns
```

### 3. `location_analysis.sql` - Analyse Géographique

#### 🗺️ Métriques par Zone
```sql
-- Popularité des zones
pickup_volume_by_zone = COUNT(*) GROUP BY pulocationid
dropoff_volume_by_zone = COUNT(*) GROUP BY dolocationid

-- Rentabilité par zone
avg_fare_by_pickup_zone = AVG(fare_amount) GROUP BY pulocationid
most_profitable_routes = TOP routes by total_revenue

-- Patterns spéciaux
airport_traffic_patterns = WHERE airport_fee > 0
manhattan_vs_outer_boroughs = comparaison volumes/revenus
```

### 4. `revenue_analysis.sql` - Analyse Financière

#### 💳 Analyse des Pourboires
```sql
-- Par type de paiement
tip_analysis_by_payment_type = AVG(tip_rate_percent) GROUP BY payment_type

-- Par distance et durée
tip_correlation_with_distance = correlation(tip_rate, distance_category)
tip_patterns_by_time = AVG(tip_rate) GROUP BY time_period
```

#### 📈 Optimisation Tarifaire
```sql
-- Opportunités d'optimisation
fare_optimization_opportunities = zones sous-facturées
revenue_by_distance_category = rentabilité par type de trajet
price_elasticity_indicators = sensibilité prix/demande
```

---

## 🧪 Validation et Tests

### 🔍 Tests de Qualité des Données
```sql
-- Tests automatiques DBT
tests:
  - unique: trip_key
  - not_null: [pickup_date, total_amount]
  - accepted_range: 
      min_value: 0
      max_value: 100 (pour trip_distance)
```

### ✅ Tests Métier
```sql
-- Durées de trajet positives
assert_positive_trip_duration.sql

-- Cohérence des montants
assert_total_amount_consistency.sql

-- Zones valides uniquement
assert_valid_location_ids.sql
```

---

## 🚀 Déploiement et Exécution

### 📋 Commandes Essentielles
```bash
# Validation de la configuration
dbt debug

# Exécution du pipeline complet
dbt deps        # Installer les packages
dbt run         # Exécuter les transformations
dbt test        # Valider la qualité
dbt docs generate  # Générer la documentation

# Exécution ciblée
dbt run --select stg_yellow_taxi_trips+
dbt test --select fact_trips
```

### 🔄 Workflow de Développement
```bash
# Développement itératif
dbt run --select nouveau_modele
dbt test --select nouveau_modele

# Validation complète avant production
dbt build  # run + test en une commande
```

---

## 📊 Résultats Attendus

### 🎯 Tables Finales Créées

| **Table** | **Type** | **Volumétrie** | **Usage** |
|-----------|----------|----------------|-----------|
| `fact_trips` | Table de faits | 3+ milliards de lignes | Analyses détaillées |
| `daily_trip_summary` | Agrégat | ~5,000 lignes | Reporting quotidien |
| `hourly_demand_patterns` | Agrégat | ~200,000 lignes | Analyse patterns |
| `location_analysis` | Agrégat | ~500 lignes | Géo-analytics |
| `revenue_analysis` | Agrégat | ~10,000 lignes | Analyses financières |

### 📈 KPIs Disponibles

**Opérationnels**
- Volume de trajets par jour/heure/zone
- Vitesses moyennes par zone et période
- Patterns de demande saisonniers

**Financiers**
- Chiffre d'affaires total et par segment
- Évolution des pourboires par type de paiement
- Rentabilité par zone géographique

**Satisfaction Client**
- Durées moyennes de trajet
- Taux de pourboire comme proxy satisfaction
- Zones préférées et évitées

---

## 🔧 Optimisations et Bonnes Pratiques

### ⚡ Performance
```sql
-- Partitionnement par date
CLUSTER BY (pickup_date)

-- Matérialisation adaptée
models:
  staging: materialized='view'      # Transformations légères
  marts: materialized='table'       # Tables fréquemment requêtées
```

### 🔒 Sécurité
```yaml
# Variables d'environnement
password: "{{ env_var('DBT_SNOWFLAKE_PASSWORD') }}"

# Permissions minimales par environnement
dev: lecture/écriture schémas staging
prod: lecture seule schémas marts
```

### 📚 Documentation
```yaml
# Description de chaque modèle
models:
  - name: fact_trips
    description: "Table de faits des trajets avec métriques calculées"
    columns:
      - name: trip_key
        description: "Clé unique générée pour chaque trajet"
```

---

## 🎯 Cas d'Usage Avancés

### 🤖 Machine Learning
- **Prédiction de demande** : Forecast des trajets par zone/heure
- **Détection d'anomalies** : Trajets suspects ou erreurs de capteur
- **Optimisation d'itinéraires** : Recommandations basées sur patterns historiques

### 📊 Business Intelligence
- **Dashboards opérationnels** : Monitoring temps réel de la flotte
- **Analyses stratégiques** : Expansion géographique, optimisation tarifaire
- **Rapports réglementaires** : Conformité avec les autorités de transport

### 🔍 Analytics Avancés
- **Segmentation clients** : Profils de voyage par type d'utilisateur
- **Analyse de cohortes** : Évolution des patterns de mobilité
- **Impact événementiel** : Influence météo, événements sur la demande

---

## ⏱️ Timeline du Projet

| **Phase** | **Durée** | **Activités** |
|-----------|-----------|---------------|
| **Setup** | 1h | Comptes, installations, configuration |
| **Data Loading** | 3-4h | Téléchargement et chargement données |
| **DBT Development** | 4-6h | Développement modèles et tests |
| **Validation** | 1-2h | Tests, documentation, optimisation |
| **Déploiement** | 1h | Production, monitoring |
| **TOTAL** | **10-14h** | **Projet complet fonctionnel** |

---

## 🎉 Conclusion

Ce pipeline vous fournit :

✅ **Infrastructure robuste** avec Snowflake  
✅ **Transformations documentées** avec DBT  
✅ **Données prêtes pour l'analyse** avec 750+ GB nettoyés  
✅ **Tests automatisés** pour la qualité  
✅ **Documentation auto-générée** pour la maintenance  

**🚀 Prochaines étapes :**
1. Démarrer avec quelques mois de données pour validation
2. Étendre progressivement le dataset
3. Ajouter des analyses spécifiques métier
4. Intégrer des outils de visualisation
5. Explorer le machine learning sur les données transformées

**💡 Ce projet vous donne une base solide pour devenir expert en ingénierie de données moderne !**