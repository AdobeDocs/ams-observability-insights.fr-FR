---
title: Couverture, environnements et conservation des données
description: Découvrez ce qu’Observability Insights surveille dans AEM Managed Services, comment les applications sont représentées et combien de temps les données de surveillance sont conservées.
feature: Operations
role: Admin
product_v2:
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
    internal-label: Experience Manager
feature_v2:
  - id: 8a70d214-ab7b-58c1-b001-2ed2e5d6303d
    internal-label: Operations
role_v2:
  - id: c66ffd68-0f65-42bb-aa23-b4020f12e0bd
    internal-label: Admin
source-git-commit: e0cc17c9d725cad021ba99da4332bca176eae6db
workflow-type: tm+mt
source-wordcount: '267'
ht-degree: 2%
---

# Couverture, environnements et conservation des données {#coverage-environments-and-data-retention}

Cette page résume les données collectées dans Observability Insights pour AEM Managed Services et la manière dont ces données sont organisées.

## Surveillance de la couverture {#monitoring-coverage}

Moniteurs Adobe :

- Niveaux de création AEM avec le plug-in Java APM Observability Insights
- Niveaux de publication AEM avec le plug-in Java APM Observability Insights
- Serveurs hébergés dans la topologie gérée avec l’agent Observability Insights Infrastructure

La surveillance personnalisée de l’APM et de l’infrastructure est activée dans les environnements Managed Services de production et hors production.

## Représentation des applications {#how-applications-are-represented}

Chaque environnement AEM Managed Services comprend généralement les éléments suivants :

- Une application APM pour l’auteur
- Une application APM pour la publication

Toutes les topologies d’un contrat Managed Services sont regroupées dans un seul compte Observability Insights.

## Conservation des données {#data-retention}

Les mesures APM, les mesures d’infrastructure et les événements associés sont conservés pendant 30 **maximum**.

## Tableaux récapitulatifs {#summary-tables}

| Zone de couverture | Éléments surveillés |
| -------------- | ------------------------------------------ |
| APM | Applications de création et de publication AEM |
| Infrastructure | Tous les serveurs hébergés dans la topologie gérée |

| Élément | Représentation |
| ------------------------------ | ------------------------------------------------------------- |
| Environnement AEM | Une application APM de création et une application APM de publication |
| Compte Observability Insights | Un compte géré par Adobe par étendue de client Managed Services |

| Type de données | Rétention |
| --------------------------------- | ------------- |
| Mesures et événements APM | Jusqu’à 30 jours |
| Mesures et événements relatifs à l’infrastructure | Jusqu’à 30 jours |

## Ce que cela signifie sur le plan opérationnel {#what-this-means-operationally}

- Observability Insights est adapté à l’analyse opérationnelle, aux incidents actifs et à la comparaison des tendances récentes.
- L’analyse historique au-delà de la période de conservation doit être traitée par d’autres processus de création de rapports ou d’archivage si nécessaire.
- Lors de l’examen de problèmes récurrents, capturez des captures d’écran ou des preuves exportées avant que les données ne vieillissent.
