# audit-performance-clinique-sql
#  Audit de Performance — Clinique Médicale

> **Projet SQL | Data Analyst Freelance**

-----

## Le Problème Métier (Situation)

Une clinique médicale privée gérait ses finances **sur Excel**, manuellement. Résultat :

-  **3h perdues par semaine** à consolider les données de paiement
-  **Aucune visibilité** sur les impayés patients en temps réel
-  **Impossible de comparer** la rentabilité de chaque médecin
-  **Zéro indicateur** sur les mois creux ou les tendances de croissance

-----

## Ma Solution SQL (Action)

J’ai conçu une base de données relationnelle et un ensemble de requêtes SQL qui **automatisent l’audit financier complet** de la clinique.

Le code couvre 4 analyses clés et une **VIEW réutilisable** par la direction :

|#|Analyse                                               |Technique SQL utilisée                |
|-|------------------------------------------------------|--------------------------------------|
|1|Liste détaillée des impayés par consultation          |`LEFT JOIN` + `CASE WHEN`             |
|2|Tableau de bord des dettes par patient                |`GROUP BY` + `SUM` + `HAVING`         |
|3|Classement des médecins par CA et taux de recouvrement|`GROUP BY` + `ROUND` + calcul de ratio|
|4|Évolution mensuelle du chiffre d’affaires             |`strftime` + `GROUP BY` mois          |
|5|Vue SQL dashboard (réutilisable)                      |`CREATE VIEW`                         |

-----

## Résultats Concrets (Résultat)

- **Rapport d’impayés** généré en **< 1 seconde** (vs 3h sur Excel)
- Identification immédiate des patients avec solde > 0
- Classement des médecins par rentabilité réelle
- Vue SQL prête à brancher sur **Power BI / Tableau / Metabase**

-----

## Structure du Projet

```
audit-clinique-sql/
│
├── audit_clinique.sql       ← Script principal (tables + données + analyses)
└── README.md                ← Ce fichier
```

-----

## Schéma de la Base de Données

```
┌─────────────┐        ┌──────────────────┐        ┌──────────────┐
│   patients  │        │  consultations   │        │   medecins   │
│─────────────│        │──────────────────│        │──────────────│
│ patient_id  │◄───────│ patient_id  (FK) │───────►│ medecin_id   │
│ nom         │        │ medecin_id  (FK) │        │ nom          │
│ prenom      │        │ date_consultation│        │ specialite   │
│ ville       │        │ montant_du       │        └──────────────┘
└─────────────┘        │ montant_paye     │
                       └──────────────────┘
```

**Relations :**

- 1 patient peut avoir **plusieurs consultations**
- 1 médecin peut faire **plusieurs consultations**
- La table `consultations` est la table centrale (fait)
-----

## Concepts SQL Clés Utilisés

```sql
-- LEFT JOIN : récupère TOUTES les lignes de gauche,
-- même si aucune correspondance à droite
SELECT * FROM consultations c
LEFT JOIN patients p ON c.patient_id = p.patient_id;

-- GROUP BY + HAVING : agrège les données PUIS filtre sur l'agrégat
-- (≠ WHERE qui filtre avant l'agrégation)
SELECT patient_id, SUM(montant_du - montant_paye) AS solde
FROM consultations
GROUP BY patient_id
HAVING solde > 0;

-- CASE WHEN : crée une colonne conditionnelle (comme un IF/ELSE)
CASE
    WHEN montant_paye = 0            THEN 'IMPAYÉ TOTAL'
    WHEN montant_paye < montant_du   THEN 'IMPAYÉ PARTIEL'
    ELSE                                  'SOLDÉ'
END AS statut;
```

-----

## Défis Rencontrés & Apprentissages

**Défi 1 — LEFT JOIN vs INNER JOIN**

> Au départ, j’utilisais un `INNER JOIN` qui excluait les patients sans consultations. J’ai réalisé que pour un audit d’impayés, on veut TOUTES les lignes — même les cas à zéro. Le `LEFT JOIN` était la bonne approche.

**Défi 2 — HAVING vs WHERE**

> J’ai d’abord écrit `WHERE solde > 0` ce qui causait une erreur car `solde` est une colonne calculée par `SUM()`. Les colonnes agrégées ne peuvent pas être filtrées par `WHERE` — il faut utiliser `HAVING` qui s’applique après le `GROUP BY`.

**Défi 3 — Calcul du taux de recouvrement**

> Pour éviter une division entière (qui donne 0 en SQL), j’ai dû multiplier par `100.0` (flottant) plutôt que `100` (entier). Subtilité importante pour tous les calculs de ratios.

-----

## Évolutions Possibles

- [ ] Ajout d’une table `paiements` pour historiser les règlements partiels
- [ ] Analyse de la saisonnalité sur 2-3 ans
- [ ] Alertes automatiques sur les impayés > 30 jours

-----

## À Propos

**Data Analyst et étudiant en médecine spécialisé dans l’optimisation et l’automatisation des données métier.**

Je transforme vos données brutes en insights actionnables via SQL, Python et des outils de visualisation.

Disponible pour missions freelance sur **[Upwork](https://upwork.com)** | **[Malt](https://malt.fr)**

-----

*Made with SQL *
