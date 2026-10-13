---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '491'
ht-degree: 1%
---
## Tutoriel 4 : Fonctions avancées {#tutorial-4-advanced-functions}

Ce tutoriel couvre les fonctions et les entités qui alimentent des investigations plus approfondies : la variabilité, la croissance, les recherches du « pire exemple », la part du total, la composition des transactions et les mesures arbitraires OpenTelemetry.

**Dans ce tutoriel :** [9.1 stddev](#91-measure-consistency-with-stddev) · [9.2 dérivée](#92-spot-growth-with-derivative) · [9.3 Classée en dernier](#93-find-the-worst-example-with-ranked-latest) · [9.4 pourcentOfTotal](#94-find-where-time-goes-with-percentoftotal) · [9.5 SpanBreakdown](#95-break-down-transactions-with-spanbreakdown) · [9.6 N’importe quelle mesure](#96-discover-and-query-any-metric) · [9.7 Choisissez l’entité](#97-choose-the-right-trace-entity) · [9.8 Agrégation imbriquée](#98-work-around-nested-aggregation)

### Mesurer la cohérence avec les `stddev`

Deux opérations peuvent partager une moyenne et se comporter très différemment. L’écart type indique l’ampleur de la latence :

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY stddevMs DESC LIMIT 20
```

Un écart type supérieur à la moyenne indique une opération avec des requêtes parfois très lentes. Regardez ensuite son p99.

### Croissance ponctuelle avec `derivative`

`derivative(attr)` renvoie la modification par seconde de la moyenne de l’attribut entre les intervalles. Il est utile pour repérer une jauge qui grimpe, comme l’utilisation du tas avant une fuite :

```sql
SELECT derivative(value) AS heapMBPerSec FROM JvmHeapUsed FACET host TIMESERIES 5 minutes
```

Une ligne positive soutenue signifie une croissance. Les valeurs négatives signifient que la jauge est en train de tomber (par exemple, après le nettoyage). Comme `rate`, `derivative` a besoin de `TIMESERIES` et doit être le seul agrégat.

### Trouver le pire exemple avec des `latest` classés

Le deuxième argument de `latest` et `earliest` remplace « le plus récent » par « le plus élevé par cet attribut ». `latest(traceId, duration)` renvoie l’ID de trace de la requête la plus lente, afin que vous puissiez le coller dans la recherche de trace :

```sql
SELECT count(*) AS requests, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace, latest(urlPath, duration) AS slowestPath FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 15
```

Utilisez `earliest(traceId, duration)` pour récupérer l’exemple le plus rapide à des fins de comparaison.

### Trouvez où va le temps avec `percentOfTotal`

`percentOfTotal(attr)` affiche la part de chaque groupe dans le total. Appliquée à `duration`, elle indique les opérations qui consomment le plus de temps serveur. Cela est souvent plus utile que de savoir quelles opérations sont les plus lentes :

```sql
SELECT count(*) AS requests, sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfServerTime FROM Transaction FACET name LIMIT 15
```

`percentOfTotal` ne peut pas être combiné avec `TIMESERIES`.

### Répartir les transactions avec `SpanBreakdown`

`SpanBreakdown` est préagrégé lors de l’ingestion. Il enregistre dans quelle opération enfant (`segment`) chaque transaction (`rootOperation`) passe du temps, minute par minute. Il répond « À quoi sert ce point d’entrée ? » bon marché :

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE '%GET%' FACET rootOperation, segment LIMIT 25
```

Établir une tendance d’un segment au fil du temps :

```sql
SELECT average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET segment TIMESERIES AUTO LIMIT 8
```

&#x200B;> [!NOTE]
> `avgDurationMs` n’est pas automatiquement balisé avec une unité. Définissez le widget **Unit** sur `ms`.

### Découvrir et interroger n’importe quelle mesure

SQL n&#39;a ni `keyset()` ni `SHOW EVENT TYPES`. Au lieu de cela, répertoriez chaque mesure qui génère des rapports, ainsi que son unité et sa dernière valeur :

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

Interrogez ensuite une mesure par nom. Par exemple, la mémoire JVM utilisée, par pool :

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 50
```

Pour voir quels **attributs d’étendue** sont renseignés pour chaque service, comptez les valeurs non vides :

```sql
SELECT count(*) FILTER (WHERE http.route != '') AS withRoute, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE peer.service != '') AS withPeer, count(*) FILTER (WHERE error.type != '') AS withErrorType FROM Span FACET service LIMIT 50
```

&#x200B;> [!TIP]
> Lorsqu’une entité sélectionnée existe (par exemple, `CpuUtilization`), préférez-la à `MetricGauge`. Il applique le filtre approprié et la conversion de l’unité, et peut être accéléré par cumul.

### Choisir l’entité de suivi appropriée

| Utiliser | Quand vous voulez | Exemple |
| ------------- | ------------------------------------------------------------------------------------------- | --------------------------------- |
| `Transaction` | KPI des requêtes HTTP/RPC entrantes (débit, latence, erreurs, Index). Cumul accéléré. | Signaux d’or |
| `AppRequest` | Les mêmes KPI, **y compris les consommateurs de messages** (Kafka et similaires) | Services pilotés par les événements |
| `Span` | Tout ce qui se trouve sous la requête : HTTP sortant, BD, messagerie, AEM Sling/JCR/JSP, plages internes | Analyse des dépendances et des chemins de code |

Comptez chaque type d’étendue pour voir ce qui est instrumenté :

```sql
SELECT count(*) AS spans, percentile(duration, 95) AS p95 FROM Span FACET kind
```

Appels sortants par domaine externe :

```sql
SELECT count(*) AS calls, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain LIMIT 20
```

### Contournement de l’agrégation imbriquée

NRQL vous permet d’agréger un agrégat avec une sous-requête, par exemple « la moyenne des maximums par hôte ». SQL ne le fait pas. Voici les alternatives :

| Objectif | Approche SQL |
| --------------------------------------- | ---------------------------------------------------------- |
| Valeur par groupe, puis classement des groupes | `FACET group` + `ORDER BY alias` + `LIMIT` |
| Un ratio de deux agrégats | [Widget de formule](#86-combine-queries-with-formula-widgets) |
| Valeur par hôte, puis « pire hôte » | `FACET host ORDER BY alias DESC LIMIT 1` |
| Moyenne par hôte dans le temps | `FACET host TIMESERIES AUTO` et lisez la série |

Exemple : le pire hôte par p95 :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET host ORDER BY p95 DESC LIMIT 1
```
