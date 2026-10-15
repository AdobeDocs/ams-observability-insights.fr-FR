---
title: Dictionnaire de données
description: Référencez les entités et les attributs disponibles pour les requêtes SQL Observability Insights.
source-git-commit: 91a61d33010192cedce7d40a45ffe86f32e4824b
workflow-type: tm+mt
source-wordcount: '598'
ht-degree: 21%
---

## Dictionnaire de données {#data-dictionary}

Il s’agit de chaque entité et attribut que vous pouvez interroger. Le catalogue en direct, avec ses descriptions, se trouve également dans le panneau **Guide de requête** dans Explorer et dans la boîte de dialogue Ajouter un widget.

### Trace des entités

| Entité | Contient | Portée fixe | Cumuls |
| ------------- | --------------------------------------------------------- | ---------------------------- | ------------------------------------------------------- |
| `Transaction` | Requêtes de serveur entrantes | `kind = Server` | 1m / 5m / 1h ([Performance](../operate/performance.md)) |
| `AppRequest` | Requêtes **et** message client entrantes du serveur | `kind IN (Server, Consumer)` | — |
| `Span` | Chaque étendue (serveur, client, producteur, consommateur, interne) | — | — |

<details open>
<summary><strong>Attributs de suivi</strong> (identique sur les trois entités)</summary>

| Attribut | Type | Description |
| --------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `timestamp` | date et heure | Heure de début de la période |
| `traceId`, `spanId`, `parentSpanId` | chaîne | Identifiants (les étendues racine ont `parentSpanId = ''`) |
| `name` | chaîne | Nom de l’opération ou de la transaction |
| `kind` | chaîne | `Server`, `Client`, `Producer`, `Consumer`, `Internal` |
| `service` | chaîne | Nom du service émetteur |
| `serviceKey` | chaîne | Identité de service effective (retourne à `appName` lorsque `service` est vide ou un parent grossier) |
| `appName` | chaîne | Nom de l’application Observability Insights |
| `appInstance` | chaîne | Identité par instance : préfixe hôte de création/publication, puis nom de l’application, puis service |
| `hostTierPrefix` | chaîne | Nom d’hôte tronqué à son suffixe de rôle `author` / `publish` / `dispatcher` |
| `duration` | nombre (ms) | Durée de la plage en millisecondes |
| `status` | chaîne | `Ok`, `Error`, `Unset` |
| `statusMessage` | chaîne | Description du statut |
| `host` | chaîne | Nom d&#39;hôte |
| `environment` | chaîne | Environnement de déploiement |
| `region` | chaîne | Région Cloud |
| `cloud.provider` | chaîne | AWS / AZURE / GCP |
| `http.method` | chaîne | `GET`, `POST`, ... |
| `http.statusCode` | nombre | Statut de réponse HTTP |
| `http.route` | chaîne | Modèle d&#39;itinéraire normalisé |
| `url` | chaîne | URL complète |
| `urlPath` | chaîne | Partie path de l’URL (`/content/site/en.html`) |
| `externalDomain` | chaîne | Domaine d’une URL sortante (à utiliser sur les plages clientes) |
| `peer.service` | chaîne | Service en aval appelé |
| `db.system`, `db.operation`, `db.namespace` | chaîne | Technologie de base de données, fonctionnement et schéma |
| `messaging.system`, `messaging.destination` | chaîne | Kafka, RabbitMQ, ... et topic ou queue |
| `rpc.system`, `rpc.method` | chaîne | gRPC et similaires |
| `error.type` | chaîne | Type d’exception ou d’erreur |
| `sling.script`, `sling.servlet`, `sling.requestUri` | chaîne | Détails AEM Sling |
| `jsp.path` | chaîne | AEM JSP exécuté |
| `jcr.operation`, `jcr.path` | chaîne | Opération et chemin du référentiel JCR AEM |
| `graphql.operation` | chaîne | Nom de l’opération GraphQL |

</details>

### Entités de mesure génériques

| Entité | Contient |
| ------------- | -------------------------------------------------- |
| `MetricGauge` | Échantillon de jauge OpenTelemetry à chaque instant |
| `MetricSum` | Chaque échantillon de somme OpenTelemetry cumulée ou delta |

Utilisez-les pour la **découverte** ou pour les mesures qui n’ont pas d’entité traitée : `WHERE metric = 'jvm.gc.duration'`.

<details open>
<summary><strong>Attributs de mesure</strong> (partagés par toutes les entités de mesure)</summary>

| Attribut | Type | Description |
| ----------------------------------------------------------------------- | --------- | ------------------------------------------------------- |
| `timestamp` | date et heure | Heure de l’échantillon |
| `metric` | chaîne | Nom de la mesure OpenTelemetry |
| `value` | nombre | Exemple de valeur (convertie sur des entités traitées ; voir ci-dessous) |
| `unit` | chaîne | Unité métrique |
| `service`, `appName`, `host`, `environment`, `region`, `cloud.provider` | chaîne | Identité de la ressource |
| `state` | chaîne | Mode CPU ou état de mémoire |
| `cpu`, `device`, `direction`, `mountpoint` | chaîne | Par CPU, par appareil, direction d’E/S, système de fichiers |
| `pool` | chaîne | Pool de mémoire JVM ou pool de connexion |
| `topic` | chaîne | Topic Kafka |
| `process.pid`, `process.name`, `process.owner` | chaîne | Identité du processus |

</details>

### Entités de mesure traitées

Chaque entité traitée enveloppe une mesure OpenTelemetry avec le filtre approprié et la conversion d’unité, `value` est donc déjà significatif. **Compteur** les entités (✔) contiennent des totaux cumulés. Exécutez-les avec `rate(value)` et `TIMESERIES`.

| Entité | `value` signifie | Unité | Compteur | Remarques |
| ------------------------------------------------------------------------------ | ----------------------------------- | --------- | ------- | -------------------------------------------- |
| `CpuUtilization` | CPU occupé | % |         | Cumul de 1 minute pour les `average(value)` par hôte |
| `CpuUserPercent` / `CpuSystemPercent` / `CpuIowaitPercent` / `CpuStealPercent` | CPU time by mode | % |         |                                              |
| `MemoryUtilization` | Mémoire utilisée | % |         | Cumul de 1 minute pour les `average(value)` |
| `LoadAverage1m` / `LoadAverage5m` / `LoadAverage15m` | Charge moyenne | — |         |                                              |
| `LoadAverageWithFallback` | 5 m de charge, retour à 1 m | — |         |                                              |
| `DiskUtilization` | Disque utilisé par montage | % |         | Facette par `mountpoint` |
| `DiskFreeBytes` | Système de fichiers racine libre | octets |         |                                              |
| `DiskReadRate` / `DiskWriteRate` | Débit du disque | octets/s | ✔ |                                              |
| `DiskReadOpsRate` / `DiskWriteOpsRate` | IOPS du disque | ops/s | ✔ |                                              |
| `NetworkReceiveRate` / `NetworkTransmitRate` | débit eth0 | octets/s | ✔ |                                              |
| `NetworkReceiveRateAllDevices` / `NetworkTransmitRateAllDevices` | Tous les appareils | octets/s | ✔ |                                              |
| `NetworkIoRate` | Toutes directions et appareils confondus | octets/s | ✔ |                                              |
| `NetworkReceiveErrorRate` / `NetworkTransmitErrorRate` | Erreurs réseau | erreurs/s | ✔ |                                              |
| `NetworkReceivePacketRate` / `NetworkTransmitPacketRate` | Paquets | paquets/s | ✔ |                                              |
| `ProcessCpuUtilization` | CPU par processus | % |         | Facette par `process.name` |
| `ProcessMemoryUsage` | Mémoire par processus | Mo |         |                                              |
| `JvmHeapUsed` / `JvmHeapLimit` | Tas JVM | Mo |         | Signalé par pool de mémoire |
| `JvmThreadCount` | Threads JVM | — |         |                                              |
| `JvmCpuRecentUtilization` | CPU du processus JVM | % |         | cumul de 1 minute |
| `DispatcherRequestRate` | Requêtes HTTP du Dispatcher AEM | req/s | ✔ |                                              |

>[!NOTE]
>
> Les mesures de mémoire JVM sont signalées **par pool de mémoire**, donc `average(value)` sur les moyennes `JvmHeapUsed`/`JvmHeapLimit` entre les pools. Utilisez `FACET host, pool` pour obtenir une vue exacte par pool.

### `SpanBreakdown` : composition des transactions pré-agrégées

`SpanBreakdown` contient la composition enfant-span par minute de chaque transaction. Il est calculé à l&#39;ingestion, donc il est bon marché à interroger.

| Attribut | Type | Description |
| --------------- | --------- | ---------------------------------------------------------------------- |
| `timestamp` | date et heure | Compartiment d’une minute |
| `service` | chaîne | Service |
| `rootOperation` | chaîne | Nom de la transaction parent |
| `segment` | chaîne | Nom de l’étendue enfant |
| `callCount` | nombre | Appels de ce segment dans l’intervalle |
| `avgDurationMs` | nombre | Durée moyenne du segment dans l’intervalle (définissez l’unité sur `ms` manuellement) |
