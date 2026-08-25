---
title: Référence du tableau de bord de l’infrastructure
description: Référence complète pour les tableaux de bord de l’infrastructure d’Observability Insights, y compris des captures d’écran, des mesures et des unités.
feature: Operations
role: Admin
source-git-commit: 1d54a6a398360b040221db5b2780d301722894bf
workflow-type: tm+mt
source-wordcount: '1091'
ht-degree: 7%

---


# Référence du tableau de bord de l’infrastructure {#infrastructure-dashboard-reference}

Cette référence documente les panneaux d’infrastructure au niveau de l’hôte utilisés dans Observability Insights for AEM Managed Services.

## Présentation du tableau de bord

Le tableau de bord de surveillance de l’infrastructure hôte offre une visibilité en temps réel sur l’utilisation et les performances de l’hôte sous-jacent. Ces mesures aident les opérateurs à surveiller les ressources de calcul, de mémoire, de stockage et de réseau, tout en identifiant les goulots d’étranglement potentiels des ressources.

Le tableau de bord comprend les panneaux de surveillance suivants :

- Utilisation du CPU hôte
- E/S du disque hôte
- E/S du réseau hôte
- Attente d’E/S CPU
- Utilisation du stockage
- Utilisation du disque
- Charge moyenne du CPU hôte
- Utilisation de la mémoire hôte

## &#x200B;1. Utilisation du CPU hôte

![Utilisation du CPU hôte](../assets/host-monitoring/host_cpu_utilization.png)

### Description

Le panneau **Utilisation du CPU hôte** affiche le pourcentage de ressources CPU actuellement utilisées par le système d’exploitation et tous les processus en cours d’exécution au fil du temps.

Cette mesure représente l’utilisation globale de CPU sur l’hôte et fournit une visualisation de série temporelle de l’activité du processeur.

Le graphique permet aux opérateurs de surveiller l’évolution de la consommation CPU au cours de la fenêtre d’observation sélectionnée.

### Mesure

| Mesure | Description |
| --------- | ---------------------------------------- |
| `cpu_pct` | Pourcentage du CPU total actuellement utilisé |

### Unités

- Pourcentage (%)

### Statistiques affichées

Le panneau résume l’utilisation de CPU à l’aide de trois valeurs :

| Statistique | Description |
| --------- | --------------------------------------------------------------- |
| Moyenne | Utilisation moyenne de CPU au cours de la période sélectionnée |
| Dernière | Dernière valeur d’utilisation de CPU collectée |
| Max | Utilisation de CPU la plus élevée observée au cours de la période sélectionnée |

### Composants de graphique

- Ligne de série temporelle représentant l’utilisation de CPU.
- Axe Y en pourcentage compris entre **0 % et 100 %**.
- Statistiques récapitulatives affichées sous le graphique.
- Tendance historique pour l’intervalle de surveillance sélectionné.

## &#x200B;2. E/S du disque hôte

![E/S de disque hôte](../assets/host-monitoring/host_disk_io.png)

### Description

Le panneau **E/S de disque hôte** affiche le débit de stockage pour les opérations de lecture et d’écriture de disque effectuées par l’hôte.

Le graphique présente deux séries temporelles indépendantes qui représentent les données transférées entre le système d’exploitation et les périphériques de stockage.

Cette visualisation permet de surveiller l’activité du stockage au fil du temps et fournit à insight le volume de données lues et écrites sur des disques.

### Mesures

| Mesure | Description |
| ------------ | --------------------------------- |
| `disk_read` | Quantité de données lues à partir du stockage |
| `disk_write` | Quantité de données écrites en stockage |

En interne, ces mesures sont affichées à l’aide de valeurs de débit lissées.

### Unités

- Octets par seconde (B/s)
- Kilo-octets par seconde (Ko/s)
- Mo/s (Mo/s)
- Gigaoctets par seconde (Go/s)

L’unité affichée est automatiquement mise à l’échelle en fonction du débit.

### Composants de graphique

- Ligne verte représentant le débit de lecture du disque.
- Ligne orange représentant le débit d’écriture du disque.
- Visualisation de série temporelle.
- Légende distincte pour chaque mesure.
- Valeurs de mesure actuelles affichées à côté de chaque série.

## &#x200B;3. E/S du réseau hôte

![E/S réseau hôte](../assets/host-monitoring/host_network_io.png)

### Description

Le panneau **E/S réseau de l’hôte** affiche le volume du trafic réseau transmis et reçu par l’hôte au fil du temps.

Le graphique mesure le débit auquel les données transitent par les interfaces réseau et offre une visibilité sur la consommation de bande passante du réseau.
Cette mesure représente le débit réseau agrégé.

### Mesure

| Mesure | Description |
| --------------- | --------------------------------------------------------------------- |
| `bytes_per_sec` | Débit réseau agrégé mesuré en octets transférés par seconde |

### Unités

Le graphique est automatiquement mis à l’échelle entre :

- Octets/s
- Ko/s
- Mo/s
- Go/s

en fonction du volume de trafic observé.

### Statistiques affichées

| Statistique | Description |
| --------- | ---------------------------------- |
| Moyenne | Débit réseau moyen |
| Dernière | Mesure de débit la plus récente |
| Max | Débit le plus élevé observé |

### Composants de graphique

- Ligne à débit unique.
- Visualisation de série temporelle.
- Mise à l’échelle dynamique de la bande passante.
- Statistiques récapitulatives affichées sous le graphique.

## &#x200B;4. Attente d’E/S CPU

![Attente D’E/S ](../assets/host-monitoring/cpu_io_wait.png)

### Description

Le panneau **Attente d’E/S de** affiche le pourcentage du temps passé par CPU à attendre la fin des opérations d’entrée/sortie.

Cette mesure représente le temps d’inactivité du processeur qui se produit car les processus actifs sont bloqués en attendant les périphériques de stockage ou d’autres opérations d’E/S.

Le graphique permet de visualiser l’évolution de l’attente d’E/S au fil du temps.

### Mesure

| Mesure | Description |
| --------- | ------------------------------------------------------ |
| `cpu_pct` | Pourcentage de temps passé par CPU à attendre des opérations d’E/S |

### Unités

- Pourcentage (%)

### Statistiques affichées

| Statistique | Description |
| --------- | ------------------------------- |
| Moyenne | Pourcentage moyen d’attente d’E/S de CPU |
| Dernière | Valeur enregistrée le plus récemment |
| Max | Valeur enregistrée la plus élevée |

### Composants de graphique

- Ligne de série temporelle.
- Axe Y en pourcentage.
- Statistiques récapitulatives.
- Visualisation des tendances historiques.

## &#x200B;5. Utilisation du stockage

![Utilisation du stockage](../assets/host-monitoring/storage_disk_usage.png)

### Description

Le panneau **Utilisation du stockage** affiche le pourcentage global de la capacité de stockage actuellement utilisée sur l’hôte surveillé.

Le graphique fournit une vue historique de l’utilisation de la capacité du système de fichiers au cours de l’intervalle de temps sélectionné.

### Mesure

| Mesure | Description |
| --------------- | -------------------------------------------------- |
| Pourcentage d’utilisation du stockage | Pourcentage de stockage alloué actuellement consommé |

### Unités

- Pourcentage (%)

### Composants de graphique

- Graphique d’utilisation de série temporelle.
- Échelle de pourcentage.
- Tendance d’utilisation du stockage dans l’historique.

## &#x200B;6. Utilisation du disque

![ Utilisation du disque ](../assets/host-monitoring/storage_disk_usage.png)

### Description

Le panneau **Utilisation du disque** affiche l’utilisation du stockage pour chaque système de fichiers ou périphérique de stockage monté.

Chaque ligne correspond à un périphérique de bloc spécifique ou à une partition montée et indique le pourcentage d&#39;espace actuellement utilisé.

Ce tableau présente la répartition de l’utilisation du stockage au niveau du système de fichiers.

### Informations affichées

Chaque entrée comprend :

| Champ | Description |
| --------------- | -------------------------------------------- |
| Appareil | Dispositif de stockage ou système de fichiers monté |
| % utilisé | Pourcentage de capacité de stockage utilisée |
| Barre d’utilisation | Représentation visuelle de la consommation de stockage |

### Unités

- Pourcentage (%)

### Composants de graphique

- Liste des systèmes de fichiers/appareils.
- Pourcentage d&#39;utilisation.
- Indicateur de capacité avec code couleur.
- Valeurs d’utilisation triées.

## &#x200B;7. Charge moyenne du CPU hôte

![Charge moyenne du CPU hôte](../assets/host-monitoring/host_cpu_load_average.png)

### Description

Le panneau **Moyenne de charge du CPU hôte** affiche les moyennes de charge du système Linux sur trois périodes flottantes.

Contrairement à l’utilisation de CPU, la moyenne de charge représente le nombre moyen de processus en cours d’exécution ou en attente de planification CPU ou d’achèvement des E/S.

Le graphique affiche simultanément trois moyennes glissantes qui fournissent des tendances de charge de travail à court et à long terme.

### Mesures

| Mesure | Description |
| ---------- | -------------------------------------------- |
| `load_1m` | Charge moyenne du système au cours de la dernière minute |
| `load_5m` | Charge moyenne du système au cours des 5 dernières minutes |
| `load_15m` | Charge moyenne du système au cours des 15 dernières minutes |

### Unités

- Charger la moyenne (valeur sans dimension)

### Statistiques affichées

Pour chaque mesure de charge moyenne :

| Statistique | Description |
| --------- | --------------------------------------- |
| Moyenne | Charge moyenne pendant la période sélectionnée |
| Dernière | Dernière valeur de charge enregistrée |
| Max | Valeur de charge la plus élevée observée |

### Composants de graphique

- Trois lignes de tendance indépendantes.
- Visualisation de série temporelle.
- Légendes individuelles pour chaque moyenne mobile.
- Statistiques récapitulatives pour chaque mesure.

## &#x200B;8. Utilisation de la mémoire hôte

![Utilisation de la mémoire hôte](../assets/host-monitoring/host_memory_usage.png)

### Description

Le panneau **Utilisation de la mémoire hôte** affiche le pourcentage de mémoire système physique actuellement allouée par le système d’exploitation.

Cette mesure représente l’utilisation globale de la RAM pour tous les processus en cours d’exécution, la mémoire du noyau, les tampons et les caches.

Le graphique fournit une vue continue de la consommation de mémoire tout au long de la période de surveillance sélectionnée.

### Mesure

| Mesure | Description |
| ------------ | ---------------------------------------------- |
| `memory_pct` | Pourcentage de la mémoire physique actuellement utilisée |

### Unités

- Pourcentage (%)

### Statistiques affichées

| Statistique | Description |
| --------- | ---------------------------------- |
| Moyenne | Utilisation moyenne de la mémoire |
| Dernière | Utilisation enregistrée le plus récemment |
| Max | Utilisation la plus élevée observée |

### Composants de graphique

- Graphique d’utilisation de la mémoire dans la série temporelle.
- Axe Y en pourcentage.
- Tendance d’utilisation historique.
- Statistiques récapitulatives.

## Résumé des mesures du tableau de bord

| Panneau de tableau de bord | Mesure de Principal | Unité |
| --------------------- | -------------------------------- | ------------ |
| Utilisation du CPU hôte | `cpu_pct` | % |
| E/S du disque hôte | `disk_read`, `disk_write` | Octets/s |
| E/S du réseau hôte | `bytes_per_sec` | Octets/s |
| Attente d’E/S CPU | `cpu_pct` | % |
| Utilisation du stockage | Pourcentage d’utilisation du stockage | % |
| Utilisation du disque | Utilisation du système de fichiers | % |
| Charge moyenne du CPU hôte | `load_1m`, `load_5m`, `load_15m` | Load Average |
| Utilisation de la mémoire hôte | `memory_pct` | % |
