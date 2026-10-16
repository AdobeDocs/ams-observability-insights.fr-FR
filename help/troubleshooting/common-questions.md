---
title: Questions fréquentes
description: Questions courantes et points de départ d’enquête pour Observability Insights dans AEM Managed Services.
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
source-git-commit: ba4f816187e77f3f7b708942b3f14535417c2fdc
workflow-type: tm+mt
source-wordcount: '253'
ht-degree: 0%
---

# Questions fréquentes {#faq}

Utilisez cette page comme point de départ lorsque vous ne savez pas par où commencer ou que vous avez besoin d&#39;une réponse rapide lors d&#39;une enquête active.

## Pourquoi ne puis-je pas accéder à Observability Insights ? {#cannot-access-observability-insights}

Commencez par [Gestion des accès et des comptes](../get-started/access-and-accounts.md). Si votre mise en service est incomplète ou obsolète, contactez votre ingénieur du succès client (CSE) pour demander un accès ou une mise à jour.

## Pourquoi « Chargement des autorisations » s’affiche-t-il lorsque j’essaie de me connecter ? {#loading-permissions-error}

Cela indique généralement un problème d’approvisionnement des utilisateurs et utilisatrices. Contactez votre ingénieur du succès client (CSE), qui peut travailler avec les équipes concernées pour résoudre le problème d’accès.

## Comment puis-je déterminer si un problème est lié à une application ou à une infrastructure ? {#application-or-infrastructure}

Commencez par [Surveillance des performances des applications](/help/applications.md) pour consulter les taux de requête, les taux d’erreur et la latence sur les instances de création ou de publication. Si les signaux de l’application sont élevés, utilisez [Hosts](/help/hosts.md) pour vérifier si la pression des ressources au niveau de l’hôte (CPU, mémoire, disque ou réseau) explique ou aggrave ce que vous voyez.

## Quelles données Observability Insights collecte-t-il réellement ? {#what-data-is-collected}

Consultez [Couverture, environnements et conservation des données](../get-started/coverage-and-data.md) pour connaître la portée de la surveillance, la représentation des applications, les périodes de conservation et les implications opérationnelles.

## Pourquoi puis-je me connecter avec d’autres navigateurs mais pas avec Google Chrome ?

Pour autoriser les cookies tiers à utiliser Observability Insights et réessayer de vous connecter :

1. Pour accéder aux paramètres des cookies, ouvrez `chrome://settings/cookies` dans Google Chrome.
2. Pour autoriser les cookies pour les insights d’observabilité, sous **Sites autorisés à utiliser des cookies tiers**, sélectionnez **Ajouter** et saisissez les `https://insights.adobecqms.net`.
3. Pour vérifier si le problème est résolu, connectez-vous à Observability Insights dans une fenêtre en mode privé.