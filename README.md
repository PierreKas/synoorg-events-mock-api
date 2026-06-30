# Synoorg Events — Mock API

Ce repository contient des données JSON statiques simulant l'API backend de **Synoorg Events**, une application Flutter de gestion d'abonnements et d'activités communautaires.

Aucun vrai backend n'existe pour ce projet. Les endpoints `GET` sont servis comme fichiers statiques via `raw.githubusercontent.com`. Le seul endpoint `POST` (`/subscription/change`) est simulé séparément via [Beeceptor](https://beeceptor.com).

---

## Base URL — Endpoints GET

```
https://raw.githubusercontent.com/PierreKas/synoorg-events-mock-api/master/
```

Chaque fichier ci-dessous est accessible en ajoutant son nom à la base URL.

| Endpoint logique | Fichier | URL complète |
|---|---|---|
| Liste des plans d'abonnement | `subscription-plans.json` | `.../subscription-plans.json` |
| Abonnement actuel de l'utilisateur | `subscription-current.json` | `.../subscription-current.json` |
| Résumé financier | `finance-summary.json` | `.../finance-summary.json` |
| Historique des paiements | `payments-history.json` | `.../payments-history.json` |
| Moyens de paiement enregistrés | `payment-methods.json` | `.../payment-methods.json` |
| Liste des activités | `activities.json` | `.../activities.json` |
| Liste des organisateurs | `activities-organizers.json` | `.../activities-organizers.json` |
| Détail d'une activité | `activities-by-id.json` | `.../activities-by-id.json` |

### Exemple de test avec `curl`

```bash
curl https://raw.githubusercontent.com/PierreKas/synoorg-events-mock-api/master/subscription-plans.json
```

### ⚠️ Important — Pas de vrais paramètres d'URL

Ces fichiers sont **statiques**. Cela signifie que :

- `activities-by-id.json` retourne **toujours** le même détail d'activité (`act_002`), peu importe l'ID demandé. Dans l'application Flutter, simule le comportement attendu : appelle ce fichier comme si l'ID était dans l'URL, mais traite la réponse normalement.
- Il n'existe pas de pagination réelle sur `payments-history.json` même si le champ `meta` dans la réponse suggère une pagination. Le champ `meta` doit être utilisé pour **afficher** une UI de pagination, mais ne déclenchera pas de changement réel de données.
- Aucun filtre côté serveur n'existe sur `activities.json` ou `payment-methods.json`. Tout filtrage (par organisateur, par statut, etc.) doit être fait **côté client**, en Dart, après réception des données.

---

## Endpoint POST — `/subscription/change`

Contrairement aux fichiers `GET` ci-dessus, cet endpoint est un **vrai POST fonctionnel**, hébergé sur Beeceptor.

### Base URL

```
https://synoorg-events.free.beeceptor.com
```

### Requête

```
POST /subscription/change
Content-Type: application/json
```

**Body :**
```json
{
  "planId": "plan_pro",
  "billingCycle": "monthly"
}
```

### Combinaisons disponibles

Seules les combinaisons suivantes ont une règle configurée et retourneront une réponse `200` :

| `planId` | `billingCycle` | Résultat |
|---|---|---|
| `plan_starter` | `monthly` | ✅ Fonctionne |
| `plan_pro` | `monthly` | ✅ Fonctionne |
| `plan_enterprise` | `yearly` | ✅ Fonctionne |
| Toute autre combinaison | — | ❌ Pas de règle — gérer l'erreur côté client |

### Exemples testés

```bash
curl -X POST https://synoorg-events.free.beeceptor.com/subscription/change \
  -H "Content-Type: application/json" \
  -d '{"planId": "plan_starter", "billingCycle": "monthly"}'
```

```bash
curl -X POST https://synoorg-events.free.beeceptor.com/subscription/change \
  -H "Content-Type: application/json" \
  -d '{"planId": "plan_pro", "billingCycle": "monthly"}'
```

```bash
curl -X POST https://synoorg-events.free.beeceptor.com/subscription/change \
  -H "Content-Type: application/json" \
  -d '{"planId": "plan_enterprise", "billingCycle": "yearly"}'
```

### Réponse type (succès)

```json
{
  "data": {
    "item": {
      "id": "sub_004",
      "planId": "plan_pro",
      "planName": "Pro",
      "billingCycle": "monthly",
      "status": "active",
      "currentPeriodStart": "2026-06-29T00:00:00.000Z",
      "currentPeriodEnd": "2026-07-29T00:00:00.000Z",
      "autoRenew": true,
      "amount": 12.99,
      "currency": "USD"
    }
  },
  "message": "Abonnement mis à jour avec succès"
}
```

### Cas d'erreur

Si tu envoies une combinaison `planId` / `billingCycle` non listée ci-dessus, Beeceptor retournera une réponse par défaut (souvent une erreur ou un body vide). **Ton code Flutter doit gérer ce cas proprement** — c'est un comportement attendu et volontaire du test, pas un bug de l'API mock.

---

## Structure générale des réponses

Toutes les réponses suivent ce format, à l'exception du POST décrit ci-dessus :

```json
{
  "data": {
    "items": [ /* tableau, pour les listes */ ]
  }
}
```

ou

```json
{
  "data": {
    "item": { /* objet unique */ }
  }
}
```

Adapte tes modèles Dart (`fromJson`) en conséquence — la donnée utile est toujours imbriquée dans `data.items` ou `data.item`.

---

## Résumé rapide pour les développeurs

1. Les `GET` sont des fichiers statiques GitHub — pas de vrais paramètres, pas de vrai filtrage serveur.
2. Le seul `POST` réel est sur Beeceptor, avec seulement 3 combinaisons fonctionnelles.
3. Tout le filtrage, la pagination visuelle, et la simulation de paramètres d'URL doivent être implémentés **côté client en Dart**.
4. Le format de réponse est toujours enveloppé dans `data.items` ou `data.item`.
