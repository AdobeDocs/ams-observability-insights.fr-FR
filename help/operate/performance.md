---
title: Performance
description: Améliorez les performances des requêtes, des graphiques et des tableaux de bord dans Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '227'
ht-degree: 3%
---

## Performance {#performance}

Chaque requête s’exécute avec un délai d’expiration de 15 secondes et un budget de lecture d’octets. Ce document explique comment Observability Insights accélère automatiquement les requêtes et comment concevoir des tableaux de bord qui restent rapides à tout moment.

### Accélération de cumul (automatique)

Les graphiques `Transaction` lisent les tableaux préagrégés lorsque **tous** ceux-ci sont conservés :

| Condition | Condition requise |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Forme | La requête a `TIMESERIES` (les totaux des nombres/tableaux sont toujours lus en brut pour l&#39;exactitude) |
| Fenêtre | ≥ 3 h (niveau 1 min) · ≥ 1 j (niveau 5 min) · ≥ 7 j (niveau 1 h) |
| Agrégats | uniquement `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | uniquement le `service`, le `appInstance` et le `kind` |

`CpuUtilization` et `MemoryUtilization` (`average(value)`) et `JvmCpuRecentUtilization` (`average(value)`) graphiques avec `FACET host`/`WHERE host …` sur ≥ 3 h lisent des cumuls d’une minute.

**Conseil de conception :** pour les widgets de tendance de 7 à 31 jours, restez fidèle à l’ensemble adapté au cumul (p95, pourcentage d’erreur, Apdex @ 500, nombre, service FACET) - ils restent rapides quelle que soit la plage. Enregistrez les filtres `FACET host`/`name`, `average(duration)` ou `urlPath` pour les fenêtres ou les tableaux plus courts.

### Conseils généraux

- Utilisez des `LIMIT` sur chaque requête à facettes, en particulier `FACET urlPath`, `traceId` et `jcr.path` (cardinalité élevée).
- Filtrez tôt avec des `WHERE` sur les attributs de faible cardinalité (`service`, `kind`, `status`).
- Préférez la `Transaction` à la `Span` lorsque vous n’avez besoin que de requêtes entrantes, avec beaucoup moins de lignes.
- `uniqueCount` est exact ; sur les attributs de cardinalité élevée sur de longues fenêtres, il s’agit de l’agrégat le plus cher.
- Evitez les `TIMESERIES MAX` sauf les fenêtres de quelques minutes.
- Les widgets du tableau de bord sont mis en cache pendant 30 s ; le bouton d’actualisation force une nouvelle exécution.
- Un widget qui atteint le délai d’expiration de 15 secondes ou les erreurs de budget de lecture d’octets : rétrécit la fenêtre, ajoute des filtres ou réduit les facettes.
