---
source-git-commit: e5523081fcd68500602e5d1bf853694d1f6c3980
workflow-type: tm+mt
source-wordcount: '1104'
ht-degree: 7%

---
# API publique Observability Insights

L’API publique Observability Insights vous permet d’extraire vos propres données d’observabilité (vues d’ensemble des demandes, catalogues de services, traces et mesures) directement dans vos propres outils, scripts et tableaux de bord.

- **URL de base de l’API (API_BASE_URL) :** `https://insights.adobecqms.net/`
- **Format :** JSON sur HTTPS
- **Authentification : clé API** (jeton du porteur)

> Remplacez `{{API_BASE_URL}}` dans ce document par l’hôte API de votre instance d’Observability Insights, par exemple `https://insights.adobecqms.net/`.

---

## &#x200B;1. Obtention d’une clé API

Les clés API sont des informations d’identification personnelles liées à votre compte et limitées à une seule organisation. Une clé peut uniquement lire les données des clients qui appartiennent à l’organisation pour laquelle elle a été créée. Elle ne peut jamais voir les données d’une autre organisation.

### Générer une clé

1. Connectez-vous au tableau de bord [Observability Insights](https://insights.adobecqms.net/).
2. Ouvrez le menu de votre profil (en haut à droite) → **Clés API**.
   ![menu Clés API](v2-assets/api-key.png)
3. Dans l&#39;onglet **Clés API**, cliquez sur **Générer la clé**.
   ![Générer la clé API](v2-assets/api-key-gen.png)
4. Donnez-lui un nom explicite (par exemple, `CI pipeline`, `Grafana datasource`), sélectionnez l’organisation vers laquelle elle doit être étendue et définissez éventuellement une date d’expiration.
5. Cliquez sur **Générer la clé**. Votre clé s’affiche **une fois** au format suivant :

   ```
   synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
   ```

   **Copiez-le immédiatement et stockez-le dans un endroit sûr** (un gestionnaire de secrets, une banque de secrets CI, etc.) — le tableau de bord ne peut plus vous l&#39;afficher. Si vous le perdez, révoquez-le et générez-en un nouveau.

### Gestion des clés existantes

La section Clés API répertorie toutes les clés que vous avez créées, y compris leur organisation, leur date de création, leur expiration et la date de dernière utilisation. Cliquez sur l’icône de la corbeille en regard d’une clé pour la **révoquer** ; la révocation est immédiate et ne peut pas être annulée.

### Sécurité des clés

- Traitez une clé API exactement comme un mot de passe. Toute personne disposant de la clé peut lire toutes les données d’observabilité pour chaque client de l’organisation auquel elle est étendue, jusqu’à sa révocation ou son expiration.
- Ne validez jamais une clé pour le contrôle de code source et ne la partagez jamais en texte brut (chat, e-mail, tickets).
- Faites pivoter les clés périodiquement et révoquez toute clé qui n&#39;est plus utilisée.
- Si une clé est compromise, révoquez-la immédiatement à partir de **Paramètres de l’organisation → Clés API** et générez un remplacement.

---

## &#x200B;2. Authentification des requêtes

Chaque requête à l’API publique doit inclure votre clé dans l’en-tête `Authorization` :

```
Authorization: Bearer synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
```

Les requêtes sans clé valide ou avec une clé expirée/révoquée reçoivent des `401 Unauthorized`. Les connexions de session (cookies/jetons de navigateur) ne sont **pas** acceptées sur cette API .

---

## &#x200B;3. Concepts de base

### Clients

Chaque point d’entrée nécessite un paramètre de requête `tenant_id` identifiant les données du client à lire. Une clé ne peut interroger que les clients appartenant à l’organisation pour laquelle elle a été créée ; la demande d’un client en dehors de cette organisation renvoie `403 Forbidden`. Il n’y a pas de mode « tous les clients » sur cette API - transmettez toujours un `tenant_id` spécifique.

Vous ne savez pas quelles valeurs de `tenant_id` votre clé peut utiliser ? Appelez [`GET /public/v1/tenants`](#get-publicv1tenants) — il répertorie exactement les clients que votre clé est autorisée à interroger.

### Périodes

Les points d’entrée qui acceptent les paramètres `from`/`to` prennent les horodatages Unix (secondes), les horodatages en millisecondes ou les chaînes datetime ISO 8601, par exemple :

```
from=1735689600
from=2025-01-01T00:00:00Z
```

Si cet attribut est omis, la plupart des points d’entrée sont définis par défaut sur une fenêtre dynamique récente (voir chaque point d’entrée ci-dessous).

### Limites de débit

Les requêtes sont limitées par taux par clé API. Si vous dépassez la limite, vous recevrez :

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{ "error": "Too Many Requests", "message": "Rate limit of 300 requests/60s exceeded" }
```

Arrêtez puis réessayez après le nombre de secondes dans l’en-tête `Retry-After`. Contactez l’assistance si votre cas d’utilisation nécessite une limite plus élevée.

### Erreurs

Les erreurs sont renvoyées au format JSON avec un champ `error` et, en règle générale, un `message` lisible par l’utilisateur :

```json
{ "error": "Bad Request", "message": "tenant_id is required" }
```

| Statut | Signification |
| ------------------------- | ------------------------------------------------------------------ |
| `400 Bad Request` | Paramètre manquant ou non valide (par exemple, aucune `tenant_id`, période incorrecte) |
| `401 Unauthorized` | Clé API manquante, non valide, expirée ou révoquée |
| `403 Forbidden` | La clé n’est pas autorisée pour le client demandé |
| `429 Too Many Requests` | Limite de taux dépassée — voir `Retry-After` |
| `502 Bad Gateway` | Échec de la requête en amont. Réessayez en toute sécurité. |
| `503 Service Unavailable` | Serveur principal de données temporairement indisponible |

---

## &#x200B;4. Points d’entrée

### `GET /public/v1/tenants`

Répertorie les identifiants de client que votre clé est autorisée à interroger. Appelez-le d’abord : tous les autres points d’entrée requièrent l’une de ces valeurs comme `tenant_id`.

```bash
curl -s "{{API_BASE_URL}}/public/v1/tenants" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{ "tenants": ["tenant1", "tenant2"] }
```

### `GET /public/v1/overview`

KPI de haut niveau d’intégrité pour un client sur une période donnée : volume des demandes, taux d’erreur et centiles de latence.

| Param | Requis | Description |
| ------------ | -------- | ------------------------------------------------------------------------- |
| `tenant_id` | Oui | Client à interroger |
| `from`, `to` | Non | Période (voir [Périodes](#time-ranges)) |
| `minutes` | Non | Raccourci de « N dernières minutes » si `from`/`to` ne sont pas donnés (`15` par défaut) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/overview?tenant_id=<tenant_id>&minutes=30" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "total_spans": 48213,
  "errors": 112,
  "error_rate_pct": 0.23,
  "p50_ms": 34,
  "p95_ms": 210,
  "p99_ms": 480,
  "service_count": 12,
  "trace_count": 9021
}
```

### `GET /public/v1/services`

Répertorie les rapports de noms de service distincts pour un client.

| Param | Requis | Description |
| ------------ | -------- | --------------------------------------------------------------------- |
| `tenant_id` | Oui | Client à interroger |
| `from`, `to` | Non | Limiter aux services affichés dans cette fenêtre ; par défaut, les 7 derniers jours |

```bash
curl -s "{{API_BASE_URL}}/public/v1/services?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "services": ["checkout-api", "payments-worker", "web-frontend"]
}
```

### `GET /public/v1/traces`

Recherche les traces récentes d’un client avec des filtres facultatifs.

| Param | Requis | Description |
| ----------------- | -------- | ------------------------------------------------- |
| `tenant_id` | Oui | Client à interroger |
| `from`, `to` | Non | Période ; 24 dernières heures par défaut |
| `limit` | Non | Nombre maximal de lignes à renvoyer (1-200, 100 par défaut) |
| `offset` | Non | Décalage de pagination (0 par défaut) |
| `service` | Non | Filtrer par nom de service |
| `app_name` | Non | Filtrer par nom d’application/instance |
| `status` | Non | Filtrer par statut de suivi : `ok`, `error` ou `unset` |
| `search` | Non | Recherche de texte libre dans les noms d’étendue/d’opération |
| `min_duration_ms` | Non | Ne laisse que les traces égales ou supérieures à cette durée |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces?tenant_id=<tenant_id>&status=error&limit=25" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TraceId": "4bf92f3577b34da6a3ce929d0e0e4736",
      "ServiceName": "checkout-api",
      "DurationMs": 812,
      "StatusCode": "Error",
      "Timestamp": "2026-08-30T09:12:44Z"
    }
  ],
  "rows": 137,
  "limit": 25,
  "offset": 0
}
```

Utilisez `rows` (nombre correspondant total) avec `limit`/`offset` pour parcourir les résultats.

### `GET /public/v1/traces/:traceId`

Renvoie la cascade complète pour une seule trace.

| Param | Requis | Description |
| ----------- | -------- | ---------------------------------------- |
| `tenant_id` | Oui | Client auquel appartient la trace |
| `limit` | Non | Plages maximales à renvoyer (1-500, 500 par défaut) |
| `offset` | Non | Décalage de pagination pour les traces très volumineuses |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces/4bf92f3577b34da6a3ce929d0e0e4736?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "spans": [
    {
      "SpanId": "00f067aa0ba902b7",
      "Name": "POST /checkout",
      "DurationMs": 812,
      "children": []
    }
  ],
  "totalDurationMs": 812,
  "spanCount": 14,
  "limit": 500,
  "offset": 0
}
```

### `GET /public/v1/metrics`

Renvoie des points de données de mesure bruts pour un client.

| Param | Requis | Description |
| ---------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `tenant_id` | Oui | Client à interroger |
| `metric` | Un de `metric`/`like` | Nom exact de la mesure |
| `like` | Un de `metric`/`like` | Modèle de `LIKE` SQL pour correspondre à plusieurs noms de mesures |
| `type` | Non | `gauge` (par défaut) ou `sum` |
| `from`, `to` | Non | Période ; 24 dernières heures par défaut |
| `service` | Non | Filtrer par nom de service |
| `host` | Non | Filtrez par nom d’hôte. Requis pour les mesures de l’hôte d’infrastructure ci-dessous : sans cela, les lectures de chaque hôte du client sont combinées |
| `attribute_key`, `attribute_value` | Non | Filtrer selon un attribut de mesure spécifique (doit être utilisé conjointement) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=jvm.memory.used&type=gauge" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TimeUnix": "2026-08-30T09:00:00Z",
      "MetricName": "jvm.memory.used",
      "Value": 512482816,
      "ServiceName": "checkout-api",
      "host": ""
    }
  ],
  "rows": 1
}
```

#### Mesures d’hôte de l’infrastructure

Le même point d’entrée sert également les mesures au niveau de l’hôte affichées dans le tableau de bord de l’infrastructure (CPU, mémoire, charge moyenne, E/S de disque, E/S réseau). Utilisez ces combinaisons `metric` / `attribute_key` / `attribute_value` exactes, toujours avec un `host` :

| Widget de tableau de bord | `metric` | `attribute_key` | `attribute_value` |
| --------------------- | ------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| % CPU | `system.cpu.utilization` | `state` | `idle` (soustrayez de 1 pour « en cours d’utilisation ») ou demandez `user`/`system`/`iowait` séparément et additionnez |
| % d’utilisation de la mémoire | `system.memory.utilization` | `state` | `used` |
| Charge moyenne (1m) | `system.cpu.load_average.1m` | — | — |
| E/S de lecture de disque | `system.disk.io` (`type=sum`) | `direction` | `read` |
| E/S d’écriture de disque | `system.disk.io` (`type=sum`) | `direction` | `write` |
| Opérations de lecture sur le disque | `system.disk.operations` (`type=sum`) | `direction` | `read` |
| Opérations d’écriture sur le disque | `system.disk.operations` (`type=sum`) | `direction` | `write` |
| Réseau dans | `system.network.io` (`type=sum`) | `direction` | `receive` |
| Réseau sortant | `system.network.io` (`type=sum`) | `direction` | `transmit` |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=system.cpu.utilization&type=gauge&attribute_key=state&attribute_value=idle&host=<host_name>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

**Important — les valeurs du disque et du réseau sont des compteurs bruts, qui ne cessent d&#39;augmenter, et non des taux.** Les graphiques « octets/s » et « opérations/s » du tableau de bord sont calculés en prenant deux lectures de compteur consécutives et en divisant par le temps écoulé :

```
rate = (value_at_t2 - value_at_t1) / (t2 - t1_in_seconds)
```

### `GET /public/v1/pages`

Pages de contenu les plus demandées (`.html`) par instance de Dispatcher, classées par nombre de demandes. Appuyé par la mesure `dispatcher.httpd.requests` — ce point d’entrée est spécifique aux journaux d’accès de style Dispatcher/CDN AEM, et non un outil d’analyse de pages général.

| Param | Requis | Description |
| ------------ | -------- | -------------------------------------- |
| `tenant_id` | Oui | Client à interroger |
| `from`, `to` | Non | Période ; 24 dernières heures par défaut |
| `limit` | Non | Nombre maximal de lignes à renvoyer (1-500, 50 par défaut) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/pages?tenant_id=<tenant_id>&limit=50" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "data": [
    {
      "instance": "<instance_name>",
      "domain": "www.abc.com",
      "path": "/join-us/insights.html",
      "full_url": "https://www.abc.com/join-us/insights.html",
      "requests": 7
    }
  ],
  "rows": 1
}
```

---

## &#x200B;5. Ce que cette API ne fait pas

- **Pas d’accès SQL brut.** Tous les points d’entrée renvoient des formes de données personnalisées — vous ne pouvez pas interroger directement la banque de données sous-jacente.
- **Aucune requête entre clients.** Chaque requête est limitée à une seule `tenant_id`.
- **Pas d’accès en écriture.** L’API publique est en lecture seule.

---

## &#x200B;6. Assistance

Si vous rencontrez des erreurs inattendues ou si un cas d’utilisation n’est pas couvert par ces points d’entrée, contactez votre ingénieur du succès client/de l’activation pour obtenir de l’aide.
