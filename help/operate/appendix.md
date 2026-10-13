---
source-git-commit: ec98f6bd132cadaba1da7c4fb727d7e031790645
workflow-type: tm+mt
source-wordcount: '134'
ht-degree: 7%
---
## Annexe {#appendix}

**Dans ce document :** [14.1 Limites](#141-limits) · [14.2 Format du fichier d’exportation](#142-export-file-format) · [14.3 Aide-mémoire d’une page](#143-one-page-cheat-sheet)

### Limites

| Élément | Limite |
| -------------------------------- | ----------------------------------------------------------------------- |
| Texte de requête | 4 KO |
| Attributs `FACET` | 5 |
| `LIMIT` | 1 - 5 000 (valeurs par défaut : nombre 1, facette 10, événements 100, série temporelle 5 000) |
| `IN (…)` valeurs | 1 - 100 |
| Fenêtre temporelle | 31 jours |
| Délai d’expiration de la requête | 15 s (plus un budget de lecture d’octets) |
| `TIMESERIES AUTO` intervalle minimum | 2 minutes |
| Formule | 2 à 6 requêtes ; expression ≤ 500 caractères ; noms ≤ 40 caractères |
| Tableau de bord | ≤ 40 onglets · ≤ 200 widgets par onglet · ≤ 50 variables |
| Widget | titre ≤ 200 · sous-titre ≤ 500 · ≤ unitaire 40 · ≤ 10 seuils |
| Widget Texte | ≤ 4 000 caractères |
| Variable | nom ≤ 64 caractères · ≤ 500 options/valeurs · requête ≤ 4 Ko |

### Format du fichier d’export

Utilisez **Export** pour le produire ; **Import** pour le charger. Utile pour les tableaux de bord de contrôle de version ou pour les promouvoir entre les clients.

```json
{
  "name": "Golden Signals",
  "templateVariables": [
    {
      "id": "var-1",
      "name": "svc",
      "label": "Service",
      "kind": "query",
      "multi": true,
      "query": "SELECT count(*) FROM Transaction FACET service LIMIT 200",
      "value": null
    }
  ],
  "pages": [
    {
      "id": "page-1",
      "name": "Overview",
      "widgets": [
        {
          "id": "sec-kpis",
          "title": "SLO KPIs",
          "queryLanguage": "text",
          "text": "",
          "isSection": true,
          "collapsed": false,
          "layout": { "x": 0, "y": 0, "w": 12, "h": 1 }
        },
        {
          "id": "w-error-rate",
          "title": "Error %",
          "queryLanguage": "SQL",
          "query": "SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction",
          "vizType": "stat",
          "unit": "%",
          "thresholds": [
            { "value": 0, "status": "ok" },
            { "value": 1, "status": "warning" },
            { "value": 5, "status": "error" }
          ],
          "layout": { "x": 0, "y": 1, "w": 3, "h": 2 }
        },
        {
          "id": "w-throughput",
          "title": "Throughput by service",
          "queryLanguage": "SQL",
          "query": "SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8",
          "vizType": "area",
          "layout": { "x": 3, "y": 1, "w": 9, "h": 4 }
        },
        {
          "id": "w-availability",
          "title": "Availability %",
          "queryLanguage": "SQL-formula",
          "formulaQueries": [
            {
              "name": "ok",
              "query": "SELECT count(*) AS v FROM Transaction WHERE status != 'Error'"
            },
            {
              "name": "total",
              "query": "SELECT count(*) AS v FROM Transaction"
            }
          ],
          "formulaExpression": "(ok / total) * 100",
          "vizType": "stat",
          "unit": "%",
          "layout": { "x": 0, "y": 3, "w": 3, "h": 2 }
        }
      ]
    }
  ],
  "exportedAt": "2026-10-01T00:00:00.000Z"
}
```

`vizType` valeurs : `timeseries` (ligne), `area`, `bar`, `pie`, `stat` (nombre), `table`. Seuil `status` : `ok` (vert), `warning` (ambre), `error` (rouge).

### Aide-mémoire d’une page

```text
SHAPE        SELECT agg FROM Entity                         -> Number
             ... FACET a[, b..e] LIMIT n                     -> Bar / Pie / Table
             ... TIMESERIES AUTO                             -> Line / Area
             ... FACET a TIMESERIES AUTO LIMIT n             -> top-n series
             SELECT attr1, attr2 FROM Entity LIMIT n         -> event Table

AGGREGATES   count(*)  average(x)  sum(x)  min(x)  max(x)  uniqueCount(x)
             percentile(x, 95)  stddev(x)  latest(x[, rank])  earliest(x[, rank])
             percentage(*, WHERE c)  apdex(duration, 500)  percentOfTotal(x)
             rate(value)        -- counters, TIMESERIES only
             derivative(value)  -- gauges,   TIMESERIES only
             agg FILTER (WHERE c) AS alias

WHERE        = != < <= > >=   LIKE / NOT LIKE ('%', '_')   IN / NOT IN (...)
             IS [NOT] NULL   AND OR NOT ( )   strings in 'single quotes'
             string attr present?  ->  attr != ''

ORDER/LIMIT  ORDER BY alias [ASC|DESC]   LIMIT 1..5000

VARIABLES    = {{name}}   IN ({{name}})   numeric: > {{name}}

FORMULA      2-6 named queries, one metric each, no FACET, same shape
             expression: + - * / ( ) numbers names
```
