# 📊 Data Warehouse - Formation et Projets

## 📁 Structure du Projet

Ce dépôt contient l'ensemble des ressources pédagogiques et pratiques pour la maîtrise des concepts de Data Warehouse, Snowflake et DBT.

### 🗂️ Organisation des Dossiers

```
Data Warehouse/
├── 📚 COURS/                    # Supports de formation théoriques
│   ├── page1_intro_datawarehouse.html
│   ├── page2_oltp_olap.html
│   ├── page3_architecture_dw.html
│   ├── page4_modelisation.html
│   ├── page5_synthese.html
│   ├── page6_cloud_techno.html
│   └── page7_pratique_modelisation.html
│
├── ❄️ SNOWFLAKE/                # Guides pratiques Snowflake
│   ├── 01-creation-compte.md
│   ├── 02-connexion.md
│   ├── 03-creation-role.md
│   ├── 04-creation-warehouse.md
│   ├── 05-creation-database-schema.md
│   ├── 06-creation-tables.md
│   ├── 07-import-donnees.md
│   ├── 08-gestion-privileges.md
│   ├── 09-monitoring.md
│   └── DATA/                    # Données d'exemple
│
├── 🔧 DBT CLOUD/                # Guides DBT Cloud
│   ├── 00-commandes-dbt.md
│   ├── 01-environnement.md
│   ├── 02-initialisation.md
│   ├── 03-premiers-modeles.md
│   ├── 04-materialisations.md
│   ├── 05-lineage.md
│   ├── 06-tests.md
│   ├── 07-incremental.md
│   └── 08-variables.md
│
└── 🚕 nyc_taxi_dbt_pipeline.md  # Projet pratique complet

```

## 🎯 Objectifs Pédagogiques

### 1. **Concepts Fondamentaux**
- ✅ Comprendre les architectures Data Warehouse
- ✅ Différencier OLTP vs OLAP
- ✅ Maîtriser la modélisation dimensionnelle (étoile, flocon)
- ✅ Appréhender les technologies cloud modernes

### 2. **Snowflake**
- ✅ Créer et configurer un environnement Snowflake
- ✅ Gérer les rôles et privilèges
- ✅ Créer des warehouses virtuels
- ✅ Importer et gérer des données volumineuses
- ✅ Monitorer les performances et coûts

### 3. **DBT (Data Build Tool)**
- ✅ Configurer DBT Cloud
- ✅ Créer des modèles de transformation
- ✅ Implémenter des tests de qualité
- ✅ Gérer les matérialisations (view, table, incremental)
- ✅ Documenter et visualiser le lineage

## 🚀 Projet Pratique : Pipeline NYC Taxi

### Description
Pipeline de données complet analysant **200+ GB** de données réelles de taxis new-yorkais avec Snowflake et DBT.

### Caractéristiques
- **Volume** : 3+ milliards de trajets depuis 2009
- **Architecture** : RAW → STAGING → INTERMEDIATE → MARTS
- **Technologies** : Snowflake + DBT Cloud
- **Analyses** : KPIs temps réel, patterns de déplacement, optimisation des coûts

### Démarrage Rapide
```bash
# 1. Installation DBT
pip install dbt-snowflake

# 2. Cloner le projet
git clone <ce-repo>

# 3. Configurer les connexions
dbt debug

# 4. Lancer les transformations
dbt run
```

## 📚 Parcours d'Apprentissage Recommandé

### 🎓 Niveau Débutant (2-3 semaines)
1. **Semaine 1** : Concepts théoriques (dossier COURS)
2. **Semaine 2** : Prise en main Snowflake (guides 01-05)
3. **Semaine 3** : Introduction DBT (guides 00-03)

### 🏆 Niveau Intermédiaire (2-3 semaines)
1. **Semaine 4** : Snowflake avancé (guides 06-09)
2. **Semaine 5** : DBT matérialisations et tests (guides 04-06)
3. **Semaine 6** : Début du projet NYC Taxi

### 🚀 Niveau Avancé (2-3 semaines)
1. **Semaine 7** : DBT incremental et variables (guides 07-08)
2. **Semaine 8** : Implémentation complète NYC Taxi
3. **Semaine 9** : Optimisations et monitoring

## 🛠️ Prérequis Techniques

### Environnement
- Python 3.11+
- Git
- Compte Snowflake (essai gratuit disponible)
- Compte DBT Cloud (version gratuite suffisante)

### Connaissances
- SQL de base
- Concepts de base de données relationnelles
- Notions de Python (optionnel mais recommandé)

## 📖 Ressources Complémentaires

### Documentation Officielle
- [Snowflake Documentation](https://docs.snowflake.com/)
- [DBT Documentation](https://docs.getdbt.com/)
- [NYC Taxi Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

### Communauté
- [DBT Community Slack](https://www.getdbt.com/community/)
- [Snowflake Community](https://community.snowflake.com/)

## 💡 Tips & Best Practices

### Snowflake
- 🔥 Utilisez des warehouses de taille appropriée (commencez petit)
- 🔥 Suspendez automatiquement les warehouses inactifs
- 🔥 Partitionnez vos données par date pour optimiser les requêtes
- 🔥 Utilisez les stages pour l'import de gros volumes

### DBT
- 🔥 Organisez vos modèles en layers (staging, intermediate, marts)
- 🔥 Documentez systématiquement vos modèles
- 🔥 Utilisez les tests pour garantir la qualité des données
- 🔥 Privilégiez les modèles incrémentaux pour les grandes tables

## 🤝 Contribution

Les contributions sont les bienvenues ! N'hésitez pas à :
- 🐛 Reporter des bugs
- 💡 Suggérer des améliorations
- 📝 Améliorer la documentation
- 🔧 Proposer de nouveaux exemples

## 📊 Statistiques du Projet

- **Guides créés** : 20+
- **Volume de données traité** : 200+ GB
- **Exemples de code** : 100+
- **Heures de formation** : 50+

## 📝 Licence

Ce projet est à des fins éducatives. Les données NYC Taxi sont publiques et disponibles sous licence open data.

---

💻 **Happy Data Engineering!** 🚀
