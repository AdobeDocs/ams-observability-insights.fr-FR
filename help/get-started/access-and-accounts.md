---
title: Gestion des accès et des comptes
description: Découvrez comment les comptes Observability Insights sont configurés, qui gère l’accès et quel niveau de contrôle les équipes clientes et clients ont.
feature: Operations
role: Admin
source-git-commit: 6526a90a017147ac3483c0b2b626b9aa903819ba
workflow-type: tm+mt
source-wordcount: '232'
ht-degree: 0%

---


# Gestion des accès et des comptes {#access-and-account-management}

Adobe approvisionne et gère le compte Observability Insights pour votre organisation AEM Managed Services. Les équipes clientes utilisent Adobe IMS pour se connecter et recevoir une visibilité en lecture seule sur les données surveillées.

## Modèle de propriété du compte {#account-ownership-model}

- Adobe Managed Services possède le compte Observability Insights.
- Les équipes clientes bénéficient d’un accès en lecture seule.
- Les modifications administratives, la mise en service et les mises à jour des accès sont gérés via Adobe.

## Accès des utilisateurs et utilisatrices {#how-users-get-access}

L’accès à Observability Insights nécessite la mise en service d’Adobe IMS.

Pour demander ou mettre à jour l’accès :

1. Contactez votre ingénieur du succès client (CSE).
2. Fournissez les détails utilisateur requis pour l’approvisionnement d’Adobe IMS.
3. Vérifiez que l’organisation et la portée d’accès appropriées ont été attribuées.

Une fois l’approvisionnement terminé, connectez-vous à l’adresse [insights.adobecqms.net](https://insights.adobecqms.net).

## Ce que les utilisateurs peuvent faire {#what-users-can-do}

Les utilisateurs peuvent généralement :

- Afficher les tableaux de bord APM
- Affichage des tableaux de bord de l’infrastructure
- Inspecter l’application surveillée et les mesures de l’hôte
- Participer aux investigations en utilisant des traces et des tableaux de bord partagés

## Ce que les utilisateurs ne peuvent pas faire {#what-users-cannot-do}

Les utilisateurs doivent supposer que le contrôle administratif reste entre les mains d’Adobe Managed Services, sauf si Adobe en documente explicitement le contraire.

Voici quelques exemples courants :

- Gestion de la propriété du compte
- Modification de l’approvisionnement au niveau de la plateforme
- Modification du comportement de l’instrumentation gérée

## Informations à ajouter ultérieurement {#information-to-add-later}

Utilisez cette section lorsque des détails de processus plus complets sont disponibles :

- Conditions préalables d’Adobe IMS
- Délais de provisionnement prévus
- Contacts et chemins d’escalade
- Gestion du cycle de vie des utilisateurs pour les rejoignants et les sortants
