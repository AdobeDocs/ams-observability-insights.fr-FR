---
title: 'Tutoriel 3 : Avancer dans votre tableau de bord'
description: Créez des tableaux de bord avec des variables, des formules et plusieurs widgets dans Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '902'
ht-degree: 3%
---

## Tutoriel 3 : Avancer dans votre tableau de bord {#tutorial-3-advance-your-dashboarding}

Dans ce tutoriel, vous allez créer les composants d’un tableau de bord de niveau production : scores d’expérience utilisateur, taux de compteur, regroupement des cas, histogrammes, mosaïques d’indicateurs clés de performance avec seuils, sections, widgets de formule et variables.

**Dans ce tutoriel :** [8.1 apdex](#81-score-user-experience-with-apdex) · [8.2 taux](#82-turn-counters-into-rates-with-rate) · [8.3 Cas](#83-group-values-into-cases-with-filter) · [8.4 Histogrammes](#84-build-a-latency-histogram) · [8.5 IPC tuiles et sections](#85-build-kpi-tiles-and-sections) · [8.6 Formules](#86-combine-queries-with-formula-widgets) · [8.7 Variables](#87-make-dashboards-reusable-with-variables) · [8.8 Assemblez-le](#88-put-it-together)

### Noter l’expérience utilisateur avec `apdex`

Apdex transforme la latence et les erreurs en un score compris entre 0 et 1. Avec un seuil de 500 ms, une requête est _satisfaite_ inférieure ou égale à 500 ms, _tolérée_ jusqu’à 2 000 ms et _frustrée_ supérieure à ce **ou en cas d’échec** :

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

Tendance par service :

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

Insérez-le dans un widget **Nombre** avec des seuils `0 → Red`, `0.7 → Amber` et `0.85 → Green`. Le groupe qui correspond le mieux gagne, donc les scores bas restent rouges.

### 8.2 Transformer les compteurs en taux avec `rate`

Les entités de compteur (✔ dans le [dictionnaire de données](#53-curated-metric-entities)) stockent des totaux toujours croissants. `rate(value)` les convertit en valeurs par seconde. Elle a besoin d’`TIMESERIES` et doit être le seul agrégat :

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host, device TIMESERIES AUTO LIMIT 10
```

>[!WARNING]
>
> Toujours facettiser les `rate()` par `host` (et par `device` pour les disques) ou les filtrer sur un hôte. Sans cela, les lectures des différents hôtes se mélangent et le résultat est dénué de sens.

Pour obtenir un taux de **requête** à partir des traces, comptez plutôt les requêtes dans des intervalles fixes. Cela donne des requêtes par minute :

```sql
SELECT count(*) AS requestsPerMinute FROM Transaction TIMESERIES 1 minute
```

### Regrouper les valeurs dans des cas avec des `FILTER`

La `FACET CASES` de NRQL regroupe les lignes dans des compartiments nommés. Dans SQL, écrivez une colonne `FILTER` par cas. Voici les classes de statut par service :

```sql
SELECT count(*) FILTER (WHERE http.statusCode < 400) AS ok, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS clientErrors, count(*) FILTER (WHERE http.statusCode >= 500) AS serverErrors FROM Transaction FACET service LIMIT 20
```

La même idée fonctionne au fil du temps, donnant une ligne par cas :

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/graphql/%') AS graphql FROM Transaction TIMESERIES AUTO
```

### Création d’un histogramme de latence

SQL n&#39;a pas de fonction `histogram()`. Créez vous-même les compartiments avec des `FILTER` et effectuez le rendu du résultat sous la forme d’un widget **Barre** ou **Tableau** :

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS 'le 100ms', count(*) FILTER (WHERE duration > 100 AND duration <= 250) AS '100-250ms', count(*) FILTER (WHERE duration > 250 AND duration <= 500) AS '250-500ms', count(*) FILTER (WHERE duration > 500 AND duration <= 1000) AS '0.5-1s', count(*) FILTER (WHERE duration > 1000 AND duration <= 2000) AS '1-2s', count(*) FILTER (WHERE duration > 2000) AS 'over 2s' FROM Transaction
```

Ajoutez des `FACET service` pour comparer les distributions de plusieurs services côte à côte.

### Créer des mosaïques et des sections d’indicateurs de performance clés

Un bon tableau de bord commence par une ligne de mosaïques d’indicateurs clés de performance, puis regroupe les graphiques de détails en sections.

1. Ajoutez quatre widgets **Nombre** de 3 colonnes chacun :

   | Titre | Requête | Unité | Seuils |
   | -------- | --------------------------------------------------------------------------- | ---- | ---------------------------------------- |
   | Demandes | `SELECT count(*) AS requests FROM Transaction` |      |                                          |
   | Erreur % | `SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction` | `%` | `0 → Green`, `1 → Amber`, `5 → Red` |
   | p95 | `SELECT percentile(duration, 95) AS p95 FROM Transaction` | `ms` | `0 → Green`, `800 → Amber`, `2000 → Red` |
   | Apdex | `SELECT apdex(duration, 500) AS apdex FROM Transaction` |      | `0 → Red`, `0.7 → Amber`, `0.85 → Green` |

2. Ajoutez un widget **Texte**, cochez la case _En-tête de section_, puis appelez-le **LATENCE**. Ajoutez des graphiques de latence en dessous.
3. Ajoutez un autre en-tête de section, **ERREURS**, suivi de graphiques d’erreurs. Réduisez une section avec son chevron pour mettre l’accent sur l’autre.
4. Utilisez **onglets** pour différentes audiences, par exemple _Présentation_, _AEM_, _Infrastructure_. Seul l&#39;onglet visible exécute ses requêtes.

### Combinaison de requêtes avec des widgets de formule

SQL ne peut pas faire de maths dans `SELECT` et n&#39;a pas de sous-requêtes. **Les widgets de formule** comblent ce vide : ils exécutent 2 à 6 requêtes nommées et combinent leurs résultats avec des `+ - * / ( )`.

**Exemple : requêtes par hôte au fil du temps** (formule, ligne)

1. **Ajouter un widget → une formule**.
2. Nommez la première `requests` de requête et la seconde `hosts`.
3. Saisissez les requêtes et la formule, puis cliquez sur **Exécuter la formule**.

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

**Exemple : part de temps passé dans la base de données** (Formule, Nombre, `%` unitaires)

`db` de requête :

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != ''
```

`server` de requête :

```sql
SELECT sum(duration) AS v FROM Transaction
```

Formule :

```text
(db / server) * 100
```

Règles pour les requêtes de formule :

| Règle | Détails |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| leur nom ; | Lettres, chiffres et `_`, commençant par une lettre ou un `_` ; jusqu’à 40 caractères ; unique |
| Expression | `+ - * / ( )`, nombres et noms de requête ; jusqu’à 500 caractères |
| Même forme | Toutes les requêtes sont des totaux simples (Nombre), **ou** toutes sont `TIMESERIES` avec la **même taille de compartiment** (Ligne / Zone / Barre) |
| Une mesure chacune | Chaque requête renvoie exactement un agrégat et aucune `FACET` |
| Données manquantes | Une valeur manquante ou une division par zéro génère un point vide, jamais `Infinity` ni `NaN` |

>[!TIP]
>
> Utilisez des `TIMESERIES AUTO` dans chaque requête d’une formule de série temporelle. Comme les requêtes partagent une fenêtre, elles sont toujours résolues sur la même taille de compartiment.

### Rendre les tableaux de bord réutilisables avec des variables

Les variables permettent à un tableau de bord de servir plusieurs services, niveaux ou seuils. Chaque variable est un sélecteur dans la barre des variables. Référencez-le en tant que `{{name}}` dans les titres de widget et les widgets de texte (et, une fois la limitation ci-dessous corrigée, dans le code SQL du widget).

**Créez une variable** (_org et super admins_) : cliquez sur **+ Variable**.

| Champ | Remarques |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Nom** | Lettres, chiffres et `_`, en commençant par une lettre ou un `_`. Doit être unique. Utilisé comme `{{name}}`. |
| **Libellé** | Affichage du sélecteur (facultatif). |
| **Type** | **Liste personnalisée** (une valeur par ligne), **À partir d’une requête** (une requête SQL avec un seul `FACET` ; ses valeurs distinctes deviennent des options) ou **Zone de texte** (texte libre). |
| **Autoriser la sélection de plusieurs valeurs** | Se développe en liste séparée par des virgules. Utilisez-le avec `IN (…)`. |
| **Les valeurs sont des nombres** | Injecte les valeurs sans guillemet (`500` au lieu de `'500'`). |

**Comment fonctionne la substitution**

| Sélection | `{{svc}}` devient |
| ---------------------------------- | ------------------------------------------------------- |
| `checkout` à chaîne unique | `'checkout'` |
| Valeurs multiples `checkout`, `cart` | `'checkout', 'cart'` |
| Nombre `500` | `500` |
| Valeur contenant des `'` | placée dans une séquence d’échappement `''` |
| Aucune sélection | Le widget attend et affiche _Sélectionnez une valeur pour{{svc}}_ |

```text
WHERE service = {{svc}}            single select
WHERE service IN ({{svc}})         multiple select
WHERE duration > {{slowMs}}        numeric text box
```

Pour une variable pilotée par une requête, **ajoutez toujours une`LIMIT`**. Une requête à facettes renvoie uniquement les 10 premières valeurs par défaut. Vous y trouverez jusqu’à 200 services :

```sql
SELECT count(*) FROM Transaction FACET service LIMIT 200
```

Les options sont actualisées lorsque le client ou le sélecteur d’heure change et sont triées par ordre alphabétique. Lorsque la sélection en cours n’est plus valide, la première valeur (sélection unique) ou toutes les valeurs (sélection multiple) sont automatiquement sélectionnées. La requête d’une variable ne peut pas référencer d’autres variables.

>[!WARNING]
>
> **Limitation actuelle de build.** La substitution se produit dans la grille du tableau de bord, mais l’aperçu **Ajouter un widget** et le contrôle d’enregistrement côté serveur **server-side save check** reçoivent le jeton de `{{name}}` brut et le rejettent avec `Unexpected character '{'`. En pratique :
>
> - `{{variables}}` dans les widgets **titres de widget** et **texte** fonctionnent de bout en bout.
> - `{{variables}}` à l’intérieur du **widget SQL** le rendu est correct après une **importation**, mais le tableau de bord ne peut pas être **enregistré** tant que de tels widgets existent.
>
> Tant que ce problème n’est pas résolu, créez des widgets par contexte avec des filtres littéraux (par exemple, `WHERE service = 'checkout'`) et utilisez **Dupliquer** pour les cloner.

### Assemblez-le

Chaque bloc de création est désormais utilisé dans les [plans directeurs de tableau de bord](#dashboard-blueprints) : mosaïques d’indicateurs clés de performance avec seuils, sections, centiles, index, taux, colonnes de cas, histogrammes, formules et variables. Les plans directeurs montrent comment les combiner dans des tableaux de bord complets de signal d’or, de tri d’erreurs, d’AEM, d’infrastructure et JVM.
