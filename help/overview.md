---
title: Surveiller votre environnement AEM Managed Services à l’aide d’Observability Insights
description: Commencez ici pour découvrir ce que couvre Observability Insights dans AEM Managed Services, à qui il s’adresse et comment naviguer dans le reste de ce guide.
feature: Operations
role: Admin
source-git-commit: 440f182902d797a91b584fe1bac7f2b417f30ebe
workflow-type: tm+mt
source-wordcount: '733'
ht-degree: 0%

---


# Surveiller votre environnement AEM Managed Services à l’aide d’Observability Insights {#observability-insights-monitoring}

**Observability Insights** offre une visibilité sur les performances des applications, l’intégrité de l’infrastructure et le comportement des services dans AEM Managed Services, sans nécessiter de plateforme de surveillance distincte.

Si vous êtes responsable de la fiabilité du service, de la réponse aux incidents ou de l’analyse des performances, **Observability Insights** vous aide à passer rapidement des symptômes aux preuves. Elle associe la télémétrie des applications et les signaux d’intégrité au niveau de l’hôte afin que les équipes clientes et les équipes Adobe puissent étudier les problèmes d’un point de vue opérationnel partagé.

## Pourquoi les équipes utilisent-elles Observability Insights ? {#why-teams-use-observability-insights}

Utilisez les insights d’observabilité pour répondre à des questions opérationnelles telles que :

- Le problème affecte-t-il l’auteur, la publication ou les deux ?
- Le problème est-il dû au comportement de l’application, à la pression des ressources de l’hôte ou à une combinaison des deux ?
- Quels sont les transactions, les points d’entrée ou les groupes de statuts qui expliquent le pic d’erreurs ou de latence ?
- Le problème est-il isolé dans un environnement ou visible dans l’ensemble de la topologie ?

Observability Insights est conçu pour l’analyse opérationnelle des comportements récents. Il vous permet d’identifier les éléments qui ont changé, les endroits où ils ont changé et les signaux les plus pertinents avant l’escalade ou l’action corrective.

## Qu’est-ce qu’Observability Insights peut vous apporter ? {#what-observability-insights-helps-you-do}

Utilisez les insights d’observabilité pour :

- Comprendre comment les niveaux de création et de publication se comportent en trafic réel.
- Associez la latence de l’application, les taux d’erreur et l’intégrité de la JVM aux signaux au niveau de l’hôte.
- Confirmez si un problème est isolé à un environnement, un niveau ou un hôte.
- Donnez à Adobe Managed Services et à vos équipes internes une vue opérationnelle partagée pendant l’enquête.

Observability Insights est inclus dans AEM Managed Services. Adobe approvisionne et gère le compte, instrumente les environnements pris en charge et expose les tableaux de bord résultants à votre équipe en tant qu’outils opérationnels en lecture seule.

Comme Adobe gère la configuration et l’instrumentation de la plateforme, vous pouvez vous concentrer sur l’investigation et l’interprétation plutôt que sur le déploiement de l’agent, l’administration de compte ou l’assemblage des tableaux de bord.

## En un coup d’œil {#at-a-glance}

Dans le cadre d’AEM Managed Services, vous recevez :

- **Compte Observability Insights dédié** — Fourni et supervisé par Adobe Managed Services, avec accès en lecture seule à votre équipe.
- **Surveillance approfondie des transactions AEM** — L’agent APM Observability Insights effectue le suivi des transactions significatives jusqu’aux appels de méthode (y compris les numéros de ligne), aux dépendances externes et aux opérations de référentiel.
- **Vue unifiée des applications et des hôtes** — Combinez les applications et les mesures au niveau de l’hôte pour optimiser les performances de manière holistique.

## À qui s’adresse cette documentation {#who-this-documentation-is-for}

Cette documentation est principalement conçue pour les éléments suivants :

- Administrateurs AEM Managed Services qui ont besoin de visibilité sur les environnements surveillés
- Les équipes opérationnelles et de support gèrent les incidents, l’analyse des tendances et la révision des services
- Équipes d’ingénieurs client travaillant en partenariat avec Adobe Managed Services pendant les investigations
- Parties prenantes qui doivent comprendre la portée du suivi et les responsabilités opérationnelles

## Ce que surveille Adobe avec Observability Insights {#what-we-monitor}

Adobe surveille les niveaux AEM **création** et **publication** à l’aide du plug-in Java APM Observability Insights. Tous les serveurs hébergés dans votre topologie sont surveillés par l’agent d’infrastructure Observability Insights. La surveillance personnalisée de l’APM et de l’infrastructure est activée dans les environnements Managed Services de production et hors production.

![Diagramme présentant la surveillance de l’APM et de l’infrastructure d’Observability Insights sur les serveurs de création, de publication et hébergés d’AEM](v2-assets/login-screen.png)

### Applications dans votre compte {#applications-in-your-account}

Votre compte Observability Insights est lié à un compte principal Adobe unique et peut recevoir des données de plusieurs applications, notamment :

- Une application APM pour le niveau **Auteur** par environnement AEM Managed Services.
- Une application APM pour le niveau **Publication** par environnement AEM Managed Services

Chaque application possède sa propre clé de licence. Toutes les topologies de votre contrat Managed Services sont regroupées dans un seul compte Observability Insights. Les mesures et événements APM et Infrastructure sont conservés pendant 30 **maximum**.

## Accéder à votre compte {#access}

Les données de surveillance sont consolidées dans un compte Observability Insights qu’Adobe approvisionne et gère. Les utilisateurs clients reçoivent **accès en lecture seule** aux données d’APM et d’infrastructure collectées par les agents. Adobe Managed Services conserve la propriété du compte et le contrôle administratif.

### Prérequis {#access-prerequisites}

Avant de vous connecter, vérifiez les points suivants :

- Votre entreprise dispose d’un abonnement à **AEM Managed Services** actif. Observability Insights est inclus sans frais supplémentaires.
- Votre ingénieur du succès client (CSE) a provisionné votre compte Adobe IMS et vous a accordé l’accès au compte Observability Insights pour votre organisation.

>[!NOTE]
>
> **Obtention de l’accès :** l’accès à Observability Insights nécessite l’approvisionnement d’Adobe IMS. Contactez l’ingénieur du succès client (CSE) pour configurer et gérer l’accès des utilisateurs pour votre entreprise.

Une fois que le CSE a configuré le compte, connectez-vous à l’adresse [insights.adobecqms.net](https://insights.adobecqms.net). Cette URL est identique pour tous les clients AEM Managed Services ; les environnements et tableaux de bord de votre organisation sont limités à votre compte configuré.
