---
title: 'Tutoriel 1 : Vos premières requêtes'
description: Écrivez vos premières requêtes SQL par rapport à la télémétrie Observability Insights.
source-git-commit: 52cc24792c47727700428be58b5bbb2a5c7223d2
workflow-type: tm+mt
source-wordcount: '552'
ht-degree: 0%
---

## Tutoriel 1 : Vos premières requêtes {#tutorial-1-your-first-queries}

Bienvenue dans SQL. Dans ce tutoriel, vous allez écrire vos premières requêtes, contrôler la fenêtre temporelle, tracer des tendances, filtrer et regrouper les résultats et placer votre premier graphique sur un tableau de bord. Chaque étape s’appuie sur la précédente.

### Avant de commencer

- Vous devez accéder à Observability Insights et à au moins un client qui envoie des traces ou des mesures.
- Sélectionnez ce **client** dans le sélecteur de client. SQL ne s&#39;exécute pas lorsque **Tous les comptes** est sélectionné.
- Ouvrez **Explorer** et passez à l’onglet **SQL**. Utilisez **Exécuter** (⌘/Ctrl + Entrée) après chaque requête.

### Événements de requête avec `SELECT`, `FROM` et `LIMIT`

Chaque requête nécessite deux clauses : `SELECT` indique ce qu’il faut renvoyer et `FROM` indique l’entité à lire. Commencez par les requêtes entrantes :

```sql
SELECT timestamp, service, name, duration FROM Transaction
```

Il s’agit d’une requête **événement**. Chaque ligne correspond à une requête. Sans `LIMIT`, vous obtenez jusqu’à 100 lignes. Demander moins :

```sql
SELECT timestamp, service, name, duration FROM Transaction LIMIT 10
```

Les lignes sont triées en fonction de la première colonne, par ordre décroissant. Ainsi, si vous placez `timestamp` en premier, vous obtenez les requêtes les plus récentes en haut.

> **Remarque**
>
> SQL n&#39;a pas de `SELECT *`. Les attributs de liste permettent de conserver des résultats petits et lisibles. Le [dictionnaire de données](#data-dictionary) et la saisie semi-automatique dans l’éditeur, affichent les éléments disponibles.

### Agréger vos données

La plupart des tableaux de bord n’affichent pas d’événements individuels. Ils les résument. Remplacez les attributs par une **fonction d’agrégation** :

```sql
SELECT count(*) FROM Transaction
```

Il s’agit du nombre de demandes entrantes dans la fenêtre. Essayez-en quelques autres :

```sql
SELECT average(duration) FROM Transaction
```

```sql
SELECT max(duration) FROM Transaction
```

```sql
SELECT uniqueCount(host) FROM Transaction
```

### Renvoyer plusieurs agrégats à la fois

Séparez les agrégats par des virgules et attribuez à chacun un nom `AS` :

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

> **Conseil**
>
> Les alias deviennent les en-têtes de colonne et les libellés de légende, et ils sont ce que vous référencez dans `ORDER BY`. Prenez l’habitude de nommer chaque agrégat.

### Définir la fenêtre temporelle

Par défaut, une requête couvre la **dernière heure**. Utilisez `SINCE` pour revenir en arrière :

```sql
SELECT count(*) AS requests FROM Transaction SINCE 1 day ago
```

Utilisez `UNTIL` pour fermer la fenêtre dans le passé. Il compte le trafic d&#39;hier, qui est passé de 48 à 24 heures auparavant :

```sql
SELECT count(*) AS requests FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

Les unités peuvent être `second`, `minute`, `hour`, `day` ou `week`. La fenêtre ne peut pas excéder 31 jours.

> **Important**
>
> Dans **Explorer** et dans **tableaux de bord**, le **sélecteur d’heure** situé en haut de la page contrôle la fenêtre et remplace `SINCE`/`UNTIL`. Pour consulter une autre période, modifiez le sélecteur. Pour cette raison, le reste de ce guide `SINCE` exclut des requêtes de tableau de bord.

### Tracer les tendances avec `TIMESERIES`

Ajoutez des `TIMESERIES` pour transformer un nombre unique en ligne au fil du temps :

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

`AUTO` choisit une taille de compartiment de sorte que le graphique ait environ 120 points (2 minutes pour une fenêtre d’une heure, 15 minutes pour 24 heures). Vous pouvez également définir le compartiment explicitement :

```sql
SELECT average(duration) AS avgMs FROM Transaction TIMESERIES 5 minutes
```

### Filtrer avec `WHERE`

`WHERE` ne conserve que les lignes qui vous intéressent. Compter uniquement les requêtes ayant échoué :

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

Combinez des conditions avec des `AND`, des `OR` et des parenthèses :

```sql
SELECT count(*) AS slowOrFailed FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR duration > 2000)
```

Utilisez `LIKE` avec `%` pour les correspondances partielles :

```sql
SELECT count(*) AS contentRequests FROM Transaction WHERE urlPath LIKE '/content/%'
```

> **Remarque**
>
> Les valeurs de chaîne respectent la casse et utilisent des guillemets simples : `'Error'`, pas `"Error"` ni `'error'`. Remplacez `checkout` par un service de votre client. L’étape suivante indique comment les répertorier.

### Regrouper avec des `FACET`

`FACET` groupe les résultats, comme les `GROUP BY` de SQL. Répertoriez vos services et le niveau d’occupation de chacun :

```sql
SELECT count(*) AS requests FROM Transaction FACET service
```

Une requête à facettes renvoie les 10 premiers groupes par défaut. Utilisez `LIMIT` pour en savoir plus :

```sql
SELECT count(*) AS requests, average(duration) AS avgMs FROM Transaction FACET service LIMIT 50
```

Combinez `FACET` et `TIMESERIES` pour obtenir une ligne par groupe :

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Ici, `LIMIT 5` garde les cinq séries les plus occupées.

### Enregistrer votre requête dans un tableau de bord

1. Ouvrez **Tableaux de bord → Créer un tableau de bord → Créez un tableau de bord personnalisé** et nommez-le _Mon premier tableau de bord_.
2. Cliquez sur **Ajouter un widget**, choisissez **Requête unique** et basculez le bouton (bascule) sur **SQL**.
3. Collez la requête à partir de l’étape 6.7 et cliquez sur **Exécuter la requête**.
4. Définissez le **Titre** sur _Demandes par service_ et choisissez le type de graphique **Ligne**.
5. Cliquez sur **Enregistrer le widget**, puis **Enregistrer** dans la barre supérieure.

Vous avez créé votre premier tableau de bord.
