---
title: Surveillez votre environnement AEM Managed Services avec  [!DNL Synoptryx]
description: 'Présentation  [!DNL Synoptryx]  la surveillance sur Adobe [!DNL Experience Manager] Managed Services : ce qu’Adobe surveille, comment votre compte est configuré et comment votre équipe obtient l’accès.'
feature: Operations
role: Admin
source-git-commit: c79ae46b8ab4f6aab02821bc4446e04a94670aef
workflow-type: tm+mt
source-wordcount: '618'
ht-degree: 0%

---


# Surveillez votre environnement AEM Managed Services avec [!DNL Synoptryx] {#synoptryx-monitoring}

[!DNL Synoptryx] offre à votre équipe une visibilité sur les performances des applications, l’intégrité de l’infrastructure et l’expérience de l’utilisateur final, sans avoir à configurer de plateforme de surveillance distincte.

>[!NOTE]
>
> Un livre blanc de présentation du produit [!DNL Synoptryx] est disponible pour une présentation complète de l’observabilité et de la surveillance d’AEM Managed Services. Il est idéal pour le partage avec les parties prenantes ou l’examen hors ligne.

## Vue d’ensemble {#overview}

[!DNL Synoptryx] est une plateforme d’observabilité nouvelle génération d’Adobe conçue pour offrir une visibilité unifiée sur les performances des applications, l’intégrité de l’infrastructure et la surveillance synthétique. Il permet une surveillance proactive des services critiques de l’entreprise grâce à une expérience unique et intégrée. [!DNL Synoptryx] associe la surveillance des performances des applications (APM), la surveillance de l’infrastructure et la surveillance du Parcours d’utilisateurs synthétiques pour aider à identifier et à résoudre les problèmes avant qu’ils n’affectent les utilisateurs finaux. La plateforme fournit un suivi détaillé des transactions, des informations JVM, une télémétrie de l’infrastructure et des diagnostics avancés pour une analyse plus rapide des causes premières. Basé sur des technologies d’observabilité modernes, il offre une surveillance évolutive et sécurisée sur des environnements d’entreprise complexes. [!DNL Synoptryx] offre une rétention des données étendue, des tableaux de bord riches et des analyses intelligentes pour soutenir l&#39;excellence opérationnelle. Une expérience de connexion transparente avec [!DNL Adobe IMS] garantit un accès et une gouvernance sécurisés. La plateforme est conçue pour améliorer la fiabilité du service, accélérer le dépannage et améliorer l’expérience client. En tant que solution d’observabilité stratégique d’Adobe, [!DNL Synoptryx] fournit une base évolutive pour la surveillance, l’automatisation et les informations opérationnelles des environnements de services gérés.

[!DNL Synoptryx] est inclus dans Adobe [!DNL Experience Manager] Managed Services ; aucune plateforme de surveillance distincte ni licence n’est requise. Adobe surveille la disponibilité et les performances de votre environnement dans le cadre de notre offre standard. Il s’[!DNL Synoptryx] de la plateforme dédiée que votre équipe peut utiliser pour évaluer les performances de votre application Adobe [!DNL Experience Manager] (AEM) et de votre infrastructure de support.

Ce guide explique les éléments surveillés, la configuration de votre compte [!DNL Synoptryx] et la navigation dans les tableaux de bord que vous utilisez pour les analyses et la résolution des problèmes quotidiens.

## En un coup d’œil {#at-a-glance}

Dans le cadre d’AEM Managed Services, vous recevez :

- **Compte [!DNL Synoptryx] dédié** — Fourni et supervisé par Adobe Managed Services, avec un accès en lecture seule pour votre équipe.
- **Surveillance approfondie des transactions AEM** — L&#39;agent [!DNL Synoptryx] APM retrace les transactions significatives jusqu&#39;aux appels de méthode (y compris les numéros de ligne), aux dépendances externes et aux opérations de référentiel.
- **Vue unifiée de l’application et de l’infrastructure** — Combinez les mesures au niveau de l’APM et de l’hôte pour optimiser les performances de manière holistique.

## Ce qu’Adobe surveille avec [!DNL Synoptryx] {#what-we-monitor}

Adobe surveille les niveaux AEM **création** et **publication** à l’aide du plug-in Java APM [!DNL Synoptryx]. Tous les serveurs hébergés de votre topologie sont surveillés par l’agent d’infrastructure [!DNL Synoptryx]. La surveillance personnalisée de l’APM et de l’infrastructure est activée dans les environnements Managed Services de production et hors production.

![Diagramme présentant la surveillance Synoptryx APM et de l’infrastructure sur les serveurs AEM de création, de publication et hébergés](assets/image6.png)

### Applications dans votre compte {#applications-in-your-account}

Votre compte [!DNL Synoptryx] est lié à un compte principal Adobe unique et peut recevoir des données de plusieurs applications, notamment :

- Une application APM pour le niveau **Auteur** par environnement AEM Managed Services.
- Une application APM pour le niveau **Publication** par environnement AEM Managed Services

Chaque application possède sa propre clé de licence. Toutes les topologies de votre contrat Managed Services sont regroupées dans un seul compte [!DNL Synoptryx]. Les mesures et événements APM et Infrastructure sont conservés pendant 30 **maximum**.

## Accès et votre compte {#access}

Les données de surveillance sont consolidées dans un compte [!DNL Synoptryx] qu’Adobe approvisionne et gère. Votre équipe reçoit **accès complet en lecture seule** à toutes les mesures d’APM et d’infrastructure collectées par les agents. Adobe Managed Services conserve la propriété et le contrôle administratif du compte.

>[!NOTE]
>
> **Obtention de l’accès :** l’accès à [!DNL Synoptryx] nécessite un approvisionnement [!DNL Adobe IMS]. Votre ingénieur du succès client (CSE) peut configurer et gérer l’accès des utilisateurs pour votre entreprise.

Une fois le compte configuré par le CSE, vous pouvez vous connecter à l’adresse [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net).

## Prochaines étapes {#whats-next}

Continuez avec les tableaux de bord de surveillance que votre équipe utilise quotidiennement :

- [Surveillance des performances des applications (APM)](application-performance-monitoring.md) — Effectuez le suivi des transactions AEM, analysez le comportement JVM et inspectez les services externes.
- [Surveillance de l’infrastructure](infrastructure-monitoring.md) — Examinez les métriques relatives au système, au réseau, aux processus et au stockage au niveau de l’hôte.
