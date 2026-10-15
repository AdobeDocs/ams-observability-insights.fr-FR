---
title: Prise en main de SQL
description: Découvrez comment interroger la télémétrie Observability Insights à l’aide de SQL.
source-git-commit: 148bf6480461a0109b221b8e7c2f9ab2035a7d68
workflow-type: tm+mt
source-wordcount: '687'
ht-degree: 1%
---

# Commencer

**Langage de requête d’Adobe Observability Insights (SQL)** — interroger, représenter sous forme de graphique et de tableau de bord votre télémétrie dans Adobe Observability Insights.

|                |                                                                  |
| -------------- | ---------------------------------------------------------------- |
| **S’applique à** | Interface utilisateur d’Insights → **Explorer** et **Tableaux de bord** (tableaux de bord personnalisés) |
| **Audience** | Ingénieurs, ingénieurs et utilisatrices et utilisateurs et utilisatrices expérimentés qui connaissent SQL |

> **Conseil**
>
> Chaque bloc de code marqué `sql` est une requête SQL que vous pouvez coller directement dans **Explorer → SQL** ou dans **Tableaux de bord → Ajouter un widget → SQL**.

## Prise en main de SQL : le langage de votre télémétrie {#get-started-with-sql-the-language-of-your-telemetry}

Le langage de requête d’Insights est un langage de requête de type SQL pour vos données Observability Insights. Il s’agit de la même langue derrière chaque widget de tableau de bord personnalisé et la page **Explorer**, afin que vous puissiez créer les vues dont votre équipe a besoin.

SQL fonctionne avec des **entités**, qui sont des vues logiques conviviales telles que `Transaction`, `Span` ou `CpuUtilization`. Vous ne devez jamais interroger directement des tables de stockage. Observability Insights compile chaque requête SQL en une requête d’entrepôt de données paramétrée et sécurisée. Il étend toujours cette requête à votre client et à la fenêtre temporelle sélectionnée.

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

### Utilisation de SQL

Utilisez SQL pour :

- **Créer des graphiques** du débit, du taux d’erreur, de la latence, d’Apdex, de CPU, de la mémoire, du tas JVM, du disque et du réseau au fil du temps.
- **Répondez à des questions spécifiques** telles que « Quels chemins d’URL au niveau de publication ont la pire p95 ? » ou « quel domaine en aval échoue ? »
- **Enquêter sur les incidents** en analysant en profondeur les requêtes individuelles et les ID de suivi, à partir d’un KPI.
- **Créer des tableaux de bord** avec des indicateurs clés de performance, des tendances, des tableaux, des formules, des notes, des en-têtes de section et des variables.
- **Charges de travail Profile AEM** (Adobe Experience Manager) à l’aide des attributs Sling, JSP, JCR, Dispatcher et GraphQL.

SQL prend en charge les requêtes **événement**, qui renvoient des lignes de requête individuelles, et les requêtes **agrégées**, qui renvoient des totaux, des centiles, des taux, des répartitions et des séries temporelles.

> **Conseil**
>
> Les modèles de **Tableaux de bord → Créer un tableau de bord** sont entièrement créés à partir de SQL. Ouvrez n’importe quel widget de modèle avec l’icône en forme de crayon pour afficher sa requête et en tirer des enseignements.

### Ce que vous pouvez interroger

SQL peut interroger quatre familles de données. La liste complète se trouve dans le [dictionnaire de données](/help/references/data-dictionary.md).

| Famille | Entités | Questions courantes |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Traces (APM)** | `Transaction` (requêtes entrantes), `AppRequest` (requêtes + consommateurs de messages), `Span` (chaque étendue) | Débit, latence, erreurs, Apdex, appels en aval, chemins de code AEM |
| **Mesures d’exécution et d’hôte traitées** | `CpuUtilization`, `MemoryUtilization`, `LoadAverage5m`, `DiskUtilization`, `NetworkReceiveRate`, `JvmHeapUsed`, `JvmThreadCount`, `DispatcherRequestRate` et plus encore | Saturation, capacité, intégrité de JVM, charge du Dispatcher |
| **Mesures génériques** | `MetricGauge`, `MetricSum` | Toute mesure OpenTelemetry par nom, découverte de mesures |
| **Composition d’étendue préagrégée** | `SpanBreakdown` | Les opérations enfants qui constituent une transaction |

> **Remarque**
>
> Les journaux, les définitions d’alerte et les relations d’inventaire ne sont pas disponibles.

### Où utiliser SQL

| Où : | Comment l’ouvrir | C&#39;est pour quoi ? |
| ----------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Explorer** | Navigation de gauche → **Explorer** | Analyse ad hoc. Utilisez l&#39;onglet **Visual Builder** ou **SQL**, **Valider**, puis **Exécuter** (⌘/Ctrl + Entrée). Comprend des requêtes **Historique** et **Enregistrées** ainsi qu’un panneau **Guide de requête**. |
| **Tableaux de bord** | La navigation de gauche → **Tableaux de bord** → ouvrir ou créer un tableau de bord → **Ajouter un widget** | Graphiques, KPI et tableaux persistants, combinés en tableaux de bord. |
| **Widgets de formule** | **Ajouter un widget → une formule** | Combinez 2 à 6 requêtes SQL avec des opérations arithmétiques (ratios, headroom, taux de combustion). |
| **Variables de tableau de bord** | **+ Variable → d’une requête** | Renseignez un sélecteur à partir des valeurs distinctes d’une requête SQL `FACET`. |
| **Inspection des requêtes** | Survoler un graphique → inspecter la requête | Affichez le code SQL sous-jacent à un graphique intégré ou de tableau de bord. |

### Structure de requête SQL

SQL suit une syntaxe basée sur des clauses. Cette requête comptabilise toutes les demandes entrantes dans la fenêtre sélectionnée :

```sql
SELECT count(*) FROM Transaction
```

Voici toutes les clauses disponibles :

```text
SELECT  function(attribute) [FILTER (WHERE condition)] [AS alias] [, ...]
        | attribute [AS alias] [, ...]
FROM    Entity
  [WHERE      condition [AND | OR ...]]
  [FACET      attribute [, attribute ...]]        -- up to 5
  [SINCE      n unit [AGO]]
  [UNTIL      NOW | n unit [AGO]]
  [TIMESERIES AUTO | MAX | n unit]
  [ORDER BY   alias [ASC | DESC]]
  [LIMIT      number]                              -- 1 to 5000
```

Les règles SQL sont les suivantes :

| Règle | Détails |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Clauses obligatoires** | `SELECT` et `FROM` sont requis. Toutes les autres clauses sont facultatives. Vous pouvez commencer par : `FROM Transaction SELECT count(*)` est également valide. |
| **Ordre des clauses** | Les clauses après `FROM` peuvent apparaître dans n’importe quel ordre. Chaque clause ne peut apparaître qu&#39;une seule fois. |
| **Taille de la requête** | Une requête doit faire 4 Ko ou moins. |
| **Respect de la casse** | Les mots-clés, noms de fonction, noms d’entité et noms d’attribut ne respectent **pas** la casse. Valeurs de chaîne **sont sensibles** la casse (`'Error'` ≠ `'error'`). |
| **chaînes** | Utilisez des guillemets simples : `WHERE status = 'Error'`. Pour inclure une citation, doublez-la : `'O''Brien'`. Les guillemets doubles ne sont pas acceptés. |
| **Littéraux** | Chaînes, nombres (y compris négatifs et décimaux), `TRUE`, `FALSE`, `NULL`. |
| **Noms d’attributs** | Les noms peuvent contenir des lettres, des chiffres, des `_` et des `.` (par exemple, `http.statusCode`). Pas besoin de backticks. |
| **Agrégats et attributs** | Une requête sélectionne des attributs bruts (une liste d’événements) ou des agrégats, jamais les deux. Pour regrouper, utilisez `FACET`. |
| **Période** | La valeur par défaut est la dernière heure. La durée maximale est de 31 jours. Dans les tableaux de bord et dans Explorer, le **sélecteur d’heure** définit la fenêtre (voir [Fenêtre temporelle](how-to-query-sql.md#time-window-precedence)). |
| **Non pris en charge** | Points-virgules, commentaires, sous-requêtes, jointures, `SELECT *`, correspondance entre les attributs dans `SELECT`, `HAVING`, `COMPARE WITH`. Voir [Ce que SQL ne fait pas](sql-query-examples.md#what-sql-doesnt-do) pour d’autres solutions. |
