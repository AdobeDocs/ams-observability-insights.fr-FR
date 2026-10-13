---
title: Plans directeurs de tableaux de bord
description: Créez des tableaux de bord Observability Insights à l’aide de plans directeurs pour la surveillance des applications et des infrastructures.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '1324'
ht-degree: 1%
---

## Plans directeurs de tableaux de bord {#dashboard-blueprints}

Chaque plan directeur est un tableau de bord complet et complexe. Il répertorie l’objectif, la disposition des onglets et des sections, ainsi que chaque widget avec son type de graphique, son unité, ses seuils, sa taille et sa requête. Les tailles sont `width×height` sur la grille de 12 colonnes. Créez chacune d’elles de haut en bas, puis faites-les glisser pour les ajuster.

| Blueprint | Audience | Tons clairs |
| ------------------------------------------------------------------------------------ | --------------------- | ---------------------------------------------------------------------------------- |
| [11.1 Golden Signals / SLO](#111-golden-signals--slo-board) | SRE, sur appel | Mosaïques de KPI avec seuils, Apdex, pourcentage d’erreur, saturation. |
| [11.2 Triage des erreurs](#112-error-triage-board) | De garde | Classes d’erreurs, zones réactives 5xx, requêtes ayant échoué avec des ID de suivi |
| [11.3 Exploration détaillée de la latence](#113-latency-deep-dive-board) | Ingénieurs de performance | Percentiles, intervalles de latence, variabilité, composition des transactions, traces les plus lentes |
| [11.4 Dépendances en aval](#114-downstream-dependencies-board) | Propriétaires des services | HTTP sortant, base de données, messagerie, RPC |
| [11.5 AEM Auteur / Publication / Dispatcher](#115-aem-author--publish--dispatcher-board) | équipes AEM | Sections par niveau, Sling/JSP/JCR, chargement du Dispatcher |
| [11.6 Capacité de l&#39;infrastructure](#116-infrastructure-capacity-board) | Plateforme/capacité | modes CPU, mémoire, disque, taux réseau, formules |
| [11.7 Intégrité JVM](#117-jvm-health-board) | Équipes Java | Formules de % du tas, tas par hôte et pool, threads, JVM CPU |
| [11,8 Mur exécutif/CNP](#118-executive--noc-wall-board) | CNO, direction | Mosaïques d’indicateurs de performance clés, formule de disponibilité, trafic par niveau, note de SLO |
| [11,9 Ratios &amp; Headroom](#119-ratios--headroom-board-formula-showcase) | Tout le monde | Présentation du widget de formule |
| [11.10 Service Explorer](#1110-service-explorer-variable-driven) | Chaque équipe | Piloté par des variables, réutilisable |

**Légende du graphique :** **N** = Nombre, **L** = Ligne, **A** = Aire, **B** = Barre, **P** = Secteurs, **T** = Tableau.



### Signaux d&#39;or / carte SLO

**Objectif :** un écran qui répond « la plateforme est-elle saine en ce moment ? » — trafic, erreurs, latence, saturation.
**Sélecteur d’heure :** dure 1 h (incident) ou 24 h (examen quotidien).

```text
┌──────────── Section: SLO KPIs ────────────────────────────────┐
│ N Requests │ N Error % │ N p95 │ N Apdex │ N Slow % │ N Hosts  │  (6 × 2w×2h)
├──────────── Section: Traffic & Errors ────────────────────────┤
│ A Throughput by service (6w) │ L Error % by service (6w)       │
├──────────── Section: Latency ─────────────────────────────────┤
│ L p50/p95/p99 (6w)           │ L Apdex by service (6w)         │
├──────────── Section: Saturation ──────────────────────────────┤
│ L CPU by host (4w) │ L Memory by host (4w) │ L Load 5m (4w)    │
└───────────────────────────────────────────────────────────────┘
```

**W1 · Demandes** — N, 2×2

```sql
SELECT count(*) AS requests FROM Transaction
```

**W2 · Erreur %** — N, unité `%`, 2×2. Seuils : `0 → Green`, `1 → Amber`, `5 → Red`.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**W3 · latence p95** — N, unité `ms`, 2×2. Seuils : `0 → Green`, `800 → Amber`, `2000 → Red`.

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**W4 · Apdex** — N, 2×2. Seuils : `0 → Red`, `0.7 → Amber`, `0.85 → Green` (la bande correspondante la plus élevée gagne, les scores faibles restent donc rouges).

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**W5 · Demandes > 2 s** — N, unité `%`, 2×2. Seuils : `0 → Green`, `2 → Amber`, `10 → Red`.

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

**W6 · Hôtes actifs** — N, 2×2

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction
```

**W7 · Débit par service** — A, 6×4

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W8 · % d&#39;erreur par service** — L, unité `%`, 6×4

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W9 · Percentiles de latence** — L, `ms` unitaire, 6×4

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**W10 · Apdex par service** — L, 6×4

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W11 · CPU par hôte** — L, 4×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO LIMIT 10
```

**W12 · Mémoire par hôte** — L, 4×4

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO LIMIT 10
```

**W13 · Charge 5m** — L, 4×4

```sql
SELECT average(value) AS load5m FROM LoadAverage5m FACET host TIMESERIES AUTO LIMIT 10
```



### Panorama de tri des erreurs

**Objectif :** de « taux d’erreur en hausse » à « ce point d’entrée, cette exception, cette trace » en trois clics.

```text
Tab "Overview"
│ N Errors │ N Error % │ N Affected traces │ N 5xx count │                 (4 × 3w×2h)
│ L Errors by service (8w) │ P Status code mix (4w)                       │
│ T Errors vs total by service (12w)                                       │
Tab "Drill-down"
│ T 5xx hot spots: service × path × code (12w)                             │
│ T Exception types (6w) │ T Downstream failures (6w)                      │
│ T Latest failing requests – event list (12w×6h)                          │
```

**Onglet Aperçu**

**W1 · Erreurs** — N, seuils `0 → Green`, `1 → Amber`, `100 → Red`

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

**W2 · Erreur %** — N, unité `%`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**W3 · Traces affectées** — N

```sql
SELECT uniqueCount(traceId) AS traces FROM Span WHERE status = 'Error'
```

**W4 · HTTP 5xx** — N, seuils `0 → Green`, `1 → Red`

```sql
SELECT count(*) AS http5xx FROM Transaction WHERE http.statusCode >= 500
```

**W5 · Erreurs par service** — L, 8×4

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error' FACET service TIMESERIES AUTO LIMIT 10
```

**W6 · Combinaison de codes d&#39;état** — P, 4×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET http.statusCode LIMIT 10
```

**S7 · Erreurs par rapport au total par service** — T, 12×4

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentage(*, WHERE status = 'Error') AS errorPct, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS http4xx FROM Transaction FACET service ORDER BY errors DESC LIMIT 30
```

**Onglet détaillé**

**W8 · 5xx points chauds** — T, 12×5

```sql
SELECT count(*) AS errors, percentile(duration, 95) AS p95, latest(traceId) AS lastTraceId FROM Transaction WHERE http.statusCode >= 500 FACET service, urlPath, http.statusCode ORDER BY errors DESC LIMIT 100
```

**W9 · Types d&#39;exceptions** — T, 6×5

```sql
SELECT count(*) AS errors, latest(statusMessage) AS lastMessage FROM Span WHERE status = 'Error' AND error.type != '' FACET service, error.type ORDER BY errors DESC LIMIT 50
```

**W10 · Défaillances en aval** — T, 6×5

```sql
SELECT count(*) AS failures, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND status = 'Error' FACET service, peer.service, externalDomain ORDER BY failures DESC LIMIT 50
```

**W11 · Dernières demandes ayant échoué** — T, 12×6

```sql
SELECT timestamp, service, host, name, urlPath, http.statusCode, duration, statusMessage, traceId FROM Transaction WHERE status = 'Error' LIMIT 200
```



### Tableau détaillé de la latence

**Objectif :** trouver _où_ le temps passe - quelles opérations, quels segments, dans quelle mesure est la cohérence.

```text
│ L p50/p95/p99 (8w)               │ N p99 (4w)                │
│ B Time share by operation (6w)   │ B Slowest ops by p95 (6w) │
│ T Latency profile per operation (12w)                         │
│ T Latency buckets per service (6w) │ B Variability (6w)       │
│ T Transaction composition – SpanBreakdown (12w)               │
│ T Slowest trace per operation (12w)                           │
```

**W1 · Percentiles** — L, `ms`, 8×4

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**W2 · p99** — N, `ms`, seuils `0 → Green`, `1500 → Amber`, `4000 → Red`, 4×4

```sql
SELECT percentile(duration, 99) AS p99 FROM Transaction
```

**W3 · Répartition du temps par opération** — B, `%`, 6×4

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Transaction FACET name LIMIT 15
```

**W4 · Opérations les plus lentes (p95)** — B, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 15
```

**W5 · Profil de latence** — T, 12×5

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99, max(duration) AS maxMs FROM Transaction FACET service, name ORDER BY p95 DESC LIMIT 100
```

**W6 · Intervalles de latence** — T, 6×4

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS fast, count(*) FILTER (WHERE duration > 100 AND duration <= 500) AS ok, count(*) FILTER (WHERE duration > 500 AND duration <= 2000) AS slow, count(*) FILTER (WHERE duration > 2000) AS verySlow FROM Transaction FACET service LIMIT 20
```

**W7 · Variabilité de latence** — B, `ms`, 6×4

```sql
SELECT stddev(duration) AS stddevMs FROM Transaction FACET service ORDER BY stddevMs DESC LIMIT 15
```

**W8 · Composition des opérations** — T, `ms` unitaire, 12×5

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET service, rootOperation, segment ORDER BY calls DESC LIMIT 100
```

**W9 · Trace la plus lente par opération** — T, 12×5

```sql
SELECT count(*) AS calls, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTraceId, latest(host, duration) AS slowestHost FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 50
```



### Panorama des dépendances en aval

**Objectif :** est-ce nous ou eux ? État de chaque base de données, API, file d&#39;attente et RPC que nous appelons.

```text
│ N Outbound calls │ N Outbound error % │ N Outbound p95 │            (3 × 4w×2h)
│ T Downstream services (12w)                                       │
│ L External domain p95 (6w) │ T External domains (6w)              │
│ B DB p95 by system/op (6w) │ T Slow DB namespaces (6w)            │
│ L Messages per destination (6w) │ T Producer vs consumer (6w)     │
│ T RPC methods (12w)                                               │
```

**W1 · Appels sortants** — N

```sql
SELECT count(*) AS calls FROM Span WHERE kind = 'Client'
```

**W2 · Erreur sortante %** — N, `%`, seuils `0 → Green`, `2 → Amber`, `10 → Red`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client'
```

**W3 · p95 sortant** — N, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client'
```

**W4 · Services en aval** — T, 12×5

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client' FACET service, peer.service ORDER BY calls DESC LIMIT 100
```

**W5 · Domaine externe p95** — L, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain TIMESERIES AUTO LIMIT 8
```

**W6 · Domaines externes** — T, 6×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain ORDER BY calls DESC LIMIT 50
```

**W7 · DB p95** — B, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE db.system != '' FACET db.system, db.operation ORDER BY p95 DESC LIMIT 20
```

**W8 · Espaces de noms de base de données lente** — T, 6×4

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 99) AS p99, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE db.system != '' FACET service, db.system, db.namespace ORDER BY p99 DESC LIMIT 50
```

**W9 · Messages par destination** — L, 6×4

```sql
SELECT count(*) AS messages FROM Span WHERE messaging.system != '' FACET messaging.destination TIMESERIES AUTO LIMIT 10
```

**W10 · Producteur et consommateur** — T, 6×4

```sql
SELECT count(*) FILTER (WHERE kind = 'Producer') AS produced, count(*) FILTER (WHERE kind = 'Consumer') AS consumed, percentile(duration, 95) AS p95 FROM Span WHERE messaging.system != '' FACET messaging.system, messaging.destination LIMIT 50
```

**W11 · Méthodes RPC** — T, 12×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE rpc.system != '' FACET service, rpc.system, rpc.method ORDER BY calls DESC LIMIT 50
```



### Panorama AEM Auteur / Publication / Dispatcher

**Objectif :** un tableau des opérations d’AEM regroupé par niveau, reflétant la mise en page New Relic AEM classique.
Utilisez des **en-têtes de section** nommés `PUBLISH`, `AUTHOR`, `DISPATCHER` et des titres de widget de fin avec `- PUBLISH` / `- AUTHOR` pour obtenir l’indice de couleur.

```text
Tab "Tiers"
├─ Section: OVERVIEW
│ A Traffic by tier (8w) │ T Content mix by tier (4w)
├─ Section: PUBLISH
│ N Requests - PUBLISH │ N Error % - PUBLISH │ N p95 - PUBLISH │ N Apdex - PUBLISH
│ L p95 by host - PUBLISH (6w) │ L CPU - PUBLISH (6w)
│ L Heap - PUBLISH (6w) │ T Slowest /bin servlets - PUBLISH (6w)
├─ Section: AUTHOR
│ N Requests - AUTHOR │ N Error % - AUTHOR │ N p95 - AUTHOR │ N Active authors' hosts - AUTHOR
│ L p95 by host - AUTHOR (6w) │ L Heap - AUTHOR (6w)
├─ Section: DISPATCHER
│ L Dispatcher req/s (8w) │ L CPU - DISPATCHER (4w)
Tab "Code hot spots"
│ T Slow Sling scripts (6w) │ T Slow JSPs (6w)
│ T Hottest JCR paths (6w)  │ B JCR operation mix (6w)
│ T Sling request URIs (6w) │ T GraphQL operations (6w)
│ B Sling servlet time share (12w)
```

**Vue d’ensemble**

**W1 · Trafic par niveau** — A, 8×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix TIMESERIES AUTO LIMIT 10
```

**W2 · Mélange de contenu par niveau** — T, 4×4

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/content/dam/%') AS assets FROM Transaction FACET hostTierPrefix LIMIT 20
```

**Section Publication**

**W3 · Demandes - PUBLICATION** — N

```sql
SELECT count(*) AS requests FROM Transaction WHERE host LIKE '%publish%'
```

**W4 · Erreur % - PUBLICATION** — N, `%`, seuils `0 → Green`, `1 → Amber`, `5 → Red`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction WHERE host LIKE '%publish%'
```

**W5 · p95 - PUBLICATION** — N, `ms`, seuils `0 → Green`, `1000 → Amber`, `3000 → Red`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%'
```

**W6 · Apdex - PUBLISH** — N, seuils `0 → Red`, `0.7 → Amber`, `0.85 → Green`

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction WHERE host LIKE '%publish%'
```

**W7 · p95 par hôte - PUBLICATION** — L, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO LIMIT 10
```

**W8 · CPU - PUBLISH** — L, `%`, 6×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO
```

**W9 · Tas - PUBLICATION** — L, `MB`, 6×4

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO
```

**W10 · Servlets /bin les plus lents - PUBLICATION** — T, 6×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE host LIKE '%publish%' AND urlPath LIKE '/bin/%' FACET urlPath ORDER BY p95 DESC LIMIT 30
```

**Section Auteur** — dupliquez W3-W9 et remplacez `'%publish%'` par `'%author%'` et le suffixe du titre par `- AUTHOR`. Widget supplémentaire :

**W11 · Hôtes auteur actifs - AUTEUR** — N

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction WHERE host LIKE '%author%'
```

**Section**

**W12 · Dispatcher req/s** — L, `req/s`, 8×4

```sql
SELECT rate(value) AS requestsPerSecond FROM DispatcherRequestRate FACET host TIMESERIES AUTO
```

**W13 · CPU - DISPATCHER** — L, `%`, 4×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization WHERE host LIKE '%dispatcher%' FACET host TIMESERIES AUTO
```

**Onglet Zone réactive du code**

**W14 · Scripts Sling lents** — T

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE sling.script != '' FACET sling.script ORDER BY totalMs DESC LIMIT 50
```

**W15 · JSP lents** — T

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95 FROM Span WHERE jsp.path != '' FACET jsp.path ORDER BY p95 DESC LIMIT 50
```

**W16 · Chemins JCR les plus chauds** — T

```sql
SELECT count(*) AS operations, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE jcr.path != '' FACET jcr.operation, jcr.path ORDER BY totalMs DESC LIMIT 50
```

**W17 · Combinaison d’opérations JCR** — B

```sql
SELECT count(*) AS operations FROM Span WHERE jcr.operation != '' FACET jcr.operation LIMIT 20
```

**W18 · URI de requête Sling** — T

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction WHERE sling.requestUri != '' FACET sling.requestUri ORDER BY requests DESC LIMIT 100
```

**W19 · Opérations GraphQL** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE graphql.operation != '' FACET graphql.operation ORDER BY calls DESC LIMIT 50
```

**W20 · Partage de temps du servlet Sling** — B, `%`, 12×4

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Span WHERE sling.servlet != '' FACET sling.servlet LIMIT 20
```



### Comité de la capacité de l&#39;infrastructure

**Objectif :** planification de la capacité et recherche de la saturation entre les hôtes.
**Sélecteur d’heure :** 7 j pour les tendances (les cumuls rendent cela rapide), 1 h pour les incidents.

```text
Tab "Compute"
│ N Hosts reporting │ N Peak CPU % │ N Peak memory % │ N Max load 5m │
│ L CPU by host (6w) │ L Memory by host (6w)                         │
│ L CPU user / system / iowait / steal (4 × 3w)                     │
│ T Host snapshot (12w)                                              │
│ T Top processes by CPU (6w) │ T Top processes by memory (6w)       │
Tab "Storage & Network"
│ T Disk used per mount (6w) │ N Lowest free root disk (6w)          │
│ L Disk read (6w) │ L Disk write (6w)                               │
│ L Network rx (6w) │ L Network tx (6w)                              │
│ L Total network I/O for one host – formula (12w)                   │
```

**W1 · Rapports des hôtes** — N

```sql
SELECT uniqueCount(host) AS hosts FROM CpuUtilization
```

**W2 · Pic CPU** — N, `%`, seuils `0 → Green`, `75 → Amber`, `90 → Red`

```sql
SELECT max(value) AS peakCpu FROM CpuUtilization
```

**W3 · Mémoire de pointe** — N, `%`, seuils `0 → Green`, `80 → Amber`, `92 → Red`

```sql
SELECT max(value) AS peakMemory FROM MemoryUtilization
```

**W4 · Charge maximale 5m** — N

```sql
SELECT max(value) AS maxLoad FROM LoadAverage5m
```

**W5 · CPU par hôte** — L, `%`, 6×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

**W6 · Mémoire par hôte** — L, `%`, 6×4

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**W7-W10 · CPU par mode** — L, `%`, 3×4 chacune

```sql
SELECT average(value) AS userPct FROM CpuUserPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS systemPct FROM CpuSystemPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS iowaitPct FROM CpuIowaitPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS stealPct FROM CpuStealPercent FACET host TIMESERIES AUTO LIMIT 10
```

**W11 · Instantané de l’hôte** — T, 12×5

```sql
SELECT average(value) AS avgCpu, percentile(value, 95) AS p95Cpu, max(value) AS peakCpu, latest(value) AS nowCpu FROM CpuUtilization FACET host ORDER BY p95Cpu DESC LIMIT 100
```

**W12 · Meilleurs processus de CPU** — T, 6×5

```sql
SELECT average(value) AS cpuPct, max(value) AS peakPct FROM ProcessCpuUtilization FACET host, process.name ORDER BY cpuPct DESC LIMIT 30
```

**W13 · Principaux processus par mémoire** — T, `MB`, 6×5

```sql
SELECT average(value) AS memMb, max(value) AS peakMb FROM ProcessMemoryUsage FACET host, process.name ORDER BY memMb DESC LIMIT 30
```

**Onglet Stockage et réseau**

**W14 · Disque utilisé par support** — T, `%`

```sql
SELECT latest(value) AS usedPct, max(value) AS peakPct FROM DiskUtilization FACET host, mountpoint ORDER BY usedPct DESC LIMIT 50
```

**W15 · Disque racine libre le plus bas** — N, unité `bytes`, seuils `0 → Red`, `5000000000 → Amber`, `20000000000 → Green`

```sql
SELECT min(value) AS freeBytes FROM DiskFreeBytes
```

**W16 / W17 · Débit du disque** — L, `bytes/s`

```sql
SELECT rate(value) AS readBytesPerSec FROM DiskReadRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host TIMESERIES AUTO
```

**W18 / W19 · Débit réseau** — L, `bytes/s`

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS txBytesPerSec FROM NetworkTransmitRate FACET host TIMESERIES AUTO
```

**W20 · E/S réseau totale pour un hôte (formule)** — Widget de formule, L, `bytes/s`, 12×4. Les formules ne peuvent pas `FACET`, et `rate()` doivent être limitées à un hôte (voir [Entités de mesure traitées](#53-curated-metric-entities)), donc épinglez l’hôte — remplacez `web-publish-01` par un nom d’hôte réel de W5.

`netRx` de requête :

```sql
SELECT rate(value) AS v FROM NetworkReceiveRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

`netTx` de requête :

```sql
SELECT rate(value) AS v FROM NetworkTransmitRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

Formule :

```text
netRx + netTx
```



### Carte d’intégrité JVM

**Objectif :** capturer la pression du tas, les fuites de thread et les pics de CPU JVM avant qu’ils ne deviennent des pannes.

```text
│ N Heap used % (formula) │ N Peak threads │ N JVM CPU % │ N Max heap MB │
│ L Heap used % over time – formula (12w)                                │
│ L Heap used by host (6w) │ T Heap limit by host (6w)                    │
│ L Threads by host (6w)   │ L JVM CPU by host (6w)                       │
│ T Memory pools (12w)                                                   │
```

**W1 · Pourcentage d&#39;utilisation du tas (formule)** — Formule, N, `%`, seuils `0 → Green`, `75 → Amber`, `90 → Red`

`used` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapUsed
```

`limit` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapLimit
```

Formule :

```text
(used / limit) * 100
```

**W2 · Threads de pointe** — N

```sql
SELECT max(value) AS peakThreads FROM JvmThreadCount
```

**W3 · JVM CPU** — N, `%`

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization
```

**W4 · Nombre maximal de tas utilisés** — N, `MB`

```sql
SELECT max(value) AS maxHeapMb FROM JvmHeapUsed
```

**W5 · Pourcentage d’utilisation du tas dans le temps (formule)** — Formule, L, `%`, 12×4

`used` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapUsed TIMESERIES AUTO
```

`limit` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapLimit TIMESERIES AUTO
```

Formule :

```text
(used / limit) * 100
```

>[!NOTE]
>
> Pour étendre une formule à un hôte, ajoutez des `WHERE host = 'my-publish-01'` aux requêtes **les deux**.

**W6 · Tas utilisé par l’hôte** — L, `MB`, 6×4

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed FACET host TIMESERIES AUTO
```

**W7 · Limite du tas par hôte** — T, `MB`, 6×4

```sql
SELECT latest(value) AS heapLimitMb FROM JvmHeapLimit FACET host LIMIT 50
```

**W8 · Threads par hôte** — L, 6×4

```sql
SELECT latest(value) AS threads FROM JvmThreadCount FACET host TIMESERIES AUTO
```

**W9 · CPU JVM par hôte** — L, `%`, 6×4

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization FACET host TIMESERIES AUTO
```

**W10 · Pools de mémoire** — T, 12×5

```sql
SELECT latest(value) AS bytesUsed, max(value) AS peakBytes FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool ORDER BY peakBytes DESC LIMIT 100
```



### 11.8 Panneau mural exécutif/CNP

**Objectif :** un écran mural lisible et aux couleurs acidulées. Principalement des widgets numériques avec des seuils, une tendance par ligne.

```text
│ N Requests │ N Availability % (formula) │ N Apdex │ N p95 │ N 5xx │ N Hosts │
│ L Throughput (12w)                                                          │
│ B Requests by service (6w) │ P Traffic by tier (6w)                         │
│ Text: "SLO: 99.9 % availability, p95 < 800 ms. Owner: platform-sre" (12w×1h) │
```

**W1 · Demandes** — N

```sql
SELECT count(*) AS requests FROM Transaction
```

**W2 · Disponibilité % (formule)** — Formule, N, `%`, seuils `0 → Red`, `99 → Amber`, `99.9 → Green`

`ok` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE status != 'Error'
```

`total` de requête :

```sql
SELECT count(*) AS v FROM Transaction
```

Formule :

```text
(ok / total) * 100
```

**W3 · Apdex** — N, seuils `0 → Red`, `0.7 → Amber`, `0.85 → Green`

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**W4 · p95** — N, `ms`, seuils `0 → Green`, `800 → Amber`, `2000 → Red`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**W5 · 5xx** — N, seuils `0 → Green`, `1 → Amber`, `50 → Red`

```sql
SELECT count(*) AS http5xx FROM Transaction WHERE http.statusCode >= 500
```

**W6 · Hôtes** — N

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction
```

**W7 · Débit** — L, 12×4

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

**W8 · Demandes par service** — B, 6×4

```sql
SELECT count(*) AS requests FROM Transaction FACET service LIMIT 10
```

**W9 · Trafic par niveau** — P, 6×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix LIMIT 10
```

**W10 · SLO note** — Widget Texte, 12×1.



### Ratios &amp; Headroom board (Vitrine de formule)

**Objectif :** indicateurs de performance clés dérivés qui nécessitent plusieurs requêtes.

**W1 · Demandes par hôte** — Formule, L, `req/host`

`requests` de requête :

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

`hosts` de requête :

```sql
SELECT uniqueCount(host) AS v FROM Transaction TIMESERIES AUTO
```

Formule :

```text
requests / hosts
```

**W2 · 5xx part de toutes les erreurs** — Formule, N, `%`

`e5xx` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE http.statusCode >= 500
```

`errors` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE http.statusCode >= 400
```

Formule :

```text
(e5xx / errors) * 100
```

**S3 · Temps dans la base de données par rapport au temps total du serveur** — Formule, L, `%`

`db` de requête :

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != '' TIMESERIES AUTO
```

`server` de requête :

```sql
SELECT sum(duration) AS v FROM Transaction TIMESERIES AUTO
```

Formule :

```text
(db / server) * 100
```

**W4 · Espace de mémoire (Mo)** — Formule, L, `MB`

`limit` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapLimit TIMESERIES AUTO
```

`used` de requête :

```sql
SELECT average(value) AS v FROM JvmHeapUsed TIMESERIES AUTO
```

Formule :

```text
limit - used
```

**W5 · Taux d’entrée/sortie réseau pour un hôte** — Formule, L (épingler l’hôte ; voir [Entités de mesure traitées](#53-curated-metric-entities))

`rx` de requête :

```sql
SELECT rate(value) AS v FROM NetworkReceiveRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

`tx` de requête :

```sql
SELECT rate(value) AS v FROM NetworkTransmitRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

Formule :

```text
tx / rx
```

**W6 · Taux de trafic de publication par rapport au trafic de création** — Formule, N

`publish` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE host LIKE '%publish%'
```

`author` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE host LIKE '%author%'
```

Formule :

```text
publish / author
```

**W7 · Erreur pondérée de budget brûlure (SLO 99,9 %)** — Formule, L

`errors` de requête :

```sql
SELECT count(*) AS v FROM Transaction WHERE status = 'Error' TIMESERIES AUTO
```

`total` de requête :

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

Formule (taux de combustion : 1,0 = consommation du budget exactement selon le calendrier) :

```text
(errors / total) / 0.001
```



### Explorateur de services (piloté par les variables)

**Objectif :** un panorama que toute équipe peut pointer vers _son service et son environnement_

>[!WARNING]
>
> Consultez la **limitation de build actuelle** dans [tutoriel 3, étape 8.7](#87-make-dashboards-reusable-with-variables) : les `{{variables}}` dans le widget SQL ne peuvent pas encore être prévisualisées ni enregistrées. Tant que cela n’est pas corrigé, remplacez chaque `{{…}}` par un littéral (par exemple, `'checkout'`) et utilisez _Dupliquer_ pour cloner la carte par service. La conception ci-dessous est prête pour le cas où la substitution serait prise en charge de bout en bout.

**Variables**

| Nom | Libellé | Type | Multi | Numérique | Définition |
| -------- | ------------------- | ------------ | ----- | ------- | ---------------------------------------------------------- |
| `svc` | Service | À partir d’une requête | ✔ |         | `SELECT count(*) FROM Transaction FACET service LIMIT 200` |
| `env` | Environnement | À partir d’une requête | ✔ |         | `SELECT count(*) FROM Span FACET environment LIMIT 50` |
| `slowMs` | Seuil lent (ms) | Zone de texte |       | ✔ | `2000` par défaut |

```text
Section: "{{svc}} in {{env}}"
│ N Requests │ N Error % │ N p95 │ N Slow requests │
│ L Throughput by service (6w) │ L p95 by service (6w)   │
│ T Operations (12w)                                     │
│ T Slow requests over {{slowMs}} ms – events (12w)       │
│ T Downstream calls (12w)                               │
```

**W1 · Demandes —{{svc}}** — N

```sql
SELECT count(*) AS requests FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W2 · Erreur %** — N, `%`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W3 · p95** — N, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W4 · Demandes lentes** — N

```sql
SELECT count(*) AS slow FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) AND duration > {{slowMs}}
```

**W5 · Débit par service** — L

```sql
SELECT count(*) AS requests FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service TIMESERIES AUTO
```

**W6 · p95 par service** — L, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service TIMESERIES AUTO
```

**W7 · Exploitation** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service, name ORDER BY calls DESC LIMIT 100
```

**W8 · Demandes lentes (événements)** — T

```sql
SELECT timestamp, service, host, name, urlPath, duration, traceId FROM Transaction WHERE service IN ({{svc}}) AND duration > {{slowMs}} ORDER BY duration DESC LIMIT 200
```

**W9 · Appels en aval** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND service IN ({{svc}}) FACET peer.service, externalDomain, db.system ORDER BY calls DESC LIMIT 50
```
