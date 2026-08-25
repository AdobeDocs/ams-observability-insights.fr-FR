---
title: Utiliser les informations d’observabilité
description: Découvrez les quatre principales expériences de surveillance et d’enquête dans Observability Insights et quand les utiliser.
feature: Operations
role: Admin
source-git-commit: 6bbc906fa1c5570bc7ee2a6f536dd806c0c0db41
workflow-type: tm+mt
source-wordcount: '426'
ht-degree: 0%

---


# Utiliser les informations d’observabilité {#use-observability-insights}

Cette section couvre les workflows de surveillance et d’investigation quotidiens les plus utilisés par votre équipe. Ils sont organisés autour de deux domaines de surveillance : la surveillance des performances des applications et la surveillance de l’infrastructure.

## Interface Observability Insights {#observability-insights-interface}

Le panneau de navigation de gauche Observability Insights vous donne accès à toutes les zones de surveillance de vos environnements AEM Managed Services.

![Interface Observability Insights présentant une navigation à gauche avec les options APM et Infrastructure, et le tableau de bord de surveillance des infrastructures avec les mesures des hôtes et les filtres d’environnement](v2-assets/navigation-panel-desc.png)

La navigation comprend les éléments suivants :

- **Catalogue** — Inventaire central des applications et hôtes AEM surveillés. Parcourez les ressources sur les niveaux **Auteur, Publication et Dispatcher**, avec des indicateurs d’intégrité et de performances clés tels que le temps de réponse, le débit, le taux d’erreur et Apdex en un coup d’œil.

- **Explorer** — Étudier la télémétrie d&#39;observabilité et explorer les données de performances sur les ressources surveillées.

- **Traces** — Analysez les transactions d&#39;application de bout en bout et demandez des traces pour identifier la latence, les erreurs et les goulots d&#39;étranglement en termes de performances.

- **Tableaux de bord** — Accédez à des tableaux de bord sélectionnés pour une visualisation et une surveillance plus approfondies des signaux des applications et des infrastructures.

Les ressources peuvent être filtrées par compte et par niveau, tandis que le catalogue fournit une vue consolidée de l’intégrité de l’application et de l’hôte dans la topologie AEM gérée.

## Applications{#applications}

Utilisez [Applications](applications.md) lorsque le problème est lié à l’application : pages lentes, taux d’erreur croissants, transactions instables ou latence inattendue sur l’instance de création ou de publication.

Les applications vous aident à répondre aux questions suivantes :

- Le problème se limite-t-il à la création, à la publication ou affecte-t-il les deux niveaux ?
- Quels points d’entrée ou transactions contribuent le plus au trafic et aux ralentissements ?
- La latence ou les erreurs ont-elles changé avant ou après un déploiement ou un pic de trafic ?
- Les traces pointent-elles vers des opérations de référentiel, des dépendances externes ou la pression JVM ?

Les applications instrumentent les transactions AEM jusqu’aux appels de méthode, aux dépendances externes et aux opérations de référentiel, afin que vous puissiez passer rapidement d’un symptôme général à un chemin d’exécution spécifique.

## Hôtes {#hosts}

Utilisez [Hôtes](hosts.md) lorsque vous devez déterminer si le comportement de l’application est causé ou aggravé par des conditions de ressources de l’hôte (saturation du CPU, pression de la mémoire, E/S du disque, débit du réseau ou capacité de stockage).

La surveillance de l’hôte permet de répondre aux questions suivantes :

- Le ralentissement de l’application s’accompagne-t-il d’une pression de CPU, de mémoire ou d’E/S au niveau de l’hôte ?
- Un hôte se comporte-t-il différemment des autres dans le même environnement ?
- Les tendances d’utilisation des disques ou du stockage indiquent-elles un problème de capacité à venir ?
- Les modèles d’infrastructure expliquent-ils le comportement de l’application ou sont-ils un effet en aval ?

Utilisez les tableaux de bord de l’hôte avec les applications pour faire la distinction entre les régressions au niveau de l’application et les contraintes de ressources au niveau de l’environnement.

Ces deux articles comprennent des workflows d’enquête, des questions pour vous guider dans le tri et une liste de preuves à capturer lors de la réaffectation à Adobe Managed Services.
