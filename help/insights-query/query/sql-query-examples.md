---
title: Exemples de requêtes SQL
description: Explorez des exemples de requête SQL pour la télémétrie Observability Insights.
source-git-commit: 4ad45ae5cbb2a7b348fdad1cd2681a72be12f5eb
workflow-type: tm+mt
source-wordcount: '311'
ht-degree: 10%
---

## Exemples de requêtes SQL

Voici un exemple plus complet. Pour cette requête :

- Vous choisissez `Transaction` comme entité.
- Vous utilisez `SELECT` pour calculer la durée du 95e centile.
- Vous regroupez les résultats par service avec `FACET`.
- Vous utilisez `TIMESERIES AUTO` pour tracer les données sur un intervalle automatiquement dimensionné.

```sql
FROM Transaction SELECT percentile(duration, 95) AS p95 FACET service TIMESERIES AUTO
```

Cela génère une ligne par service sur un graphique linéaire, avec les 10 services les plus occupés affichés par défaut.

### Requête de base des requêtes entrantes

+++Afficher la requête

Temps de réponse moyen, en millisecondes, de toutes les requêtes entrantes :

```sql
SELECT average(duration) FROM Transaction
```

+++

### Noms d’attributs contenant des points

+++Afficher la requête

Les noms en pointillés n’ont pas besoin de guillemets :

```sql
SELECT count(*) FROM Transaction WHERE http.statusCode >= 500 FACET http.route
```

+++

### Requête renvoyant plusieurs colonnes

+++Afficher la requête

Plusieurs agrégats dans une requête deviennent des colonnes dans un tableau :

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, percentile(duration, 95) AS p95, max(duration) AS maxMs FROM Transaction FACET service
```

+++

### Agrégats conditionnels en une seule passe

+++Afficher la requête

Total, erreurs et 5xx côte à côte :

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx FROM Transaction FACET service
```

+++

### Requête d’une mesure sélectionnée

+++Afficher la requête

CPU moyen par hôte au fil du temps :

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

+++

### Requête d’une mesure OpenTelemetry par nom

+++Afficher la requête

Utilisez `MetricGauge` / `MetricSum` pour les mesures sans entité curée :

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool
```

+++

<a id="what-sql-doesnt-do"></a>

## Ce que SQL ne fait pas

SQL est délibérément petit. Cela permet de garder les requêtes sécurisées, rapides et isolées par le client ou la cliente. Lorsque vous avez besoin de l’une de ces fonctionnalités, utilisez l’alternative .

| Fonction | Statut | Alternative |
| ------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `SELECT *` | Non pris en charge. | Répertoriez les attributs souhaités : `SELECT timestamp, name, duration, traceId …`. |
| Mélange d’attributs bruts et d’agrégats | Non pris en charge. | Utilisez `FACET attr` ou `latest(attr)` / `earliest(attr)`. |
| Mathématiques en `SELECT` (`a - b`, `count(*) / 60`) | Non pris en charge. | **Formule** widget (tableaux de bord). |
| Jointures et sous-requêtes | Non pris en charge. | **Formule** widget ou deux widgets côte à côte. |
| `HAVING` | Non pris en charge. | `ORDER BY alias` + `LIMIT`. |
| `COMPARE WITH` | Non pris en charge. | Modifiez le sélecteur d’heure ou placez deux widgets sur des onglets différents. |
| `FACET CASES` | Non pris en charge. | Une colonne `count(*) FILTER (WHERE …)` par cas (voir [Tutoriel 3](../../tutorials/tutorial-3.md#group-values-into-cases-with-filter)). |
| `histogram()` | Non pris en charge. | Colonnes `FILTER` regroupées (voir [tutoriel 3](../../tutorials/tutorial-3.md#build-a-latency-histogram)). |
| `uniques()` / `keyset()` | Non pris en charge. | `FACET attr LIMIT n` répertorie des valeurs distinctes. Les requêtes de découverte répertorient les mesures et les attributs renseignés. |
| `RLIKE` (regex) | Non pris en charge. | `LIKE` / `NOT LIKE` avec des caractères génériques `%` et `_`, ou `IN (…)`. |
| Commentaires (`--`, `//`, `/* */`) | Non pris en charge. | Insérez des notes dans le titre du widget ou dans un widget **Texte**. |
| Dates absolues en `SINCE` | Non pris en charge. | Utilisez la plage absolue du sélecteur d’heure. |
| Conversion de type (`numeric()`, `string()`) | Pas nécessaire | Les attributs sont saisis dans le catalogue. |
