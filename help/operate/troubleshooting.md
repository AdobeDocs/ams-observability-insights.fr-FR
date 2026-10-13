---
title: Résolution des problèmes
description: Dépannage des requêtes SQL, des graphiques et des tableaux de bord dans Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '337'
ht-degree: 6%
---

## Résolution des problèmes {#troubleshooting}

Des messages d’erreur s’affichent sous l’éditeur lorsque vous **Valider** ou **Exécuter** une requête, ainsi que sur le widget lui-même dans un tableau de bord. Recherchez le message dans la première colonne.

| Message/symptôme | Cause | Correctif |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Unknown entity 'X'` | Nom de faute de frappe ou de non-catalogue | Cochez [Dictionnaire de données](#data-dictionary) / le panneau du guide de requête . |
| `Attribute 'x' is not available on Entity` | Attribut qui ne figure pas sur cette entité (par exemple, `duration` sur `CpuUtilization`) | Attributs de trace par rapport aux attributs de mesure — voir [Dictionnaire de données](#data-dictionary). |
| `Raw attributes cannot be mixed with aggregates; use FACET to group results` | `SELECT service, count(*)` | `SELECT count(*) … FACET service`. |
| `FACET and TIMESERIES require an aggregate query` | Requête d’événement avec FACET/TIMESERIES | Utilisez un agrégat. |
| `FACET supports at most five attributes` | > 5 facettes | Réduire. |
| `average requires a numeric attribute` | e.g. `average(service)` | Utilisez `uniqueCount`/`latest` ou un attribut numérique. |
| `rate requires TIMESERIES` / `rate is only available for counter measures` | `rate()` sans TIMESERIES ou sur une jauge | Ajoutez des `TIMESERIES AUTO` ; utilisez une entité de compteur de ✔ ([entités de mesure traitées](#53-curated-metric-entities)) ; pour les jauges, utilisez `derivative()`. |
| `rate cannot be combined with other aggregates` | Plusieurs éléments avec `rate()` | Un `rate()` par widget ; à combiner avec une formule. |
| `percentOfTotal cannot be combined with TIMESERIES` | — | Supprimez des SÉRIES CHRONOLOGIQUES. |
| `apdex requires an entity with a status attribute` | `apdex` sur une entité de mesure | Utilisez `Transaction`/`Span`/`AppRequest`. |
| `ORDER BY 'x' must reference a selected value or alias` | Tri par un élément non sélectionné | Alias l’agrégat et classement par alias. |
| `IN requires between 1 and 100 values` | — | Fractionner ou utiliser `LIKE`. |
| `Use IS NULL or IS NOT NULL for null comparisons` | `attr = NULL` | Utilisation `IS NULL`. |
| `Unexpected character '{'` | `{{variable}}` dans l’aperçu du widget Ajouter ou sur Enregistrer | Voir [Tutoriel 3, étape 8.7](#87-make-dashboards-reusable-with-variables) limitation. |
| `Unexpected character ';'` / `'"'` | Points-virgules ou guillemets doubles | Supprimez `;` ; utilisez des guillemets simples. |
| `Expected a positive duration such as 30 minutes` | `TIMESERIES 5m` | Écrivez `TIMESERIES 5 minutes`. |
| `Time range cannot exceed 31 days` | Sélecteur > 31 j | Étroite la fenêtre. |
| `Select a customer before running a SQL query` | Aucun client/client de démonstration | Sélectionnez un client dans le sélecteur de client. |
| `All queries in a formula must be the same kind…` | Combinaison du total et des SÉRIES TEMPORELLES, ou une FACETTE | Rendre toutes les sous-requêtes simples ou `TIMESERIES AUTO` ; supprimer la FACETTE. |
| `Query 'x' must select exactly one metric…` | La sous-requête comporte deux agrégats | Un agrégat par sous-requête. |
| `Unknown query name 'x' - must be one of: …` | La formule référence un nom manquant. | Faire correspondre exactement les noms (respect de la casse). |
| **Échec de l’enregistrement** | Une requête de widget est non valide côté serveur (souvent `{{var}}`) | Ouvrez chaque widget récemment modifié et **Exécuter** ; corrigez ou supprimez le widget non valide. |
| _Sélectionner une valeur pour{{x}}_ | La variable n’a pas de sélection. | Sélectionnez une valeur dans la barre des variables. |
| _Aucune activité au cours de la dernière ..._ | Aucune ligne dans la fenêtre | Élargissez la fenêtre ; vérifiez les filtres avec une requête de découverte ([découverte des données](#101-data-discovery)). |
| Lignes brisées en points | Compartiment inférieur au rythme des données | Utilisez `TIMESERIES AUTO` (≥ 2 min) ou un graphique à barres. |
| La facette affiche un groupe de chaînes vide | Attribut manquant sur certaines lignes | Ajoutez `WHERE attr != ''`. |
| _Ce panneau SQL hérité n’est plus exécuté_ | Panneau de l’ancien créateur raw-SQL | Modifier → recréer dans SQL. |
