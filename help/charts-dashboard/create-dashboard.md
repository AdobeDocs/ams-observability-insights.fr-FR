---
source-git-commit: 05ed98c07602488843c52cd262af2d8cc204fef1
workflow-type: tm+mt
source-wordcount: '1117'
ht-degree: 2%
---
### Créer un tableau de bord, étape par étape

#### Étape 1 : Avant de commencer

- **Sélectionnez un seul client** dans le sélecteur de client. **Créer un tableau de bord** est désactivé lorsque **Tous les comptes** est sélectionné. SQL ne peut pas s&#39;exécuter sur le client de démonstration.
- **Roles.** Toute personne disposant d’un accès aux tableaux de bord peut ajouter, modifier et enregistrer des widgets. La création ou la modification de **variables** et l’inspection de la requête d’un widget se limitent aux **administrateurs d’organisation** et **super administrateurs**.

#### Étape 2 : créer le tableau de bord

1. Ouvrez **Tableaux de bord** dans le volet de navigation de gauche.
2. Cliquez sur **Créer un tableau de bord**.
3. Choisissez un point de départ :

   | Option | Ce que vous obtenez |
   | ---------------------------- | -------------------------------------------------------------------------------------------------------- |
   | **Créer un tableau de bord personnalisé** | Tableau de bord vide avec un seul onglet. |
   | **Concepts de base de** | CPU de l’hôte et du processus (utilisateur/système/iowait/steal), débit, chargement, Apdex. |
   | **Présentation tout-en-un** | KPI (volume, pourcentage d’erreur, Apdex, p95), secteurs d’erreur/de trafic, tables CPU/de mémoire, threads JVM, opérations lentes. |
   | **Principales transactions** | Apdex, p95 par opération, chargement par hôte/CPU, transactions les plus lentes avec ID de trace, répartition des erreurs HTTP. |

4. Saisissez un **nom** et confirmez. La zone de travail du tableau de bord s’ouvre.

#### Étape 3 : Découvrir la zone de travail

| Domaine | Contrôles |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Barre supérieure** | **Ajouter un widget**, **Guide** (aide-mémoire intégré à l’application), **Importer**, **Exporter**, **Enregistrer**, sélecteur d’heure et actualisation |
| **Barre variable** | Badge client, un sélecteur par variable, **+ Variable** (administrateurs) |
| **Barre de tabulation** | Basculer entre les onglets ; renommer (crayon), supprimer (×), ajouter (+) |
| **Grille** | 12 colonnes. Faites glisser les widgets près de la poignée, redimensionnez-les depuis le coin (minimum 2×2). |
| **En-tête du widget** (au survol) | Télécharger CSV, Télécharger PNG, Modifier, Dupliquer, Supprimer, faire glisser la poignée |

> [!IMPORTANT]
> **L’enregistrement est explicite.** Les modifications de disposition, les nouveaux widgets, onglets et variables restent dans votre navigateur jusqu’à ce que vous cliquiez sur **Enregistrer**. Le bouton indique _Enregistrement... → enregistré_ ou _Échec de l’enregistrement_ (voir [Dépannage](#troubleshooting)).

#### Étape 4 : ajouter un widget

Cliquez sur **Ajouter un widget**. La boîte de dialogue comporte trois modes : **Requête unique**, **Formule** et **Texte**. Il dispose également de son propre sélecteur d’heure. Vous pouvez donc prévisualiser le widget sur n’importe quelle fenêtre.

**Option A : le Builder (pas de code)**

1. _(Facultatif)_ **Commencez par un assistant** : choisissez l’un des 57 paramètres prédéfinis, regroupés sous la forme de signaux d’or, APM, AEM, Transactions, Traces et étendues, Infrastructure, Éditeurs, JVM et runtime et Découverte de données. Un préréglage remplit le titre, la requête, le type de graphique et l’unité.
2. **Afficher** : sélectionnez `Count`, `Average`, `Percentile` (et son niveau), `Sum`, `Min`, `Max` ou `Unique count`, puis un attribut.
3. **de** : sélectionnez l’entité.
4. **Filtre** : ajoutez des lignes de `attribute operator value` (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`). Toutes les lignes sont combinées avec `AND`.
5. **Regrouper par** : choisissez un attribut de facette, puis **N en haut** pour ne conserver que la série la plus occupée.
6. **Au fil du temps** : active `TIMESERIES`, avec une taille de compartiment **toutes les N minutes**.
7. Vérifiez le code SQL généré sous le formulaire, puis cliquez sur **Exécuter la requête**.

**Option B : éditeur SQL**

1. Basculez le bouton (bascule) de **Builder** vers **SQL**. La modification d’un widget existant s’ouvre toujours dans SQL.
2. Saisissez ou collez une requête. L’éditeur complète automatiquement à partir du catalogue dynamique.
3. Utilisez le panneau **Guide de requête** pour parcourir les entités, les attributs et les exemples.
4. Cliquez sur **Exécuter la requête**. Vous verrez _`<Entity>`requête est valide_ un aperçu en direct et un résumé (`N rows · N time points · N groups`). Un avertissement _série clairsemée_ s’affiche lorsqu’il y a moins de trois points temporels.

> [!TIP]
> Le Builder couvre les requêtes les plus courantes. Basculez vers SQL pour `apdex`, `percentage`, `rate`, `FILTER`, plusieurs agrégats, `OR`, `IN`, plusieurs facettes ou `ORDER BY`.

#### Étape 5 : Configurer l’apparence

| Champ | Remarques |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Titre** | Jusqu’à 200 caractères. Peut contenir des jetons `{{variable}}`. |
| **Type de graphique** | `Line`, `Area`, `Bar`, `Pie / Donut`, `Number`, `Table`. Un type de graphique est recommandé après chaque exécution jusqu’à ce que vous en choisissiez un vous-même. |
| **Unité** | Facultatif ; jusqu’à 40 caractères (`ms`, `%`, `req/s`, `MB`, `bytes`). Laissez vide pour utiliser l’unité déduite (par exemple `ms` pour `duration`, `%` pour `CpuUtilization`). |
| **Seuils de couleur** | Widgets numériques uniquement. Jusqu&#39;à 10 bandes de la forme _au `value` ou plus → Vert/Jaune/Rouge_. La bande correspondante la plus élevée gagne. |

#### Étape 6 : enregistrer le widget, puis le tableau de bord

Le **widget d’enregistrement** n’est activé qu’après le renvoi des données par une exécution. Les nouveaux widgets comportent 6 colonnes × 3 lignes et sont compressés à raison de deux par ligne. Organisez-les, puis cliquez sur **Enregistrer** dans la barre supérieure.

### Choisir le bon graphique

| Forme de requête | Meilleur graphique | Pourquoi ? |
| ------------------------------------------------- | ------------------- | ---------------------------------------------- |
| Un agrégat, pas de `FACET`, pas de `TIMESERIES` | **Nombre** | Une mosaïque d’indicateurs de performance clés unique ; ajoutez des seuils. |
| Agrégat + `TIMESERIES` | **Ligne** / **Zone** | Une tendance ; une série par valeur de facette. |
| Agrégat + `FACET` (quelques valeurs) | **Barre** / **Secteurs** | Compare les catégories ; secteur pour part d&#39;ensemble. |
| Plusieurs agrégats + `FACET` | **Tableau** | Chaque agrégat devient une colonne. |
| Attributs bruts | **Tableau** | Une liste d’événements. |
| Agrégat + `FACET` + `TIMESERIES` sur des données éparses | **Barre** | Lit mieux que des lignes joyeuses. |

Lorsque la ligne unique d’un widget Nombre contient une valeur de texte (par exemple `latest(host)`), le widget l’affiche sous forme de sous-titre.

### Widgets de texte : notes et en-têtes de section

**Ajouter un widget → Texte** :

- **Remarque** : texte brut (jusqu’à 4 000 caractères, lignes vides conservées). Utilisez-le pour les liens de runbook, les propriétaires et les définitions de SLO. Les `{{variables}}` sont rendues.
- **En-tête de section** : cochez la case _En-tête de section_. Il s’étend sur toute la largeur et son chevron réduit chaque widget situé en dessous, jusqu’à l’en-tête de section suivant. Utilisez-le pour des regroupements tels que **PUBLICATION**, **AUTEUR** et **DISPATCHER**.

> [!TIP]
> Les titres des widgets se terminant par `- PUBLISH` ou `- AUTHOR` reçoivent une indice de couleur subtil, ce qui facilite la numérisation des tableaux de bord AEM par niveau.

### Sélecteur d’heure

- Il définit la fenêtre pour **chaque** widget et pour les options de variable pilotée par les requêtes.
- Il remplace `SINCE` / `UNTIL` dans les requêtes de widget.
- La fenêtre maximale est de **31 jours**.
- `TIMESERIES AUTO` adapte sa taille de compartiment à la fenêtre.
- Le sous-titre de chaque widget affiche la fenêtre réellement appliquée ; par exemple, _Depuis il y a 1 heure_.
- Les résultats sont mis en cache pendant 30 secondes. Utilisez le bouton d’actualisation pour forcer une nouvelle exécution.

### Variables

Les variables permettent à un tableau de bord de servir de nombreux contextes, tels qu’un service, un environnement ou un seuil à la fois. Ils sont abordés en détail dans le [tutoriel 3](#87-make-dashboards-reusable-with-variables).

> [!WARNING]
> **Limites connues dans la version actuelle :** `{{variables}}` fonctionner dans les widgets **titres** et **texte**. Dans le widget **SQL**, ils s’affichent dans le tableau de bord, mais l’aperçu **Ajouter un widget** et le **Enregistrer** côté serveur vérifient et rejettent le jeton `{{name}}` brut avec _Caractère inattendu &#39;{&#39;_. Tant que ce problème n’est pas résolu, utilisez des valeurs littérales dans les requêtes de widget et utilisez **Dupliquer** pour créer des variantes.

### Widgets de formule

Une formule combine de 2 à 6 requêtes SQL nommées de manière arithmétique, par exemple `(errors / total) * 100`. Utilisez-en une lorsqu’un indicateur de performance clé nécessite plusieurs requêtes, par exemple un ratio, une marge, une somme de taux ou un taux de combustion. Voir [ Tutoriel 3](#86-combine-queries-with-formula-widgets).

### Partager, exporter et importer

| Action | Effets |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Enregistrer** | Conserve le nom, les onglets, les widgets, la disposition et les variables. Chaque requête de widget est revalidée côté serveur ; un widget non valide ne parvient pas à effectuer l’enregistrement complet. |
| **Exporter** | Téléchargements `<dashboard-name>.json` (onglets, widgets, variables). |
| **Importer** | Charge une exportation JSON **dans le tableau de bord ouvert** en remplaçant ses onglets et variables. Cliquez ensuite sur **Enregistrer**. Conseil : créez d’abord un tableau de bord vide, puis importez-le. |
| **CSV/PNG** | Téléchargements par widget à partir de l’en-tête du widget. |

> [!TIP]
> L’exportation et l’importation vous permettent de contrôler les versions des tableaux de bord dans Git et de promouvoir un tableau de bord d’un client à un autre. Le format du fichier est décrit dans l’[Annexe](#142-export-file-format).

### Conseils pour optimiser les performances des tableaux de bord et des graphiques

- **Utilisez des tableaux pour les jeux de résultats volumineux** au lieu de barres ou de lignes de séries multiples.
- **Fractionner les tableaux de bord occupés en onglets.** Seuls les widgets de l’onglet visible s’exécutent.
- **Toujours `LIMIT` les requêtes à facettes**, en particulier sur les attributs à cardinalité élevée tels que `urlPath`, `traceId` et `jcr.path`.
- **Filtrez tôt** sur les attributs de faible cardinalité (`service`, `kind`, `status`).
- **Préférez les`TIMESERIES AUTO`** aux petits compartiments fixes et évitez les `TIMESERIES MAX` sauf pendant quelques minutes.
- **Concevoir des tendances à long terme pour les cumuls.** Voir [Performances](#performance).
