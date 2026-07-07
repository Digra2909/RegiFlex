#— API Doc

> **Stack**: Laravel 13.x + Sanctum (token auth) + Spatie Permissions  
> **Base URL**: `https://votre-domaine.com/api`  
> **Format**: JSON — toutes les requêtes et réponses sont en JSON.  
> **Encodage**: `Content-Type: application/json` (sauf `/api/import` qui est `multipart/form-data`).

---

## 1. Authentification

### 1.1 Connexion

```
POST /api/login
Content-Type: application/json
```

**Corps de la requête :**

```json
{
  "email": "admin@example.com",
  "password": "password"
}
```

**Réponse (200) :**

```json
{
  "message": "Connexion réussie",
  "user": {
    "id": 1,
    "name": "Admin",
    "email": "admin@example.com",
    "roles": ["Admin"]
  },
  "token": "1|abc123..."
}
```

> ⚠️ Limité à **5 requêtes par minute**.  
> Le token expire après **7 jours**.

### 1.2 Utilisation du token

Ajouter l'en-tête HTTP à chaque requête authentifiée :

```
Authorization: Bearer 1|abc123...
```

### 1.3 Déconnexion

```
POST /api/logout
Authorization: Bearer <token>
```

**Réponse (200) :**

```json
{
  "message": "Déconnexion réussie"
}
```

### 1.4 Profil utilisateur courant

```
GET /api/me
Authorization: Bearer <token>
```

**Réponse (200) :**

```json
{
  "user": {
    "id": 1,
    "name": "Admin",
    "email": "admin@example.com",
    "roles": ["Admin"],
    "permissions": [
      "create-sites", "read-sites", "update-sites", "delete-sites",
      "create-structures", "read-structures", "update-structures", "delete-structures",
      "create-agents", "read-agents", "update-agents", "delete-agents",
      "create-equipements", "read-equipements", "update-equipements", "delete-equipements",
      "import-data", "view-logs", "manage-users"
    ]
  }
}
```

### 1.5 Inscription d'un nouvel utilisateur (Admin uniquement)

```
POST /api/register
Authorization: Bearer <token_admin>
Content-Type: application/json
```

**Corps de la requête :**

```json
{
  "name": "Jean Dupont",
  "email": "jean@example.com",
  "password": "secret123",
  "password_confirmation": "secret123",
  "role": "Rapporteur"
}
```

- `role` : optionnel, valeurs possibles : `Admin`, `Rapporteur` (défaut : `Rapporteur`)

**Réponse (201) :**

```json
{
  "message": "Inscription réussie",
  "user": {
    "id": 2,
    "name": "Jean Dupont",
    "email": "jean@example.com",
    "roles": ["Rapporteur"]
  },
  "token": "2|def456..."
}
```

> ⚠️ Limité à **10 requêtes par minute**.

---

## 2. Rôles et Permissions

| Rôle        | Description                                      |
|-------------|--------------------------------------------------|
| Admin       | Accès complet à toutes les fonctionnalités        |
| Rapporteur  | Lecture seule + consultation des logs             |

**Permissions** (16) : `{create,read,update,delete}-{sites,structures,agents,equipements}`  
+ `import-data`, `view-logs`, `manage-users`

Utilisez la réponse de `GET /api/me` pour afficher/cacher des éléments d'interface selon les permissions de l'utilisateur.

---

## 3. Gestion des Sites

### 3.1 Liste des sites

```
GET /api/sites
Authorization: Bearer <token>
Permission: read-sites
```

**Paramètres de requête (optionnels) :** `?page=2`

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": 1,
      "desi_site": "Site Principal",
      "created_at": "2025-01-15T10:00:00.000000Z",
      "updated_at": "2025-01-15T10:00:00.000000Z"
    }
  ],
  "meta": {
    "total": 10,
    "page": 1,
    "per_page": 15,
    "dernier_page": 1
  }
}
```

### 3.2 Détail d'un site

```
GET /api/sites/{id}
Authorization: Bearer <token>
Permission: read-sites
```

**Réponse (200) :**

```json
{
  "data": {
    "id": 1,
    "desi_site": "Site Principal",
    "structures": [
      { "id": 1, "desi_struct": "Direction Générale" }
    ],
    "created_at": "2025-01-15T10:00:00.000000Z",
    "updated_at": "2025-01-15T10:00:00.000000Z"
  }
}
```

**Réponse (404) :** `{ "message": "Site introuvable" }`

### 3.3 Création d'un site

```
POST /api/sites
Authorization: Bearer <token>
Permission: create-sites
Content-Type: application/json
```

**Corps :**

```json
{
  "desi_site": "Nouveau Site"
}
```

**Réponse (201) :**

```json
{
  "message": "Site créé avec succès",
  "site": {
    "id": 2,
    "desi_site": "Nouveau Site",
    "created_at": "2025-06-15T12:00:00.000000Z",
    "updated_at": "2025-06-15T12:00:00.000000Z"
  }
}
```

### 3.4 Modification d'un site

```
PUT|PATCH /api/sites/{id}
Authorization: Bearer <token>
Permission: update-sites
Content-Type: application/json
```

**Corps :**

```json
{
  "desi_site": "Site Modifié"
}
```

**Réponse (200) :**

```json
{
  "message": "Site modifié avec succès",
  "site": {
    "id": 1,
    "desi_site": "Site Modifié",
    "created_at": "2025-01-15T10:00:00.000000Z",
    "updated_at": "2025-06-15T12:00:00.000000Z"
  }
}
```

### 3.5 Suppression d'un site

```
DELETE /api/sites/{id}
Authorization: Bearer <token>
Permission: delete-sites
```

**Réponse (200) :**

```json
{
  "message": "Site supprimé avec succès"
}
```

---

## 4. Gestion des Structures

### 4.1 Liste des structures

```
GET /api/structures
Authorization: Bearer <token>
Permission: read-structures
```

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": 1,
      "desi_struct": "Direction Générale",
      "adresse": "123 Avenue du Centre",
      "type": "direction générale",
      "site": { "id": 1, "desi_site": "Site Principal" },
      "parent": null,
      "responsable": { "id": 1, "noms": "M. Dupont" },
      "created_at": "2025-01-15T10:00:00.000000Z",
      "updated_at": "2025-01-15T10:00:00.000000Z"
    }
  ],
  "meta": {
    "total": 5,
    "page": 1,
    "per_page": 15,
    "dernier_page": 1
  }
}
```

- `site`, `parent`, `responsable` peuvent être `null`

### 4.2 Détail d'une structure

```
GET /api/structures/{id}
Authorization: Bearer <token>
Permission: read-structures
```

**Réponse (200) :**

```json
{
  "data": {
    "id": 1,
    "desi_struct": "Direction Générale",
    "adresse": "123 Avenue du Centre",
    "type": "direction générale",
    "site": { "id": 1, "desi_site": "Site Principal" },
    "parent": null,
    "enfants": [
      { "id": 2, "desi_struct": "Service Informatique" }
    ],
    "responsable": { "id": 1, "noms": "M. Dupont" },
    "agents": [
      { "id": 1, "noms": "M. Dupont", "matricule": "AG001" }
    ],
    "created_at": "2025-01-15T10:00:00.000000Z",
    "updated_at": "2025-01-15T10:00:00.000000Z"
  }
}
```

### 4.3 Création d'une structure

```
POST /api/structures
Authorization: Bearer <token>
Permission: create-structures
Content-Type: application/json
```

**Corps :**

```json
{
  "desi_struct": "Service Informatique",
  "adresse": "456 Rue du Progrès",
  "type": "service",
  "site_id": 1,
  "structure_id": null,
  "agent_id": 2
}
```

- `type` : `direction générale`, `direction provinciale`, `direction`, `centre`, `service`
- `structure_id` : ID de la structure parente (optionnel, pour hiérarchie)
- `agent_id` : ID de l'agent responsable (optionnel)

**Réponse (201) :**

```json
{
  "message": "Structure créée avec succès",
  "structure": { "...même format que l'index..." }
}
```

### 4.4 Modification d'une structure

```
PUT|PATCH /api/structures/{id}
Authorization: Bearer <token>
Permission: update-structures
Content-Type: application/json
```

**Corps (mêmes champs que création, tous optionnels) :**

```json
{
  "desi_struct": "Service Informatique Mis à Jour",
  "adresse": "789 Boulevard Tech"
}
```

**Réponse (200) :** `{ "message": "Structure modifiée avec succès", "structure": {...} }`

### 4.5 Suppression d'une structure

```
DELETE /api/structures/{id}
Authorization: Bearer <token>
Permission: delete-structures
```

**Réponse (200) :** `{ "message": "Structure supprimée avec succès" }`

---

## 5. Gestion des Agents

### 5.1 Liste des agents

```
GET /api/agents
Authorization: Bearer <token>
Permission: read-agents
```

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": 1,
      "matricule": "AG001",
      "noms": "M. Dupont Jean",
      "genre": "Masculin",
      "sexe": "M",
      "fonction": "Directeur Général",
      "type_agent": "Directeur",
      "type_directeur": "Provincial",
      "structure": { "id": 1, "desi_struct": "Direction Générale" },
      "created_at": "2025-01-15T10:00:00.000000Z",
      "updated_at": "2025-01-15T10:00:00.000000Z"
    }
  ],
  "meta": {
    "total": 20,
    "page": 1,
    "per_page": 15,
    "dernier_page": 2
  }
}
```

- `structure`, `type_directeur` peuvent être `null`

### 5.2 Détail d'un agent

```
GET /api/agents/{id}
Authorization: Bearer <token>
Permission: read-agents
```

**Réponse (200) :**

```json
{
  "data": {
    "id": 1,
    "matricule": "AG001",
    "noms": "M. Dupont Jean",
    "genre": "Masculin",
    "sexe": "M",
    "fonction": "Directeur Général",
    "type_agent": "Directeur",
    "type_directeur": "Provincial",
    "structure": { "id": 1, "desi_struct": "Direction Générale" },
    "equipements": [
      { "id": 1, "desi_equipement": "Ordinateur Portable Dell" }
    ],
    "created_at": "2025-01-15T10:00:00.000000Z",
    "updated_at": "2025-01-15T10:00:00.000000Z"
  }
}
```

### 5.3 Création d'un agent

```
POST /api/agents
Authorization: Bearer <token>
Permission: create-agents
Content-Type: application/json
```

**Corps :**

```json
{
  "matricule": "AG042",
  "noms": "Mme. Kamara Fatou",
  "genre": "Féminin",
  "sexe": "F",
  "fonction": "Chef de Service",
  "type_agent": "Chef-service",
  "type_directeur": null,
  "structure_id": 2
}
```

- `type_agent` : `Commun`, `Chef-service`, `Directeur` (obligatoire)
- `type_directeur` : `Provincial`, `Opération` (obligatoire si `type_agent` = `Directeur`)

### 5.4 Modification d'un agent

```
PUT|PATCH /api/agents/{id}
Authorization: Bearer <token>
Permission: update-agents
```

### 5.5 Suppression d'un agent

```
DELETE /api/agents/{id}
Authorization: Bearer <token>
Permission: delete-agents
```

---

## 6. Gestion des Équipements

### 6.1 Liste des équipements

```
GET /api/equipements
Authorization: Bearer <token>
Permission: read-equipements
```

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": 1,
      "desi_equipement": "Ordinateur Portable Dell",
      "date_acc": "2024-03-15",
      "n_immo": "IMMO-001",
      "n_serie": "SN123456789",
      "ram": "16 Go",
      "type": "PC Portable",
      "cpu": "Intel Core i7",
      "disque_dur": "512 Go SSD",
      "observation": "Bon état",
      "agent": { "id": 1, "noms": "M. Dupont Jean" },
      "created_at": "2025-01-15T10:00:00.000000Z",
      "updated_at": "2025-01-15T10:00:00.000000Z"
    }
  ],
  "meta": {
    "total": 30,
    "page": 1,
    "per_page": 15,
    "dernier_page": 2
  }
}
```

### 6.2 Détail d'un équipement

```
GET /api/equipements/{id}
Authorization: Bearer <token>
Permission: read-equipements
```

### 6.3 Création d'un équipement

```
POST /api/equipements
Authorization: Bearer <token>
Permission: create-equipements
Content-Type: application/json
```

**Corps :**

```json
{
  "desi_equipement": "Ordinateur Portable HP",
  "date_acc": "2025-06-01",
  "n_immo": "IMMO-050",
  "n_serie": "HP987654321",
  "ram": "32 Go",
  "type": "PC Portable",
  "cpu": "Intel Core i9",
  "disque_dur": "1 To SSD",
  "observation": "Neuf",
  "agent_id": 2
}
```

- Tous les champs sauf `desi_equipement` sont optionnels
- `n_immo` est vérifié pour unicité (insensible à la casse)

### 6.4 Modification d'un équipement

```
PUT|PATCH /api/equipements/{id}
Authorization: Bearer <token>
Permission: update-equipements
```

### 6.5 Suppression d'un équipement

```
DELETE /api/equipements/{id}
Authorization: Bearer <token>
Permission: delete-equipements
```

---

## 7. Gestion des Utilisateurs (Admin uniquement)

### 7.1 Liste des utilisateurs

```
GET /api/users
Authorization: Bearer <token>
Role: Admin
```

**Paramètres optionnels :** `?role=Admin` ou `?role=Rapporteur`

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": 1,
      "name": "Admin",
      "email": "admin@example.com",
      "roles": ["Admin"],
      "created_at": "2025-01-01T00:00:00.000000Z"
    }
  ]
}
```

### 7.2 Détail d'un utilisateur

```
GET /api/users/{id}
Authorization: Bearer <token>
Role: Admin
```

### 7.3 Modification d'un utilisateur

```
PUT /api/users/{id}
Authorization: Bearer <token>
Role: Admin
Content-Type: application/json
```

**Corps :**

```json
{
  "name": "Admin Modifié",
  "email": "nouveau@example.com",
  "password": "nouveaumotdepasse",
  "password_confirmation": "nouveaumotdepasse",
  "role": "Admin"
}
```

- Tous les champs sont optionnels (`sometimes`)
- `password` nécessite `password_confirmation`

**Réponse (200) :**

```json
{
  "message": "Utilisateur mis à jour avec succès",
  "user": {
    "id": 1,
    "name": "Admin Modifié",
    "email": "nouveau@example.com",
    "roles": ["Admin"]
  }
}
```

### 7.4 Suppression d'un utilisateur

```
DELETE /api/users/{id}
Authorization: Bearer <token>
Role: Admin
```

> ❌ Impossible de se supprimer soi-même — retourne `403` avec message : `"Vous ne pouvez pas vous supprimer vous-même"`

---

## 8. Import de données (Admin uniquement)

```
POST /api/import
Authorization: Bearer <token>
Role: Admin
Content-Type: multipart/form-data
```

**Paramètres :**
- `file` (obligatoire) : fichier CSV, JSON, XLSX, XLS ou TXT — max **10 Mo**

Le fichier doit contenir le nom de la table cible dans son nom (ex: `sites.xlsx`, `agents.csv`, `structures.json`, `equipements.xls`).

**Réponse (200) :**

```json
{
  "message": "Import terminé",
  "table": "agents",
  "total": 50,
  "importes": 48,
  "erreurs": [
    {
      "ligne": 3,
      "erreur": "The matricule field is required.",
      "donnees": { "noms": "Test", ... }
    }
  ]
}
```

---

## 9. Logs d'activité

```
GET /api/logs
Authorization: Bearer <token>
Permission: view-logs
```

**Paramètres de requête (optionnels) :**

| Paramètre       | Type   | Description                                      |
|-----------------|--------|--------------------------------------------------|
| `model`         | string | Filtrer par modèle : `Site`, `Structure`, `Agent`, `Equipement` |
| `action`        | string | `created`, `updated`, `deleted`                  |
| `utilisateur_id`| int    | ID de l'utilisateur ayant fait l'action           |
| `from`          | date   | Date début (format `Y-m-d`)                      |
| `to`            | date   | Date fin (format `Y-m-d`)                        |
| `page`          | int    | Page courante (défaut: 1)                        |
| `per_page`      | int    | Éléments par page (défaut: 20, max: 100)         |

**Réponse (200) :**

```json
{
  "data": [
    {
      "id": "1718467200_1",
      "date": "2025-06-15 12:00:00",
      "event": "Agent 'AG001' a été créé par Admin",
      "model": "Agent",
      "model_id": 1,
      "action": "created",
      "user": "Admin",
      "data": { "...snapshot des données..." }
    }
  ],
  "total": 150,
  "page": 1,
  "per_page": 20,
  "dernier_page": 8
}
```

> ⚠️ Les logs sont stockés dans un fichier JSON — pas de pagination base de données. La pagination est appliquée en mémoire.

---

## 10. Modèles de données (schéma)

### 10.1 Site

| Champ      | Type         | Règles                                    |
|------------|--------------|-------------------------------------------|
| id         | int (PK)     | auto-incrément                            |
| desi_site  | string(150)  | **obligatoire**, unique (insensible casse) |
| created_at | timestamp    |                                           |
| updated_at | timestamp    |                                           |

### 10.2 Structure

| Champ        | Type         | Règles                                                |
|--------------|--------------|-------------------------------------------------------|
| id           | int (PK)     | auto-incrément                                        |
| desi_struct  | string(150)  | **obligatoire**, unique (insensible casse)             |
| adresse      | string(255)  | nullable                                               |
| type         | enum         | `direction générale`, `direction provinciale`, `direction`, `centre`, `service` |
| site_id      | FK → sites   | nullable, `nullOnDelete`                               |
| structure_id | FK → structures | nullable (parent hiérarchique), `nullOnDelete`       |
| agent_id     | FK → agents  | nullable (responsable), `nullOnDelete`                 |
| created_at   | timestamp    |                                                       |
| updated_at   | timestamp    |                                                       |

### 10.3 Agent

| Champ         | Type         | Règles                                                |
|---------------|--------------|-------------------------------------------------------|
| id            | int (PK)     | auto-incrément                                        |
| matricule     | string(50)   | **obligatoire**, **unique** (insensible casse)         |
| noms          | string(150)  | **obligatoire**                                       |
| genre         | string(10)   | nullable                                               |
| sexe          | string(1)    | nullable                                               |
| fonction      | string(100)  | nullable                                               |
| type_agent    | enum         | **obligatoire** : `Commun`, `Chef-service`, `Directeur`|
| type_directeur| enum         | nullable : `Provincial`, `Opération`                   |
| structure_id  | FK → structures | nullable, `nullOnDelete`                             |
| created_at    | timestamp    |                                                       |
| updated_at    | timestamp    |                                                       |

### 10.4 Équipement

| Champ           | Type         | Règles                                                |
|-----------------|--------------|-------------------------------------------------------|
| id              | int (PK)     | auto-incrément                                        |
| desi_equipement | string(150)  | **obligatoire**                                       |
| date_acc        | date         | nullable                                               |
| n_immo          | string(100)  | nullable, **unique** (insensible casse) si présent    |
| n_serie         | string(100)  | nullable                                               |
| ram             | string(50)   | nullable                                               |
| type            | string(50)   | nullable                                               |
| cpu             | string(100)  | nullable                                               |
| disque_dur      | string(100)  | nullable                                               |
| observation     | text         | nullable                                               |
| agent_id        | FK → agents  | nullable, `nullOnDelete`                               |
| created_at      | timestamp    |                                                       |
| updated_at      | timestamp    |                                                       |

### 10.5 Utilisateur

| Champ        | Type         | Règles                                      |
|--------------|--------------|---------------------------------------------|
| id           | int (PK)     | auto-incrément                              |
| name         | string(255)  | **obligatoire**                             |
| email        | string(255)  | **obligatoire**, **unique**                 |
| password     | string(255)  | **obligatoire**, min 6 car., hashé          |
| role         | string       | `Admin` ou `Rapporteur` (assigné à part)    |
| created_at   | timestamp    |                                             |
| updated_at   | timestamp    |                                             |

---

## 11. Relations entre modèles

```
Site
 └── hasMany → Structure

Structure
 ├── belongsTo → Site
 ├── belongsTo → Structure (parent)
 ├── hasMany   → Structure (enfants)
 ├── belongsTo → Agent (responsable)
 └── hasMany   → Agent

Agent
 ├── belongsTo → Structure
 ├── hasMany   → Equipement
 └── hasOne    → Structure (en tant que responsable)

Equipement
 └── belongsTo → Agent
```

---

## 12. Codes d'erreur communs

| Code | Signification                          |
|------|----------------------------------------|
| 200  | Succès                                 |
| 201  | Création réussie                       |
| 401  | Non authentifié (token manquant/invalide) |
| 403  | Non autorisé (rôle/permission manquante) |
| 404  | Ressource introuvable                  |
| 422  | Erreur de validation                   |
| 429  | Trop de requêtes (rate limit)          |
| 500  | Erreur serveur                         |

### Format des erreurs de validation (422)

```json
{
  "message": "Erreur de validation",
  "errors": {
    "desi_site": ["Le nom du site est obligatoire."]
  }
}
```

### Format des erreurs 404

```json
{
  "message": "Site introuvable"
}
```

### Format des erreurs 401

```json
{
  "message": "Unauthenticated."
}
```

---

## 13. Pagination

Tous les endpoints `index` retournent une structure `meta` :

```json
"meta": {
  "total": 100,
  "page": 1,
  "per_page": 15,
  "dernier_page": 7
}
```

Utilisez `?page=N` pour naviguer entre les pages.

---

## 14. Arborescence de navigation suggérée

```
┌─────────────────────────────────────────────────────┐
│                     Sidebar                          │
├─────────────────────────────────────────────────────┤
│ 📊 Dashboard                                         │
│ 📍 Sites                                             │
│ 🏢 Structures                                        │
│ 👥 Agents                                            │
│ 💻 Équipements                                       │
│ ─────────────────────                                 │
│ 👤 Utilisateurs (Admin only)                         │
│ 📋 Logs d'activité                                   │
│ 📤 Import de données (Admin only)                    │
└─────────────────────────────────────────────────────┘
```

---

## 15. Notes pour l'agent IA frontend React

1. **Stack suggérée** : React 18+ avec React Router v6, Axios pour les appels API, Context API ou Zustand pour la gestion d'état, Ant Design ou MUI pour l'UI.
2. **Intercepteur Axios** : configurer un intercepteur pour attacher le token `Authorization: Bearer <token>` à chaque requête, et gérer les 401 (rediriger vers login).
3. **Stockage du token** : `localStorage` ou `httpOnly cookies`.
4. **Gestion des permissions** : après `GET /api/me`, stocker `roles` et `permissions` dans le contexte global pour conditionner l'affichage des boutons/pages.
5. **Traduction** : tous les messages API sont en français.
6. **Dates** : format ISO 8601 — utiliser `dayjs` ou `date-fns` pour l'affichage.
7. **CSRF** : pas nécessaire (Sanctum API token).
