---
source-git-commit: 05ed98c07602488843c52cd262af2d8cc204fef1
workflow-type: tm+mt
source-wordcount: '292'
ht-degree: 3%
---
## Tableaux de bord et graphiques : visualisation des données {#charts-and-dashboards-visualizing-your-data}

Les requêtes trouvent les informations dont vous avez besoin. Les graphiques et les tableaux de bord rendent ces informations utiles à votre organisation. Ce document explique comment les résultats SQL deviennent des graphiques et comment créer, organiser et partager des tableaux de bord.

### Graphiques

Chaque résultat SQL peut être rendu sous l’un des six types de graphique.

| Type de graphique | Utilisez-le pour | Forme de requête |
| --------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Ligne** | Tendances dans le temps ; une ligne par valeur de facette | Agrégat + `TIMESERIES` |
| **Zone** | Volume au fil du temps, par exemple débit par service | Agrégat + `TIMESERIES` |
| **Barre** | Comparaison de catégories ; séries temporelles éparses | Agrégat + `FACET` (± `TIMESERIES`) |
| **circulaire/anneau** | Part d’un tout (mix de codes d’état, trafic par niveau) | Un agrégat + `FACET`, quelques valeurs |
| **Nombre** | Un KPI unique, avec des seuils de couleur facultatifs | Panneau d’affichage (agrégats uniquement) |
| **Tableau** | Agrégats multiples par groupe ou listes d’événements | Plusieurs agrégats + `FACET` ou attributs bruts |

Une fois qu’un widget est sur un tableau de bord, vous pouvez :

- **Modifier** sa requête, son titre, son type de graphique et son unité (icône de crayon).
- **Dupliquer** permet de créer une variante, telle que le même graphique pour un autre niveau.
- **Télécharger CSV** des lignes actives ou **PNG** à la résolution 2×.
- **Faites-le glisser** à côté de la poignée et **redimensionnez** à partir du coin inférieur droit.

### Tableaux de bord

Les graphiques fonctionnent bien par eux-mêmes, mais les tableaux de bord les rassemblent. Un tableau de bord est une collection d’**onglets** nommée et étendue au client. Chaque onglet est une grille de 12 colonnes de **widgets**.

| Concept | Signification |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Tableau de bord** | Collection nommée de 1 à 40 onglets, stockée côté serveur pour le client sélectionné. |
| **Onglet** | Une grille de widgets (jusqu’à 200). Renommez-le avec le crayon, supprimez-le avec ×, ajoutez-en un avec **+**. Il doit rester au moins un onglet. |
| **Widget** | **requête unique**, **formule** ou **texte** (en-tête de section réductible ou de note). |
| **Variable** | Un sélecteur au-dessus de la grille. Sa valeur est injectée en tant que `{{name}}`. |
| **Sélecteur d’heure** | La fenêtre pour chaque widget du tableau de bord. |
