---
title: Référence SQL
description: Référencez la syntaxe SQL, les clauses et les fonctions prises en charge par Observability Insights.
source-git-commit: 898e474e3ab7914f8de2df286c67ac046d3b22b0
workflow-type: tm+mt
source-wordcount: '1504'
ht-degree: 12%
---

## Référence SQL {#sql-reference}

Cette référence couvre la syntaxe SQL, les clauses, les fonctions et les opérateurs. Développez chaque entrée pour obtenir des détails et des exemples.

### Composants de requête

```text
SELECT <select-item> [, <select-item> ...]
FROM   <Entity>
[WHERE      <condition>]
[FACET      <attribute> [, <attribute> ...]]          -- up to 5
[SINCE      <n> <unit> [AGO]]
[UNTIL      NOW | <n> <unit> [AGO]]
[TIMESERIES AUTO | MAX | <n> <unit>]
[ORDER BY   <alias-or-selected-name> [ASC | DESC]]
[LIMIT      <1..5000>]

<select-item> := <function>(<args>) [FILTER (WHERE <condition>)] [AS <alias>]
               | <attribute> [AS <alias>]
<unit>        := second | seconds | minute | minutes | hour | hours | day | days | week | weeks
```

>[!NOTE]
>
> Les annotations `--` ci-dessus sont uniquement de la documentation. SQL lui-même n&#39;accepte pas les commentaires.

### Clauses obligatoires

#### `SELECT` — que retourner

+++Afficher les détails

`SELECT` répertorie **attributs bruts** (requête d’événement) ou **fonctions d’agrégation** (requête d’agrégation). On ne peut pas mélanger les deux.

| Formulaire | Exemple | Résultat |
| ------------ | -------------------------------------------------- | ------------------------------------------------- |
| Attributs | `SELECT timestamp, name, duration` | Lignes d’événements individuels (100 par défaut) |
| Agrégats | `SELECT count(*), average(duration)` | Une valeur par agrégat (par facette/intervalle) |
| Avec des alias | `SELECT count(*) AS requests` | `requests` du libellé de la colonne |
| Alias cité | `SELECT percentile(duration, 95) AS 'p95 latency'` | Libellé avec espaces |

Liste des événements :

```sql
SELECT timestamp, service, name, duration, status, traceId FROM Transaction WHERE status = 'Error' LIMIT 50
```

Plusieurs agrégats :

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

Règles :

- `SELECT *` n’est pas pris en charge. Répertoriez les attributs souhaités.
- Les correspondances entre attributs ou agrégats (`sum(a) / count(*)`) ne sont pas prises en charge. Utilisez un [widget de formule](#86-combine-queries-with-formula-widgets).
- Le mélange des attributs et des agrégats échoue avec _Les attributs bruts ne peuvent pas être mélangés avec des agrégats ; utilisez la facette pour regrouper les résultats_.

+++

#### `FROM` : entité à interroger

+++Afficher les détails

`FROM` noms exacts **un** entité du [dictionnaire de données](#data-dictionary). Le nom de l’entité n’est pas sensible à la casse.

```sql
SELECT count(*) FROM Transaction
```

```sql
FROM Span SELECT count(*) FACET kind
```

Chaque entité possède une portée fixe. Par exemple, `Transaction` correspond toujours à l’étendue du serveur et `CpuUtilization` correspond toujours à la mesure CPU occupé avec `value` converti en pourcentage. Vous ne pouvez pas interroger plus d&#39;une entité dans une seule requête. Pour combiner des entités, utilisez un [widget de formule](#86-combine-queries-with-formula-widgets).

+++

### Clauses facultatives

#### `AS` — étiqueter une colonne

+++Afficher les détails

Utilisez `AS` pour donner à un agrégat ou à un attribut un nom lisible. L’alias devient l’en-tête de colonne, le libellé de légende et le nom que vous référencez dans `ORDER BY`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET service ORDER BY p95 DESC LIMIT 10
```

Utilisez une chaîne entre guillemets simples pour les libellés comportant des espaces :

```sql
SELECT percentile(duration, 95) AS 'p95 latency (ms)' FROM Transaction TIMESERIES AUTO
```

+++

#### `FACET` — résultats du groupe

+++Afficher les détails

`FACET` regroupe les résultats en fonction d’attributs **5** maximum, tels que les `GROUP BY` SQL.

```sql
SELECT count(*) FROM Transaction FACET service, http.method, http.statusCode LIMIT 30
```

| Requête | Ce que `LIMIT n` signifie | Par défaut |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `FACET` sans `TIMESERIES` | Nombre de lignes | 10 |
| `FACET` avec `TIMESERIES` | **N premières séries**, classées par la somme du premier agrégat sur l’ensemble de la fenêtre. Chaque intervalle est conservé pour ces séries. | Toutes les séries (avec une limite de 5 000 lignes) |

Série temporelle à facettes, top 5 :

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Les valeurs de facettes peuvent être des chaînes ou des nombres. Les chaînes vides forment leur propre groupe. Les exclure avec `WHERE attr != ''`.

+++

#### `FILTER` — agrégats conditionnels

+++Afficher les détails

Ajoutez des `FILTER (WHERE condition)` à n’importe quel agrégat pour le calculer sur un sous-ensemble de lignes. Vous pouvez ainsi comparer plusieurs conditions en une seule passe.

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentile(duration, 95) FILTER (WHERE http.method = 'GET') AS getP95 FROM Transaction FACET service LIMIT 20
```

`FILTER` n’est pas autorisé avec `apdex`, `rate` ou `derivative`. `percentage` prend déjà sa propre condition.

+++

#### `LIMIT` : nombre de lignes ou de séries

+++Afficher les détails

`LIMIT` prend un entier positif. Les valeurs supérieures à 5 000 sont fixées à 5 000.

| Forme du résultat | Par défaut |
| ------------ | ---------- |
| Panneau D&#39;Affichage | 1 |
| Facette | 10 |
| Série temporelle | 5 000 lignes |
| Événements | 100 |

```sql
SELECT count(*) FROM Transaction FACET urlPath LIMIT 50
```

>[!TIP]
>
> Définissez toujours un `LIMIT` sur les variables pilotées par les requêtes et sur les facettes de cardinalité élevée (`urlPath`, `traceId`, `jcr.path`, `url`).

+++

#### `ORDER BY` — trier les résultats

+++Afficher les détails

`ORDER BY` trie selon un **alias sélectionné ou attribut sélectionné**. La direction par défaut est `DESC`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

| Forme du résultat | Ordre par défaut |
| ----------------- | --------------------------------------------------------------------------- |
| Panneau publicitaire / facette | Premier agrégat, `DESC` |
| Série temporelle | Heure, `ASC` |
| Événements | Première colonne sélectionnée, `DESC` (`timestamp` donc en premier pour « la plus récente ») |

Règles :

- Référencer quelque chose qui n&#39;est pas sélectionné échoue avec _ORDER BY &#39;x&#39; doit référencer une valeur ou un alias sélectionné_.
- N’ajoutez pas de `ORDER BY` à une requête `TIMESERIES`. Il remplace l’ordre de temps nécessaire aux graphiques.

+++

#### `SINCE` — début de la fenêtre

+++Afficher les détails

`SINCE n unit [AGO]` définit l’intervalle à partir duquel la fenêtre commence. Unités : `second`, `minute`, `hour`, `day`, `week` (au singulier ou au pluriel). `AGO` est facultatif.

```sql
SELECT count(*) FROM Transaction SINCE 6 hours ago
```

- La valeur par défaut est `SINCE 1 hour ago`.
- La fenêtre ne peut pas dépasser **31 jours** (_la période ne peut pas dépasser 31 jours_).
- Les dates absolues (`SINCE '2026-01-01'`) et `TODAY` / `YESTERDAY` ne sont pas prises en charge.

>[!IMPORTANT]
>
> Dans les tableaux de bord et dans Explorer, le **sélecteur d’heure) remplace** `SINCE` et `UNTIL`. Voir [ Priorité de la fenêtre temporelle ](#time-window-precedence).

+++

#### `TIMESERIES` — tracé dans le temps

+++Afficher les détails

`TIMESERIES` renvoie une ligne par intervalle temporel (par valeur de facette), qui est prête pour les graphiques en courbes, en aires ou à barres.

| Formulaire | Taille du compartiment |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIMESERIES 5 minutes` | Exactement ce que vous demandez (n&#39;importe quelle unité) |
| `TIMESERIES AUTO` | A propos de la fenêtre ÷ 120, arrondie à 1s / 5s / 10s / 15s / 30s / 1m / 5m / 10m / 15m / 30m / 1h / 3h / 6h / 12h / 1d. **Jamais en dessous de 2 minutes.** |
| `TIMESERIES MAX` | Intervalles d’une seconde. Utiliser seulement pendant quelques minutes. |

Intervalles de `AUTO` standard :

| Fenêtre | 1 h | 6 h | 24 h | 7 j | 30 j |
| ------ | ----- | ----- | ------ | --- | ---- |
| Compartiment | 2 min | 5 min | 15 min | 3 h | 6 h |

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction TIMESERIES AUTO
```

```sql
SELECT count(*) FROM Transaction TIMESERIES 1 hour SINCE 1 day ago
```

>[!TIP]
>
> Privilégiez les `AUTO` sur les tableaux de bord. Il conserve environ 120 points, quelle que soit l’heure indiquée par le sélecteur. Il évite également les lignes « brisées » par hôte, car les mesures de l’hôte sont grattées environ toutes les 90 secondes.

+++

#### `UNTIL` — fin de la fenêtre

+++Afficher les détails

`UNTIL NOW` (par défaut) ou `UNTIL n unit [AGO]`. Combinez-la avec `SINCE` pour consulter une fenêtre passée :

```sql
SELECT count(*) AS yesterday FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

`SINCE` doit être antérieur à `UNTIL`. Sinon, la requête échoue avec _SINCE doit être antérieur à Until_.

+++

#### `WHERE` — filtrer les lignes

+++Afficher les détails

`WHERE` conserve uniquement les lignes correspondant à une condition. Combinez des conditions avec des `AND`, `OR`, `NOT` et parenthèses. Voir [Opérateurs et conditions](#45-operators-and-conditions).

```sql
SELECT count(*) FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR http.statusCode >= 500)
```

`WHERE` ne peut apparaître qu’une seule fois. Mettez toutes les conditions dedans.

+++

### Fonctions d’agrégateur

| Fonction | Arguments | Renvoie | Travaille sur |
| -------------------------------------- | ----------------------------------- | ---------------------- | ------------------------------------- |
| [`apdex`](#fn-apdex) | durée numérique, seuil `T` | 0-1 | Trace des entités |
| [`average` / `avg`](#fn-average) | numérique | Même unité que l&#39;entrée | N’importe lequel |
| [`count`](#fn-count) | `*` ou attribut | Nombre de lignes | N’importe lequel |
| [`derivative`](#fn-derivative) | numérique | Modification par seconde | Any (nécessite `TIMESERIES`) |
| [`earliest`](#fn-latest) | n&#39;importe quel [, rang ] | Valeur d’attribut | N’importe lequel |
| [`latest`](#fn-latest) | n&#39;importe quel [, rang ] | Valeur d’attribut | N’importe lequel |
| [`max` / `min`](#fn-minmax) | quelconque | Même unité que l&#39;entrée | N’importe lequel |
| [`percentage`](#fn-percentage) | `*` ou attribut, condition de `WHERE` | % | N’importe lequel |
| [`percentile`](#fn-percentile) | numérique, niveau `p` | Même unité que l&#39;entrée | N’importe lequel |
| [`percentOfTotal`](#fn-percentoftotal) | numérique | % | Tous (pas de `TIMESERIES`) |
| [`rate`](#fn-rate) | compteur numérique | Par seconde | Entités de compteur (`TIMESERIES` nécessaire) |
| [`stddev`](#fn-stddev) | numérique | Même unité que l&#39;entrée | N’importe lequel |
| [`sum`](#fn-sum) | numérique | Même unité que l&#39;entrée | N’importe lequel |
| [`uniqueCount`](#fn-uniquecount) | quelconque | Valeurs distinctes | N’importe lequel |

<a id="fn-apdex"></a>

#### `apdex(attribute, T)`

+++Afficher les détails

Calcule un score [Apdex](https://www.apdex.org/) compris entre 0 (frustré) et 1 (satisfait), en utilisant le seuil `T` dans l’unité de l’attribut (millisecondes pour `duration`).

| Compartiment | Condition | Poids |
| ---------- | ------------------------------------ | ------ |
| Satisfait | `duration ≤ T` et non une erreur | 1 |
| Tolérant | `T < duration ≤ 4T` et non une erreur | 0.5 |
| Frustré | `duration > 4T`, **ou toute erreur** | 0 |

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service
```

- Fonctionne uniquement sur les entités de suivi (`Transaction`, `AppRequest`, `Span`).
- `T` doit être supérieur à 0. `FILTER` n&#39;est pas autorisé.
- `apdex(duration, 500)` sur `Transaction` est accéléré par cumul sur de longues fenêtres.

+++

<a id="fn-average"></a>

#### `average(attribute)` / `avg(attribute)`

+++Afficher les détails

Moyenne arithmétique d’un attribut numérique.

```sql
SELECT average(duration) AS avgMs FROM Transaction FACET service TIMESERIES AUTO
```

+++

<a id="fn-count"></a>

#### `count(*)` / `count(attribute)`

+++Afficher les détails

- `count(*)` compte les lignes.
- `count(attribute)` compte les lignes dont l&#39;attribut n&#39;est **pas NULL**. Pour les attributs de chaîne, les valeurs manquantes sont vides (`''`) plutôt que NULL. Elles sont donc toujours comptabilisées (voir [ Gestion des types ](#46-type-handling-blank-vs-null)).

```sql
SELECT count(*) AS requests, count(http.statusCode) AS withStatusCode FROM Transaction
```

+++

<a id="fn-derivative"></a>

#### `derivative(attribute)`

+++Afficher les détails

Changement de `average(attribute)` par seconde entre des intervalles consécutifs : (cet intervalle − l’intervalle précédent) ÷ intervalles de secondes. Le résultat peut être négatif. Utilisez-la sur des jauges pour voir si une valeur augmente ou diminue.

```sql
SELECT derivative(value) AS heapGrowthPerSec FROM JvmHeapUsed WHERE host = 'publish-1' TIMESERIES 5 minutes
```

- Nécessite `TIMESERIES` et doit être le **seul** agrégat. `FILTER` n&#39;est pas autorisé.
- Le premier compartiment n’a pas de prédécesseur.

+++

<a id="fn-latest"></a>

#### `latest(attribute [, rankAttribute])` / `earliest(…)`

+++Afficher les détails

- `latest(attr)` renvoie la valeur de la ligne la plus récente. `earliest(attr)` renvoie la valeur de la ligne la plus ancienne.
- Avec un deuxième argument, la ligne est choisie par cet attribut plutôt que par le temps. `latest(traceId, duration)` renvoie l’ID de suivi de la requête **lest**. `earliest(traceId, duration)` renvoie le plus rapide.

```sql
SELECT latest(value) AS current FROM MemoryUtilization FACET host
```

```sql
SELECT max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 10
```

+++

<a id="fn-minmax"></a>

#### `max(attribute)` / `min(attribute)`

+++Afficher les détails

Valeur la plus grande ou la plus petite. Fonctionne sur les nombres, les chaînes et les horodatages.

```sql
SELECT min(duration) AS fastestMs, max(duration) AS slowestMs, max(timestamp) AS lastSeen FROM Transaction FACET service
```

+++

<a id="fn-percentage"></a>

#### `percentage(*, WHERE condition)`

+++Afficher les détails

Part de lignes correspondant à `condition`, de 0 à 100.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO
```

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

`percentage` a sa propre condition, donc cela ne prend pas de `FILTER`.

+++

<a id="fn-percentile"></a>

#### `percentile(attribute, p)`

+++Afficher les détails

Le p-ième centile approximatif, où `0 < p ≤ 100`. Utilisez une `percentile` par niveau.

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

p50, p95 et p99 sur `Transaction` sont accélérés par cumul sur de longues fenêtres.

+++

<a id="fn-percentoftotal"></a>

#### `percentOfTotal(attribute)`

+++Afficher les détails

Part de chaque ligne de facette dans la **somme totale** d’un attribut numérique. Les lignes s’additionnent jusqu’à 100 %.

```sql
SELECT sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfTime FROM Transaction FACET name LIMIT 15
```

Non autorisé avec `TIMESERIES`.

+++

<a id="fn-rate"></a>

#### `rate(attribute)`

+++Afficher les détails

Taux par seconde d&#39;un **compteur cumulé**. Dans chaque intervalle, `rate` prend le relevé de compteur le plus élevé par série et le diffère par rapport à l&#39;intervalle précédent. Les réinitialisations de compteur sont planifiées à 0, et le premier compartiment est 0.

```sql
SELECT rate(value) AS bytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

- Fonctionne uniquement sur les entités de compteur (marquées ✔ dans le [dictionnaire de données](#53-curated-metric-entities)).
- Nécessite `TIMESERIES` et doit être le **seul** agrégat. `FILTER` n&#39;est pas autorisé.

>[!WARNING]
>
> Toujours `FACET host` ou filtrer sur un seul hôte. Un `rate()` sans facettes sur de nombreux hôtes compare les compteurs des différents hôtes et produit des nombres dénués de sens. Les compteurs de disque sont par périphérique, donc `FACET host, device` est le plus précis.

+++

<a id="fn-stddev"></a>

#### `stddev(attribute)`

+++Afficher les détails

Écart type de l’échantillon. Utilisez-le avec `average` pour évaluer la cohérence de la latence.

```sql
SELECT average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name LIMIT 20
```

+++

<a id="fn-sum"></a>

#### `sum(attribute)`

+++Afficher les détails

Total d’un attribut numérique.

```sql
SELECT sum(callCount) AS calls FROM SpanBreakdown FACET segment LIMIT 20
```

+++

<a id="fn-uniquecount"></a>

#### `uniqueCount(attribute)`

+++Afficher les détails

Nombre **exact** de valeurs distinctes.

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

+++

Contraintes de fonction en un coup d’œil :

| Contrainte | Fonctions |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| Attribut numérique obligatoire | `average`, `sum`, `percentile`, `stddev`, `percentOfTotal`, `rate`, `derivative`, `apdex` |
| Aucune `FILTER (WHERE …)` | `apdex`, `rate`, `derivative`, `percentage` |
| Nécessite `TIMESERIES`, agrégat uniquement | `rate`, `derivative` |
| Pas avec `TIMESERIES` | `percentOfTotal` |
| Trace des entités uniquement | `apdex` |
| Entités de compteur uniquement | `rate` |

### Opérateurs et conditions

| Opérateur | Signification | Exemple |
| ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `=` | Égal | `status = 'Error'` |
| `!=`, `<>` | Non égal à | `kind != 'Internal'` |
| `<`, `<=`, `>`, `>=` | Comparaison | `http.statusCode >= 500` |
| `LIKE` | Correspondance par caractères génériques : `%` = n’importe quel caractère, `_` = un caractère. Respect de la casse. | `urlPath LIKE '/content/%'` |
| `NOT LIKE` | Caractère générique annulé | `urlPath NOT LIKE '%.css'` |
| `IN (…)` | Correspond à l’une des valeurs 1 à 100 | `service IN ('checkout', 'cart')` |
| `NOT IN (…)` | Ne correspond à aucune des valeurs | `http.method NOT IN ('OPTIONS', 'HEAD')` |
| `IS NULL` / `IS NOT NULL` | Chèque nul | `http.statusCode IS NOT NULL` |
| `AND`, `OR`, `NOT` | Logique | `NOT (status = 'Ok')` |
| `( … )` | Regroupement | `(a = 1 OR b = 2) AND c = 3` |

```sql
SELECT count(*) FROM Transaction WHERE (status = 'Error' OR duration > 2000) AND NOT host LIKE '%dispatcher%' AND http.method IN ('GET', 'POST')
```

> **Avertissement**
>
> `attr = NULL` est une erreur. Utilisation `attr IS NULL`.

### Gestion des types : vide ou NULL

Chaque attribut possède un type dans le catalogue : `string`, `number`, `boolean` ou `timestamp`. SQL vérifie les fonctions associées (par exemple, `average(service)` est rejeté). Aucune fonction de conversion de type n’est nécessaire ou disponible.

> **Important**
>
> **Vide ou NUL.** La plupart des attributs de chaîne proviennent de mappages d’attributs OpenTelemetry, où une clé _manquante_ se lit comme une `''` de chaîne vide, et non comme une `NULL`. Pour ne conserver que les plages comportant un attribut, utilisez **`attr != ''`** plutôt que `IS NOT NULL`. Les attributs numériques tels que `http.statusCode` sont véritablement `NULL` en cas d’absence, donc `IS NOT NULL` fonctionne pour eux.

```sql
SELECT count(*) AS jcrCalls FROM Span WHERE jcr.operation != '' FACET jcr.operation
```

### Traduire SQL en SQL

| SQL | SQL |
| ------------------------------------------------------------ | ------------------------------------------------------------------- |
| `WHERE ts > now() - INTERVAL 1 HOUR` | Rien ; le sélecteur d’heure (ou `SINCE 1 hour ago`) définit la fenêtre |
| `GROUP BY service` | `FACET service` |
| `GROUP BY toStartOfInterval(ts, INTERVAL 5 MINUTE), service` | `FACET service TIMESERIES 5 minutes` |
| `quantile(0.95)(duration)` | `percentile(duration, 95)` |
| `countIf(status = 'Error')` | `count(*) FILTER (WHERE status = 'Error')` |
| `100 * countIf(x) / count()` | `percentage(*, WHERE x)` |
| `count(DISTINCT host)` | `uniqueCount(host)` |
| `argMax(traceId, duration)` | `latest(traceId, duration)` |
| `x / sum(x) OVER ()` | `percentOfTotal(x)` |
| `ORDER BY p95 DESC LIMIT 20` | `ORDER BY p95 DESC LIMIT 20` |
| `HAVING count() > 100` | _Non pris en charge_. Triez et `LIMIT` à la place. |
| `JOIN` / sous-requête | _Non pris en charge_. Utilisez un widget **Formule**. |
| `CASE WHEN` | _Non pris en charge_. Utilisez `FILTER` colonnes ou `WHERE … LIKE` + `FACET`. |

### Importation de tableaux de bord existants dans Insights

Si vous connaissez la requête existante, voici le détail de la migration

| Requête existante | SQL | Remarques |
| ------------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `FROM Transaction SELECT count(*)` | Identique | Le `FROM`-first order est pris en charge |
| `SINCE 1 day ago` | Identique | Ignoré sur les tableaux de bord ; le sélecteur gagne |
| `TIMESERIES AUTO` / `MAX` / `5 minutes` | Identique | `AUTO` étage est à 2 minutes |
| `FACET appName` | `FACET appName` / `FACET service` |                                                                                  |
| `percentage(count(*), WHERE error IS true)` | `percentage(*, WHERE status = 'Error')` |                                                                                  |
| `filter(count(*), WHERE …)` | `count(*) FILTER (WHERE …)` | Formulaire standard SQL |
| `apdex(duration, t: 0.5)` | `apdex(duration, 500)` | `duration` est en **ms** |
| `latest(x)` / `earliest(x)` | Identique | Plus un formulaire avec classement : `latest(x, rankAttr)` |
| `uniqueCount(x)` | Identique | Exact |
| `rate(count(*), 1 minute)` | `count(*)` avec `TIMESERIES 1 minute` | SQL `rate()` est réservé aux compteurs |
| `derivative(x, 1 minute)` | `derivative(x)` + `TIMESERIES` | Par seconde |
| `uniques(x)` | `count(*) … FACET x LIMIT n` |                                                                                  |
| `keyset()` / `SHOW EVENT TYPES` | [Requêtes de découverte](../examples/query-cookbook.md#data-discovery) ou le panneau du guide de requête |                                                                                  |
| `histogram(duration, …)` | `FILTER` compartiments | Voir [ Tutoriel 3](../tutorials/tutorial-3.md#build-a-latency-histogram) |
| `FACET CASES (…)` | `FILTER` colonnes | Voir [ Tutoriel 3](../tutorials/tutorial-3.md#group-values-into-cases-with-filter) |
| `COMPARE WITH 1 week ago` | Non pris en charge. | Modifiez le sélecteur d’heure ou utilisez `SINCE … UNTIL …` via l’API |
| `RLIKE` | `LIKE` / `IN` |                                                                                  |
| Mathématiques dans l’agrégation `SELECT`/imbriquée | **Formule** widget |                                                                                  |
| `WITH TIMEZONE`, `EXTRAPOLATE`, `SLIDE BY` | Non pris en charge. |                                                                                  |
