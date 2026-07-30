---
title: Surveiller votre environnement AEM Managed Services à l’aide de Synoptryx
description: 'Présentation de la surveillance Synoptryx sur Adobe Experience Manager Managed Services : ce qu’Adobe surveille, comment votre compte est configuré et comment votre équipe obtient l’accès.'
feature: Operations
role: Admin
source-git-commit: f937aa4e3cebd1aae6945a35a77154add5db980c
workflow-type: tm+mt
source-wordcount: '651'
ht-degree: 0%

---


# Surveiller votre environnement AEM Managed Services à l’aide de Synoptryx {#synoptryx-monitoring}

Synoptryx offre à votre équipe une visibilité sur les performances des applications, l&#39;intégrité de l&#39;infrastructure et l&#39;expérience de l&#39;utilisateur final, sans configurer de plateforme de surveillance distincte.

>[!NOTE]
>
> Un livre blanc de présentation du produit Synoptryx est disponible pour une présentation complète de l’observabilité et de la surveillance d’AEM Managed Services. Il est idéal pour le partage avec les parties prenantes ou l’examen hors ligne.

## Vue d’ensemble {#overview}

Synoptryx est une plateforme d’observabilité de nouvelle génération d’Adobe conçue pour offrir une visibilité unifiée sur les performances des applications, l’intégrité des infrastructures et la surveillance synthétique. Il permet une surveillance proactive des services critiques de l’entreprise grâce à une expérience unique et intégrée. Synoptryx associe la surveillance des performances des applications (APM), la surveillance de l&#39;infrastructure et la surveillance du Parcours d&#39;utilisateur synthétique pour aider à identifier et à résoudre les problèmes avant qu&#39;ils n&#39;affectent les utilisateurs finaux. La plateforme fournit un suivi détaillé des transactions, des informations JVM, une télémétrie de l’infrastructure et des diagnostics avancés pour une analyse plus rapide des causes premières. Basé sur des technologies d’observabilité modernes, il offre une surveillance évolutive et sécurisée sur des environnements d’entreprise complexes. Synoptryx offre une rétention de données étendue, des tableaux de bord riches et des analyses intelligentes pour soutenir l&#39;excellence opérationnelle. Une expérience de connexion transparente avec Adobe IMS garantit un accès et une gouvernance sécurisés. La plateforme est conçue pour améliorer la fiabilité du service, accélérer le dépannage et améliorer l’expérience client. En tant que solution d’observabilité stratégique d’Adobe, Synoptryx fournit une base évolutive pour la surveillance, l’automatisation et les informations opérationnelles des environnements de services gérés.

Synoptryx est inclus dans Adobe Experience Manager Managed Services ; aucune plateforme de surveillance distincte ni licence n&#39;est requise. Adobe surveille la disponibilité et les performances de votre environnement dans le cadre de notre offre standard. Synoptryx est la plateforme dédiée que votre équipe peut utiliser pour évaluer les performances de votre application Adobe Experience Manager (AEM) et de votre infrastructure de support.

Ce guide explique ce qui est surveillé, comment votre compte Synoptryx est configuré et comment naviguer dans les tableaux de bord que vous utilisez pour l&#39;analyse et le dépannage quotidiens.

## En un coup d’œil {#at-a-glance}

Dans le cadre d’AEM Managed Services, vous recevez :

- **Compte Synoptryx dédié** — Fourni et supervisé par Adobe Managed Services, avec un accès en lecture seule pour votre équipe.
- **Surveillance approfondie des transactions AEM** — L&#39;agent Synoptryx APM suit les transactions significatives jusqu&#39;aux appels de méthode (y compris les numéros de ligne), aux dépendances externes et aux opérations de référentiel.
- **Vue unifiée de l’application et de l’infrastructure** — Combinez les mesures au niveau de l’APM et de l’hôte pour optimiser les performances de manière holistique.

## Ce que Adobe surveille avec Synoptryx {#what-we-monitor}

Adobe surveille les niveaux AEM **auteur** et **publication** avec le plug-in Java Synoptryx APM. Tous les serveurs hébergés de votre topologie sont surveillés par l’agent d’infrastructure Synoptryx. La surveillance personnalisée de l’APM et de l’infrastructure est activée dans les environnements Managed Services de production et hors production.

![Diagramme présentant la surveillance Synoptryx APM et de l’infrastructure sur les serveurs AEM de création, de publication et hébergés](assets/image6.png)

### Applications dans votre compte {#applications-in-your-account}

Votre compte Synoptryx est lié à un compte maître Adobe unique et peut recevoir des données de plusieurs applications, notamment :

- Une application APM pour le niveau **Auteur** par environnement AEM Managed Services.
- Une application APM pour le niveau **Publication** par environnement AEM Managed Services

Chaque application possède sa propre clé de licence. Toutes les topologies de votre contrat Managed Services sont regroupées dans un seul compte Synoptryx. Les mesures et événements APM et Infrastructure sont conservés pendant 30 **maximum**.

## Accès et votre compte {#access}

Les données de surveillance sont consolidées dans un compte Synoptryx qu’Adobe approvisionne et gère. Votre équipe reçoit **accès complet en lecture seule** à toutes les mesures d’APM et d’infrastructure collectées par les agents. Adobe Managed Services conserve la propriété et le contrôle administratif du compte.

>[!NOTE]
>
> **Obtention de l’accès :** l’accès à Synoptryx nécessite l’approvisionnement Adobe IMS. Votre ingénieur du succès client (CSE) peut configurer et gérer l’accès des utilisateurs pour votre entreprise.

Une fois le compte configuré par le CSE, vous pouvez vous connecter à l’adresse [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net).

## Prochaines étapes {#whats-next}

Continuez avec les tableaux de bord de surveillance que votre équipe utilise quotidiennement :

- [Surveillance des performances des applications (APM)](application-performance-monitoring.md) — Effectuez le suivi des transactions AEM, analysez le comportement JVM et inspectez les services externes.
- [Surveillance de l’infrastructure](infrastructure-monitoring.md) — Examinez les métriques relatives au système, au réseau, aux processus et au stockage au niveau de l’hôte.
