---
title: Applications
description: Les applications fournissent des fonctionnalités de surveillance des performances des applications (APM), offrant une vue unifiée de l’intégrité des applications, des performances, des transactions et de l’infrastructure sous-jacente prenant en charge chaque service.
feature: Operations
role: Admin
source-git-commit: efddec659ebb1cdd22537d60ccca175680dfdab4
workflow-type: tm+mt
source-wordcount: '546'
ht-degree: 0%

---


# Applications

Les applications fournissent des fonctionnalités de surveillance des performances des applications (APM), offrant une vue unifiée de l’intégrité des applications, des performances, des transactions et de l’infrastructure sous-jacente prenant en charge chaque service. Il permet aux équipes opérationnelles et d’ingénierie de comprendre le comportement des applications, d’identifier les goulots d’étranglement en termes de performances et de passer d’indicateurs de santé généraux à des transactions individuelles pour une enquête plus approfondie.

## Résumé de l&#39;application

Le résumé **applications** offre une vue d’ensemble de l’application sélectionnée. Les indicateurs clés tels que la latence p95, le débit du serveur, le taux d’erreur et Apdex permettent d’évaluer facilement l’intégrité de l’application au cours de la période sélectionnée.

Les filtres pour le type de transaction, l&#39;hôte et la résolution permettent d&#39;affiner la vue pour une investigation spécifique. Les tendances de temps de réponse et de débit fournissent un contexte supplémentaire, ce qui aide les équipes à distinguer les pics isolés des changements de performances prolongés.

![résumé des applications](v2-assets/1_apm-services-landing-page.png)

## Temps de réponse, débit et index

Les performances des applications peuvent être évaluées à l’aide des temps de réponse centiles parallèlement au débit des requêtes. L’affichage de la latence p50, p95 et p99 permet de distinguer les expériences utilisateur standard des valeurs aberrantes plus lentes.

Apdex fournit une mesure complémentaire de la réactivité de l’application en traduisant les performances du temps de réponse en un score de satisfaction facile à comprendre. Ces mesures, ainsi que le taux d’erreur, indiquent de manière concise si une application fonctionne avec les niveaux de performances attendus.

![Temps de réponse, débit et Apdex](v2-assets/2_apm-summary-apdex.png)

## Erreurs et transactions lentes

Les applications font continuellement apparaître les tendances de taux d&#39;erreur et les transactions lentes pour aider à identifier les demandes qui peuvent affecter les performances des applications. La vue Taux d’erreur permet de reconnaître facilement les modifications au fil du temps, tandis que la tendance Apdex indique l’impact correspondant sur la réactivité de l’application.

La vue **Transactions les plus lentes** met en évidence les transactions avec la durée moyenne la plus élevée et inclut le volume d’appels, ce qui facilite la distinction entre les charges de travail fréquemment exécutées et les requêtes lentes isolées.

![Taux d’erreur, Apdex et transactions les plus lentes](v2-assets/3_error-rate-transactions.png)

## Corrélation entre les transactions et les infrastructures

La liste des transactions fournit une vue ciblée des types de transactions les plus lents, y compris leur trace la plus lente observée, leur taux d’erreur et leur durée moyenne. Cela permet aux équipes d’identifier rapidement les modèles de transaction qui nécessitent une enquête plus approfondie.

Les données des applications sont corrélées avec les hôtes sous-jacents afin que les performances des transactions puissent être évaluées parallèlement à des indicateurs d’infrastructure tels que le temps de réponse, le débit, l’utilisation de CPU et l’utilisation de la mémoire. Cette corrélation permet de déterminer si un problème de performances trouve son origine dans le traitement des applications ou peut être associé à l’infrastructure de prise en charge.

![Corrélation entre les transactions et l’infrastructure](v2-assets/4_transaction-listing.png)

## Analyse des performances des transactions

La vue d’analyse des transactions classe les transactions par caractéristiques de performances et résume les indicateurs clés tels que la transaction la plus longue, le temps de réponse p95 le plus lent, le taux d’erreur le plus élevé, le débit et l’Apdex.

Les visualisations de série temporelle montrent comment les transactions les plus importantes contribuent au temps de traitement global et comment le débit des requêtes change au cours de la période sélectionnée. Cela facilite l’identification des points d’entrée à fort impact, la comparaison du comportement des transactions et la détermination des requêtes qui doivent être examinées en premier.

![Analyse des performances des transactions](v2-assets/5_transaction-graphs.png)

## Enquête sur les problèmes de performances

Les applications prennent en charge un workflow d’investigation progressif : commencez par définir des indicateurs d’intégrité et de performances au niveau de l’application, identifiez les temps de réponse anormaux, les erreurs ou les changements de débit, puis limitez l’investigation aux transactions qui contribuent le plus au problème. Les données de transaction peuvent être corrélées aux mesures d’infrastructure au niveau de l’hôte.

Ce workflow permet aux équipes de s’éloigner efficacement des **tendance de l’intégrité → des performances des applications → des hôtes de → des transactions**, ce qui réduit le temps nécessaire pour isoler la source d’un problème de performances.
