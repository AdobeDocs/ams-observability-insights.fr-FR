---
title: Demander le guide pas à pas
description: Utilisez les recettes de requête SQL pour étudier la télémétrie des applications et des infrastructures dans Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '580'
ht-degree: 0%
---

## Demander le guide pas à pas {#query-cookbook}

Le guide pas à pas est une bibliothèque de requêtes prêtes à l’emploi, regroupées par sujet. Chaque requête est précédée du type de graphique qui lui convient le mieux, en **gras**. Les clauses de temps sont ignorées, car le sélecteur de temps contrôle la fenêtre.

**Dans ce document :** [10.1 Découverte de données](#101-data-discovery) · [10.2 Débit](#102-throughput) · [10.3 Latence](#103-latency) · [10.4 Erreurs](#104-errors) · [10.5 Apdex](#105-apdex--user-experience) · [10.6 Services et hôtes](#106-services-hosts--instances) · [10.7 Dépendances](#107-downstream-dependencies) · [10.8 AEM](#108-aem-adobe-experience-manager) · [10.9 Infrastructure](#109-infrastructure) · [10.10 JVM](#1010-jvm--runtime)

>[!TIP]
>
> Remplacez les exemples de valeurs tels que `'checkout'` ou `'web-publish-01'` par les valeurs réelles de votre client. Les requêtes de découverte de données ci-dessous les répertorient.

### Découverte des données

**Tableau** — Quels services signalent le trafic :

```sql
SELECT count(*) AS requests, uniqueCount(host) AS hosts FROM Transaction FACET service LIMIT 200
```

**Tableau** — quels sont les hôtes/niveaux :

```sql
SELECT count(*) AS requests FROM Transaction FACET hostTierPrefix, host LIMIT 200
```

**Tableau** — Quels environnements et quelles régions existent :

```sql
SELECT count(*) AS spans FROM Span FACET environment, region LIMIT 50
```

**Tableau** — Chaque rapport de mesure de jauge, avec sa dernière valeur :

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

**Tableau** — chaque mesure de somme/compteur :

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

**Tableau** — Attributs d’étendue renseignés (exemple AEM) :

```sql
SELECT count(*) FILTER (WHERE sling.script != '') AS slingScripts, count(*) FILTER (WHERE jcr.operation != '') AS jcrOps, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE graphql.operation != '') AS graphql FROM Span FACET service LIMIT 50
```

### Débit

**Nombre** — nombre total de demandes :

```sql
SELECT count(*) AS requests FROM Transaction
```

**Line** — Tendance du débit :

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

**Zone** — Débit par service, top 8 :

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**Bar** — Débit par méthode HTTP :

```sql
SELECT count(*) AS requests FROM Transaction FACET http.method LIMIT 10
```

**Tableau** — Chemins d’URL les plus fréquentés :

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET urlPath ORDER BY requests DESC LIMIT 50
```

**Nombre** — traces distinctes :

```sql
SELECT uniqueCount(traceId) AS traces FROM Span
```

### Latence

**Nombre** — temps de réponse p95 (unité auto = ms) :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**Ligne** — p50 / p95 / p99 sur un graphique :

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**Ligne** — p95 par service, top 10 les plus occupés :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**Bar** — opérations les plus lentes par p95 :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

**Tableau** — profil de latence par opération :

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99, max(duration) AS maxMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY p95 DESC LIMIT 50
```

**Bar** — part du temps total passé par opération (où va le budget temps) :

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Transaction FACET name LIMIT 20
```

**Table** — ID de suivi de la requête la plus lente unique par opération :

```sql
SELECT count(*) AS calls, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTraceId FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 50
```

**Nombre** — part des demandes inférieures à 2 s :

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowShare FROM Transaction
```

**Tableau** — intervalles de latence via des nombres filtrés (un substitut d’histogramme) :

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS under100ms, count(*) FILTER (WHERE duration > 100 AND duration <= 500) AS to500ms, count(*) FILTER (WHERE duration > 500 AND duration <= 2000) AS to2s, count(*) FILTER (WHERE duration > 2000) AS over2s FROM Transaction FACET service LIMIT 20
```

### Erreurs

**Nombre** — taux d&#39;erreur % :

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**Line** — taux d&#39;erreur % par service :

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**Ligne** — Tendance du nombre d&#39;erreurs :

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error' TIMESERIES AUTO
```

**circulaire** — Mélange de classes d’état HTTP :

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode IS NOT NULL FACET http.statusCode LIMIT 15
```

**Tableau** — Points chauds 5xx :

```sql
SELECT count(*) AS errors5xx FROM Transaction WHERE http.statusCode >= 500 FACET service, urlPath, http.statusCode LIMIT 50
```

**Table** — erreurs par type d&#39;exception :

```sql
SELECT count(*) AS errors, uniqueCount(traceId) AS traces, latest(statusMessage) AS lastMessage FROM Span WHERE status = 'Error' FACET service, error.type LIMIT 50
```

**Tableau** — erreurs par rapport au total côte à côte :

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service ORDER BY errors DESC LIMIT 20
```

**Tableau** — Dernières demandes ayant échoué (liste des événements) :

```sql
SELECT timestamp, service, name, urlPath, http.statusCode, duration, traceId FROM Transaction WHERE status = 'Error' LIMIT 100
```

### Apdex et expérience utilisateur

**Nombre** — Apdex (T = 500 ms) :

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**Line** — Apdex par service :

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**Bar** — Chemin Apdex par URL le plus mauvais (cible plus stricte de 300 ms) :

```sql
SELECT apdex(duration, 300) AS apdex, count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%' FACET urlPath ORDER BY apdex ASC LIMIT 20
```

### Services, hôtes et instances

**Tableau** — Fiche d&#39;évaluation du service :

```sql
SELECT count(*) AS requests, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95, apdex(duration, 500) AS apdex, uniqueCount(host) AS hosts FROM Transaction FACET serviceKey ORDER BY requests DESC LIMIT 50
```

**Table** — par instance (AppRequest inclut les consommateurs) :

```sql
SELECT count(*) AS requests, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM AppRequest FACET appInstance ORDER BY requests DESC LIMIT 50
```

**Line** — Débit par hôte :

```sql
SELECT count(*) AS requests FROM Transaction FACET host TIMESERIES AUTO LIMIT 10
```

**Bar** — vérification de la répartition de la charge (partage des requêtes par hôte) :

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 30
```

### Dépendances en aval

**Tableau** — services en aval appelés :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client' AND peer.service != '' FACET service, peer.service LIMIT 50
```

**Table** — domaines externes appelés :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain ORDER BY calls DESC LIMIT 50
```

**Bar** — latence de la base de données par système et opération :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95 FROM Span WHERE db.system != '' FACET db.system, db.operation LIMIT 30
```

**Table** — Espaces de noms de base de données lents :

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 99) AS p99 FROM Span WHERE db.system != '' FACET db.system, db.namespace ORDER BY p99 DESC LIMIT 30
```

**Line** — Débit de messagerie par destination :

```sql
SELECT count(*) AS messages FROM Span WHERE messaging.system != '' FACET messaging.destination TIMESERIES AUTO LIMIT 10
```

**Tableau** — Producteur/Consommateur par thème :

```sql
SELECT count(*) FILTER (WHERE kind = 'Producer') AS produced, count(*) FILTER (WHERE kind = 'Consumer') AS consumed, percentile(duration, 95) AS p95 FROM Span WHERE messaging.system != '' FACET messaging.system, messaging.destination LIMIT 30
```

**Table** — Méthodes RPC :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE rpc.system != '' FACET rpc.system, rpc.method LIMIT 30
```

### AEM (Adobe Experience Manager)

**Zone** — trafic par niveau AEM (préfixe auteur/publication/dispatcher) :

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix TIMESERIES AUTO LIMIT 10
```

**Line** — niveau publication p95 :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO LIMIT 10
```

**Line** — niveau auteur p95 :

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%author%' FACET host TIMESERIES AUTO LIMIT 10
```

**Tableau** — requêtes de page (`/content`) par rapport aux appels de servlet (`/bin`) :

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/etc.clientlibs/%') AS clientlibs, count(*) FILTER (WHERE urlPath LIKE '/content/dam/%') AS assets FROM Transaction FACET hostTierPrefix LIMIT 20
```

**Tableau** — Servlets `/bin` les plus lents :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE urlPath LIKE '/bin/%' FACET urlPath ORDER BY p95 DESC LIMIT 30
```

**Table** — URI de requête Sling :

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction WHERE sling.requestUri != '' FACET sling.requestUri ORDER BY requests DESC LIMIT 100
```

**Table** — Scripts Sling lents :

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE sling.script != '' FACET sling.script ORDER BY totalMs DESC LIMIT 50
```

**Bar** — Servlets Sling par partage temporel :

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Span WHERE sling.servlet != '' FACET sling.servlet LIMIT 20
```

**Tableau** — JSP lents :

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95 FROM Span WHERE jsp.path != '' FACET jsp.path ORDER BY p95 DESC LIMIT 50
```

**Bar** — Combinaison d’opérations JCR :

```sql
SELECT count(*) AS operations FROM Span WHERE jcr.operation != '' FACET jcr.operation LIMIT 20
```

**Tableau** — Chemins JCR les plus chauds :

```sql
SELECT count(*) AS operations, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE jcr.path != '' FACET jcr.operation, jcr.path ORDER BY totalMs DESC LIMIT 50
```

**Tableau** — Opérations GraphQL (découplées) :

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE graphql.operation != '' FACET graphql.operation ORDER BY calls DESC LIMIT 50
```

**Line** — Taux de requêtes du Dispatcher par hôte :

```sql
SELECT rate(value) AS requestsPerSecond FROM DispatcherRequestRate FACET host TIMESERIES AUTO
```

**Tableau** — de quoi est faite une transaction (répartition des segments ; définir des `ms` unitaires) :

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE 'GET /content%' FACET rootOperation, segment ORDER BY calls DESC LIMIT 50
```

### Infrastructure

**Line** — CPU par hôte :

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

**Line** — Temps utilisateur CPU :

```sql
SELECT average(value) AS userPct FROM CpuUserPercent TIMESERIES AUTO
```

>[!NOTE]
>
> SQL lit une entité par requête, donc user/system/iowait/steal a besoin de quatre widgets côte à côte (voir [blueprint 11.6](#116-infrastructure-capacity-board) W7-W10). Une formule les réduirait en un seul nombre, et non en quatre séries.

**Bar** — CPU p95 par hôte :

```sql
SELECT percentile(value, 95) AS cpuP95 FROM CpuUtilization FACET host LIMIT 30
```

**Tableau** — instantané de l&#39;intégrité de l&#39;hôte :

```sql
SELECT average(value) AS avgCpu, max(value) AS peakCpu, latest(value) AS nowCpu FROM CpuUtilization FACET host ORDER BY peakCpu DESC LIMIT 50
```

**Ligne** — Mémoire utilisée % :

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**Ligne** — taux de croissance de la mémoire (recherche de fuites) :

```sql
SELECT derivative(value) AS memoryTrend FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**Table** — disque utilisé par montage :

```sql
SELECT latest(value) AS usedPct FROM DiskUtilization FACET host, mountpoint ORDER BY usedPct DESC LIMIT 50
```

**Nombre** — disque racine libre le plus bas entre les hôtes (valeur en octets ; définissez l&#39;unité `bytes`) :

```sql
SELECT min(value) AS freeBytes FROM DiskFreeBytes
```

**Ligne** — Débit de lecture et d&#39;écriture du disque (un chacun) :

```sql
SELECT rate(value) AS readBytesPerSec FROM DiskReadRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host TIMESERIES AUTO
```

**Line** — réception/transmission réseau :

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS txBytesPerSec FROM NetworkTransmitRate FACET host TIMESERIES AUTO
```

**Line** — erreurs réseau :

```sql
SELECT rate(value) AS rxErrors FROM NetworkReceiveErrorRate FACET host TIMESERIES AUTO
```

**Ligne** — charge moyenne :

```sql
SELECT average(value) AS load5m FROM LoadAverage5m FACET host TIMESERIES AUTO
```

**Tableau** — Principaux processus par CPU et mémoire :

```sql
SELECT average(value) AS cpuPct, max(value) AS peakCpuPct FROM ProcessCpuUtilization FACET host, process.name ORDER BY cpuPct DESC LIMIT 30
```

```sql
SELECT average(value) AS memMb FROM ProcessMemoryUsage FACET host, process.name ORDER BY memMb DESC LIMIT 30
```

**Tableau** — toute jauge brute par nom (aucune entité traitée nécessaire) :

```sql
SELECT average(value) AS avgValue, max(value) AS maxValue FROM MetricGauge WHERE metric = 'system.cpu.utilization' FACET host, state LIMIT 50
```

### JVM et runtime

**Ligne** — tas utilisé par hôte (Mo) :

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed FACET host TIMESERIES AUTO
```

**Tableau** — tas utilisé ou limite (deux widgets ou une formule — voir [plan directeur 11.7](#117-jvm-health-board)) :

```sql
SELECT latest(value) AS heapLimitMb FROM JvmHeapLimit FACET host LIMIT 50
```

**Line** — Threads JVM :

```sql
SELECT latest(value) AS threads FROM JvmThreadCount FACET host TIMESERIES AUTO
```

**Line** — CPU du processus JVM :

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization FACET host TIMESERIES AUTO
```

**Tableau** — Groupes de mémoire JVM (mesure de somme générique) :

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 100
```
