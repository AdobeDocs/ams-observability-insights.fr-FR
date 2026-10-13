---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '344'
ht-degree: 0%
---
## Tutoriel 2 : Contrôler vos données {#tutorial-2-control-your-data}

Dans ce tutoriel, vous découvrirez les fonctions et les clauses qui transforment les décomptes bruts en réponses : décomptes distincts, dernières valeurs, pourcentages, centiles, caractères génériques, tri et regroupement multidimensionnel.

**Dans ce tutoriel :** [7.1 uniqueCount](#71-count-distinct-values-with-uniquecount) · [7.2 au plus tard/au plus tôt](#72-find-the-latest-and-earliest-values) · [7.3 pourcentage](#73-calculate-percentages) · [7.4 percentile](#74-understand-latency-with-percentile) · [7.5 Libellés](#75-make-results-readable-with-labels) · [7.6 Caractères génériques et listes](#76-match-patterns-and-lists) · [7.7 Tri](#77-sort-and-rank-with-order-by) · [7.8 Facettes multiples](#78-group-by-several-attributes) · [7.9 Fenêtres passées](#79-look-at-a-past-window)

### Compter les valeurs distinctes avec `uniqueCount`

Combien d’hôtes et de traces distinctes chaque service possède-t-il ?

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

`uniqueCount` est exact, pas une estimation.

Pour **répertorier** les valeurs distinctes elles-mêmes (ce que NRQL appelle `uniques()`), facettez sur l’attribut :

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 100
```

### Rechercher les valeurs les plus récentes et les plus anciennes

`latest(attr)` renvoie la valeur la plus récente et `earliest(attr)` la plus ancienne. Elles sont utiles pour les tableaux « état actuel » :

```sql
SELECT latest(value) AS currentCpu FROM CpuUtilization FACET host
```

```sql
SELECT earliest(timestamp) AS firstSeen, latest(timestamp) AS lastSeen FROM Transaction FACET service
```

### Calculer les pourcentages

`percentage(*, WHERE condition)` renvoie le nombre de lignes correspondant à une condition. Taux d&#39;erreur :

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction
```

Tendance par service :

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

Et suivez le partage des requêtes lentes :

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction FACET service
```

### Comprendre la latence avec `percentile`

Les moyennes masquent les valeurs aberrantes. Les centiles montrent ce que la plupart des utilisateurs expérimentent (p50) et ce que les malchanceux expérimentent (p95, p99) :

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

> [!TIP]
> p50, p95 et p99 de `Transaction.duration` sont servis à partir de rollups sur de longues fenêtres, de sorte qu&#39;ils restent rapides pour des vues hebdomadaires.

### Rendre les résultats lisibles avec des libellés

Utilisez des `AS` avec des guillemets simples pour les libellés contenant des espaces :

```sql
SELECT count(*) AS 'Requests', percentile(duration, 95) AS 'p95 (ms)' FROM Transaction FACET service
```

> [!NOTE]
> Les alias s’appliquent uniquement aux éléments `SELECT`. Les colonnes à facettes sont toujours étiquetées avec le nom d’attribut.

### Modèles et listes de correspondance

`LIKE` utilise `%` pour un nombre indéfini de caractères et `_` pour un seul :

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%.html' FACET urlPath LIMIT 20
```

Exclure les ressources statiques avec `NOT LIKE` :

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath NOT LIKE '%.js' AND urlPath NOT LIKE '%.css' AND urlPath NOT LIKE '%.png' FACET urlPath LIMIT 20
```

Correspondance avec une liste comportant `IN` et `NOT IN` (jusqu’à 100 valeurs) :

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.method IN ('POST', 'PUT', 'DELETE') FACET service
```

> [!NOTE]
> SQL n&#39;a pas d&#39;expressions régulières (`RLIKE`). Combinez des modèles de `LIKE` avec des `OR` ou utilisez des `IN`.

### Trier et classer par `ORDER BY`

Par défaut, les résultats à facettes sont triés selon le premier agrégat. Utilisez `ORDER BY` pour établir un classement selon autre chose, par exemple les opérations les plus lentes par p95 :

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

Ou les services les moins utilisés en premier :

```sql
SELECT count(*) AS requests FROM Transaction FACET service ORDER BY requests ASC LIMIT 10
```

> [!TIP]
> Il n&#39;y a pas de `HAVING`. Pour rechercher uniquement les opérations « occupées », classez par `requests` et conservez le N premiers ou ajoutez une `WHERE` qui supprime le bruit (par exemple `WHERE http.method != 'OPTIONS'`).

### Regrouper par plusieurs attributs

`FACET` accepte jusqu’à cinq attributs. Chaque combinaison devient une ligne ou une série :

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET service, http.method, http.statusCode LIMIT 30
```

### Regarde une fenêtre passée

Il n&#39;y a pas de `COMPARE WITH`. Dans les tableaux de bord et dans Explorer, modifiez le **sélecteur d’heure**. Lorsque vous appelez l’API sans plage explicite, `SINCE … UNTIL …` vous donne une fenêtre passée :

```sql
SELECT count(*) AS lastWeekSameHour FROM Transaction SINCE 169 hours ago UNTIL 168 hours ago
```
