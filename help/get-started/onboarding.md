---
title: Prise en main d’Observability Insights
description: Découvrez comment accéder à Observability Insights, ce qu’Adobe surveille en votre nom et où trouver ce dont vous avez besoin dans ce guide.
feature: Operations
role: Admin
source-git-commit: cc405e8b70973c33ecc6137114315998e8f9af50
workflow-type: tm+mt
source-wordcount: '533'
ht-degree: 0%

---


# Prise en main d’Observability Insights {#get-started}

Cette section couvre les éléments essentiels pour les nouveaux utilisateurs : comment accéder à votre compte Observability Insights, quels environnements et quelles données Adobe surveille en votre nom et comment naviguer dans le reste de cette documentation.

## Interface Observability Insights {#observability-insights-interface}

Lorsque vous vous connectez à l’adresse [insights.adobecqms.net](https://insights.adobecqms.net), l’écran d’ouverture vous donne un point d’entrée dans toutes les zones de surveillance de vos environnements AEM Managed Services.

![Écran d’ouverture d’Observability Insights présentant les points d’entrée de surveillance APM et Infrastructure](../v2-assets/observability-catalog-listing.png)

L’interface est organisée autour de deux zones de surveillance principales :

- **Applications** : affiche les données de performances des applications pour les niveaux de création et de publication. Utilisez-le pour examiner le débit des requêtes, les taux d’erreur, la latence, le comportement de JVM et les détails d’exécution au niveau de la trace. Voir [Applications](../applications.md).
- **Hôtes** — Affiche les données d&#39;intégrité au niveau de l&#39;hôte dans l&#39;ensemble de la topologie gérée. Utilisez-le pour évaluer les signaux CPU, mémoire, disque, réseau et de stockage sur des serveurs individuels. Voir [Hôtes](../hosts.md).

Les deux zones sont en lecture seule pour les utilisateurs et utilisatrices clients. Adobe Managed Services gère l’approvisionnement des comptes, l’instrumentation et le contrôle administratif.

## Gestion des accès et des comptes {#access-overview}

L’accès à Observability Insights est géré via Adobe IMS. Adobe approvisionne et gère le compte de votre entreprise. Les équipes clients reçoivent un accès en lecture seule à toutes les données surveillées.

Points clés :

- Le compte Observability Insights de votre organisation est lié à un seul compte principal Adobe.
- Tous les environnements de votre contrat Managed Services (auteur et publication, production et hors production) font rapport sur ce compte.
- L’accès utilisateur est configuré et géré par votre ingénieur du succès client (CSE).

Pour connaître les étapes d’attribution de privilèges d’accès, les rôles utilisateur et ce que les utilisateurs clients peuvent et ne peuvent pas faire, voir [Gestion des accès et des comptes](access-and-accounts.md).

## Couverture, environnements et conservation des données {#coverage-overview}

Adobe surveille vos niveaux de création et de publication AEM à l’aide du plug-in Java Observability Insights APM et tous les serveurs hébergés à l’aide de l’agent d’infrastructure Observability Insights. La surveillance est activée dans les environnements de production et hors production.

Points clés :

- Chaque environnement AEM Managed Services comprend une application APM pour la création et une pour la publication.
- Les mesures APM, les mesures d’infrastructure et les événements sont conservés pendant 30 **maximum**.
- Observability Insights est adapté à l’analyse opérationnelle et à la comparaison récente des tendances. Il ne s’agit pas d’un outil d’archivage ou de création de rapports à long terme. Capturez des captures d’écran ou des preuves exportées avant que les données ne vieillissent.

Pour plus d’informations sur la couverture complète, notamment sur la représentation des applications dans votre compte et les implications opérationnelles de la période de conservation, reportez-vous à la section [Couverture, environnements et conservation des données](coverage-and-data.md).

## Structure de ce guide {#how-this-guide-is-structured}

La documentation est organisée en quatre domaines. Utilisez les descriptions ci-dessous pour accéder directement à ce dont vous avez besoin.

**Commencer** — Cette section. Couvre l’accès, le provisionnement des comptes, la portée de la surveillance et la conservation des données.

**[Utiliser les informations d’observabilité](../use-observability-insights.md)** — Conseils axés sur les tâches pour les enquêtes quotidiennes. Utilisez [Applications](../applications.md) lorsque le symptôme est visible par l’application : pages lentes, pics d’erreur ou transactions instables. Utilisez [Hôtes](../hosts.md) lorsque vous devez déterminer si la pression des ressources au niveau de l’hôte (CPU, mémoire, disque ou réseau) explique ce que vous voyez dans les applications. Des flux d’enquête détaillés sont disponibles dans [Enquête sur les problèmes d’application](../use-cases/investigate-application-issues.md) et [Enquête sur les problèmes d’infrastructure](../use-cases/investigate-infrastructure-issues.md).

**[Questions fréquentes](../troubleshooting/common-questions.md)** — Questions courantes et points d&#39;entrée orientés support pour les cas où vous ne savez pas par où commencer ou avez besoin de réponses rapides lors d&#39;un incident actif.
