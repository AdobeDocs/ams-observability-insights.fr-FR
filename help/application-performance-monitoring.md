---
title: Surveillance des performances des applications (APM) avec  [!DNL Synoptryx]
description: Utilisez le plug [!DNL Synoptryx] in APM pour suivre les transactions AEM, surveiller la JVM, analyser les transactions et inspecter les traces de transaction et les services externes sur AEM Managed Services.
feature: Operations
role: Admin
source-git-commit: 12876ba185fd6d155f02639fba9601a3616c7e90
workflow-type: tm+mt
source-wordcount: '1078'
ht-degree: 5%

---


# Surveillance des performances des applications (APM) avec [!DNL Synoptryx] {#application-performance-monitoring}

La surveillance des performances des applications [!DNL Synoptryx] (APM) fournit des données historiques et en temps réel d’insight sur les performances d’Adobe [!DNL Experience Manager] (AEM) et l’expérience de l’utilisateur final. Le suivi des transactions de bout en bout, les graphiques et les rapports offrent une visibilité sur le comportement de l’application jusqu’au niveau du code Java.

## Module externe Managed Services [!DNL Synoptryx] APM {#apm-plugin}

AEM s’exécute en tant qu’application Java sur Jetty avec les modules OSGi Apache Felix, basés sur Apache Sling et Jackrabbit Oak. Adobe Managed Services, l’ingénierie AEM et l’ingénierie [!DNL Synoptryx] ont développé conjointement une instrumentation personnalisée pour les environnements Managed Services.

Cette instrumentation collecte :

- **Nommage significatif des transactions** — Les extensions Sling alignent les noms des transactions sur la structure de la page et ajoutent un attribut `requestURL` sur les événements Insights afin que vous puissiez corréler les URL Sling entre les tableaux de bord.

![Vue de trace Synoptryx APM montrant un nom de transaction AEM descriptif avec l’itinéraire de vérification d’intégrité Sling et la chronologie de durée](assets/image19a.png)

- **JCR instrumentation** — Les opérations au niveau du référentiel (y compris XPath et JCR-SQL2) sont classées et associées aux traces de transaction dans la section de base de données d’APM.

![Vue de trace APM Synoptryx montrant les plages de composants AEM imbriqués et la chronologie d’exécution d’une requête de page](assets/image19.png)

## Utilisation d’[!DNL Synoptryx] APM {#using-apm}

Utilisez APM pour identifier les problèmes d’application avant qu’ils n’affectent les utilisateurs finaux. L’auteur et la publication partagent une base de code, mais sont surveillés en tant qu **applications APM distinctes** afin que vous puissiez analyser chaque niveau indépendamment.

Chaque environnement Managed Services comprend :

- Une application APM pour l’auteur
- Une application APM pour la publication

Sélectionnez un nom d’application dans [!DNL Synoptryx]’APM pour ouvrir son tableau de bord de présentation et de surveillance.

![Liste synoptique des applications APM montrant les applications de création et de publication](assets/image1a.png)

## Sections du tableau de bord

Le tableau de bord Gestion des performances des applications contient les sections suivantes :

- Vue d’ensemble
- Mesures RED (Taux · Erreurs · Durée)
- Trafic
- Latence et performances
- Détails de l’erreur
- Principales transactions
- Intégrité JVM
- Mémoire JVM
- Nettoyage de la mémoire

Seules les sections présentées ci-dessous sont documentées dans ce guide.

## Navigation dans le tableau de bord

![Navigation dans le tableau de bord](assets/apm/1_opening_screen.png)

Le tableau de bord est organisé en sections extensibles qui regroupent les mesures de performances des applications associées. Le développement d’une section révèle un ou plusieurs graphiques associés à cette catégorie.

## Vue d’ensemble

![Vue d’ensemble](assets/apm/1.1_apm_overview.png)

### Description

La section **[!UICONTROL Présentation]** présente des indicateurs clés de performance (KPI) de haut niveau qui résument l’état actuel de l’application surveillée.

Ces KPI fournissent un résumé d’un coup d’œil de l’activité de l’application, du débit, du succès de la requête et de l’expérience utilisateur globale.

### Mesures

#### Nombre total de demandes

Affiche le nombre total de demandes traitées par l’application au cours de la période sélectionnée.

**Mesure**

```
total_requests
```

**Unité**

- Décompte

#### Débit actuel

Affiche le taux actuel de traitement des demandes.

**Mesure**

```
throughput
```

**Unité**

- Demandes par seconde (req/s)

#### Taux d’erreur actuel

Affiche le pourcentage de demandes entraînant des erreurs.

**Mesure**

```
error_rate
```

**Unité**

- Pourcentage (%)

#### Score APDEX

Affiche l’indice de performance des applications (APDEX), une mesure normalisée de la satisfaction de l’utilisateur final basée sur les temps de réponse des applications.

Le seuil configuré est affiché dans le widget.

**Mesure**

```
apdex_score
```

**Unité**

- Score (0,0 - 1,0)

## Mesures ROUGES

La méthodologie RED mesure trois caractéristiques principales d&#39;une application :

- **Taux**
- **Erreurs**
- **Durée**

### Taux de demande

![Taux de demande](assets/apm/2_red_metrics_request_rate.png)

#### Description

Affiche le nombre de demandes d&#39;application reçues au fil du temps.

Ce graphique représente le débit des requêtes à l’aide d’une visualisation de série temporelle.

#### Mesure

```
req_min
```

#### Unité

- Demandes par minute (req/m)

#### Informations affichées

- Taux de requête de série temporelle
- Activité de requête historique
- Tendance du taux de demande
- Légende des mesures

### Taux d’erreurs

![Taux d’erreur](assets/apm/3_error_rate.png)

#### Description

Affiche le pourcentage de requêtes ayant entraîné des erreurs.

Le graphique compare les pourcentages d’erreur historiques et actuels.

#### Mesures

```
error_pct (now)
error_pct (1h ago)
```

#### Unité

- Pourcentage (%)

#### Informations affichées

- Pourcentage d’erreur actuel
- Comparaison historique
- Valeurs moyennes
- Tendance de série temporelle

### Durée de la demande

![Durée de la demande](assets/apm/4_request_duration_p50_p95.png)

#### Description

Affiche la latence des requêtes sur plusieurs centiles de temps de réponse.

Le graphique trace simultanément les mesures de latence en centiles collectées pendant la période d’observation sélectionnée.

#### Mesures

```
P50
P75
P90
```

#### Unités

- Millisecondes (ms)
- Seconde (s)

Les unités sont automatiquement mises à l’échelle en fonction de la durée de réponse.

#### Statistiques affichées

Pour chaque centile :

- Moyenne
- Dernière
- Maximum

#### Définitions des centiles

| Mesure | Description |
| ------ | ----------------------------- |
| P50 | Temps de réponse du 50e centile |
| P75 | Temps de réponse du 75e centile |
| P90 | 90e centile du temps de réponse |

## Trafic

### Demandes par code d’état HTTP

![Demandes par code d’état](assets/apm/5_requests_by_status_code.png)

#### Description

Affiche le débit des requêtes, regroupé par code de statut de réponse HTTP.

Chaque code d’état est tracé indépendamment au fil du temps.

#### Mesures

Les mesures courantes sont les suivantes :

```
req_s 200
req_s 300
req_s 400
req_s 500
```

en fonction de l’activité de l’application.

#### Unité

- Demandes par seconde (req/s)

#### Informations affichées

- Débit par statut HTTP
- Débit moyen
- Débit le plus récent
- Débit maximal
- Activité de série temporelle

### Taux de requêtes par point d’entrée

![Taux de requêtes par point d’entrée](assets/apm/6_request_rate_by_end_point.png)

#### Description

Affiche les points d’entrée d’application à trafic le plus élevé classés par taux de requête.

Chaque point d’entrée s’affiche sous la forme d’une barre horizontale représentant le volume des requêtes.

#### Mesure

```
endpoint_request_rate
```

#### Unité

- Demandes par minute (req/m)

#### Informations affichées

- Chemin du point d’entrée
- Taux de demande
- Liste des points d’entrée avec classement
- Volume relatif des requêtes

## Latence et performances

### Temps de réponse : P95 contre 1 heure

![Temps de réponse P95](assets/apm/7_response_time_p95_1h.png)

#### Description

Affiche une comparaison du temps de réponse P95 actuel par rapport au temps de réponse P95 enregistré une heure plus tôt.

Les deux jeux de données s’affichent sur le même graphique de série temporelle.

#### Mesures

```
P95 (Current)
P95 (1 Hour Ago)
```

#### Unités

- Millisecondes (ms)
- Seconde (s)

#### Statistiques affichées

- Moyenne
- Dernière
- Maximum

### Score APDEX au fil du temps

![&#x200B; APDEX &#x200B;](assets/apm/8_apdex_score_overtime.png)

#### Description

Affiche l&#39;indice de performance de l&#39;application sous la forme d&#39;une série temporelle continue.

Le graphique permet de visualiser les valeurs APDEX tout au long de l’intervalle de surveillance sélectionné.

#### Mesure

```
APDEX Score
```

#### Unité

- Score (0.0-1.0)

#### Statistiques affichées

- Moyenne
- Dernière
- Maximum

### Débit vs latence P95

![Débit et latence](assets/apm/9_throughput_vs_p95latency.png)

#### Description

Affiche le débit des requêtes et la latence de réponse P95 sur la même chronologie.

Le graphique permet de visualiser simultanément le volume de trafic et la latence de réponse.

#### Mesures

```
Throughput
P95 Latency
```

#### Unités

| Mesure | Unité |
| ----------- | ------------ |
| Débit | Demandes/s |
| Latence P95 | Millisecondes |

#### Informations affichées

- Débit de la série temporelle
- Latence de série temporelle
- Comparaison des mesures doubles

## Détails de l’erreur

### Taux d&#39;erreurs % par groupe de statuts

![Taux d’erreurs par groupe de statuts](assets/apm/10_error_rate_pct_by_status_group.png)

#### Description

Affiche les pourcentages d’erreur de l’application, regroupés par classe de réponse HTTP.

Des séries distinctes sont tracées pour chaque catégorie de réponse.

#### Mesures

Les groupes courants sont les suivants :

```
2xx
3xx
4xx
5xx
Combined Error Trend
```

en fonction du trafic observé.

#### Unité

- Pourcentage (%)

#### Informations affichées

- Pourcentage d’erreur par classe de réponse
- Pourcentage d’erreur moyen
- Tendance de série temporelle


### Tendance du taux d’erreurs - Maintenant par rapport à il y a 1 heure

![Taux D’Erreurs Sur 1 Heure](assets/apm/11_error_ratio_trend_1h.png)

#### Description

Affiche le taux d&#39;erreurs actuel de l&#39;application avec le taux d&#39;erreurs enregistré une heure plus tôt.

#### Mesures

```
Current Error Ratio
1 Hour Error Ratio
```

#### Unité

- Pourcentage (%)

#### Informations affichées

- Tendance actuelle
- Comparaison historique
- Visualisation de série temporelle

### Tendance du taux d’erreurs - Maintenant par rapport à il y a 6 heures

![Taux d’erreurs : 6 heures](assets/apm/12_error_ratio_trend_6h.png)

#### Description

Affiche le taux d’erreurs actuel de l’application avec le taux d’erreurs enregistré six heures plus tôt.

#### Mesures

```
Current Error Ratio
6 Hour Error Ratio
```

#### Unité

- Pourcentage (%)

#### Informations affichées

- Taux d’erreurs actuel
- Comparaison historique
- Visualisation de série temporelle

## Résumé des mesures du tableau de bord

| Tableau de bord | Mesures de Principal |
| -------------------------- | --------------------------------------------- |
| Vue d’ensemble | Nombre total de requêtes, débit, taux d’erreur, APDEX |
| Taux de demande | Demandes par minute |
| Taux d’erreurs | Pourcentage d’erreur |
| Durée de la demande | P50, P75, P90 Latence |
| Demandes par code d’état | Débit du statut HTTP |
| Taux de requêtes par point d’entrée | Volume de requêtes de point d’entrée |
| Comparaison du temps de réponse | P95 actuel et historique |
| Score APDEX | Indice de satisfaction des utilisateurs |
| Débit et latence | Débit des demandes et latence P95 |
| Taux d&#39;erreurs par groupe de statuts | Pourcentage d’erreur du groupe de statut HTTP |
| Tendances du taux d’erreurs | Ratio d’erreurs actuelles par rapport à historiques |
