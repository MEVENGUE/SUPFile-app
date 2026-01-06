# 📚 SUPFile - Documentation Complète

> Documentation technique complète du projet SUPFile - Système de stockage de fichiers cloud sécurisé

## 📋 Table des Matières

1. [Architecture](#architecture)
2. [Fonctionnalités Détaillées](#fonctionnalités-détaillées)
3. [Installation et Configuration](#installation-et-configuration)
4. [Déploiement](#déploiement)
5. [Authentification et OAuth2](#authentification-et-oauth2)
6. [API Reference](#api-reference)
7. [Sécurité](#sécurité)
8. [Dépannage](#dépannage)
9. [Roadmap](#roadmap)

---

## 🏗️ Architecture

### Architecture Globale

SUPFile suit une architecture **client-serveur** moderne avec séparation frontend/backend :

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (Vercel)                    │
│              React + TypeScript + Vite                   │
│                  Port: 3000 (dev)                       │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTPS/REST API
                        │ JWT Authentication
┌───────────────────────▼─────────────────────────────────┐
│                   Backend (Railway)                      │
│                FastAPI + Python 3.11                     │
│                  Port: 8000 (dev)                       │
└───────┬───────────────────────────────┬─────────────────┘
        │                               │
┌───────▼────────┐            ┌────────▼──────────┐
│  PostgreSQL    │            │  Azure Blob        │
│  (Railway)     │            │  Storage           │
│  Métadonnées   │            │  Fichiers binaires │
└────────────────┘            └────────────────────┘
```

### Structure du Projet

```
SUPFile/
├── frontend/                 # Application React
│   ├── src/
│   │   ├── components/       # Composants réutilisables
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Breadcrumbs.tsx
│   │   │   ├── SearchBar.tsx
│   │   │   ├── FileViewer.tsx
│   │   │   ├── ShareModal.tsx
│   │   │   ├── RenameModal.tsx
│   │   │   ├── MoveModal.tsx
│   │   │   ├── Pagination.tsx
│   │   │   ├── ThemeToggle.tsx
│   │   │   └── LoadingSpinner.tsx
│   │   ├── pages/            # Pages de l'application
│   │   │   ├── Login.tsx
│   │   │   ├── Register.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── MyFilesPage.tsx
│   │   │   ├── TrashPage.tsx
│   │   │   ├── SharedFilesPage.tsx
│   │   │   ├── OAuthCallback.tsx
│   │   │   └── AboutPage.tsx
│   │   ├── services/         # Services API
│   │   │   ├── authService.ts
│   │   │   ├── fileService.ts
│   │   │   ├── folderService.ts
│   │   │   └── shareService.ts
│   │   ├── contexts/         # Contextes React
│   │   │   ├── AuthContext.tsx
│   │   │   └── ThemeContext.tsx
│   │   ├── hooks/            # Hooks personnalisés
│   │   │   └── useWebSocket.ts
│   │   └── utils/            # Utilitaires
│   │       └── fileIcons.ts
│   ├── public/               # Fichiers statiques
│   │   ├── sw.js             # Service Worker (PWA)
│   │   └── manifest.json
│   └── package.json
│
├── backend/                  # API FastAPI
│   ├── app/
│   │   ├── api/v1/           # Routes API version 1
│   │   │   ├── auth.py       # Authentification JWT
│   │   │   ├── oauth.py      # OAuth2 (Google, GitHub, Microsoft)
│   │   │   ├── files.py      # Gestion fichiers
│   │   │   ├── folders.py    # Gestion dossiers
│   │   │   ├── share.py      # Partage de fichiers
│   │   │   ├── dashboard.py  # Statistiques
│   │   │   ├── file_history.py
│   │   │   ├── file_comments.py
│   │   │   └── file_versions.py
│   │   ├── core/             # Configuration
│   │   │   ├── config.py      # Settings (Pydantic)
│   │   │   ├── database.py    # SQLAlchemy
│   │   │   ├── security.py     # JWT, bcrypt
│   │   │   └── middleware.py   # CORS, auth
│   │   ├── models/            # Modèles SQLAlchemy
│   │   │   ├── user.py
│   │   │   ├── file.py
│   │   │   ├── folder.py
│   │   │   ├── share_link.py
│   │   │   ├── oauth_account.py
│   │   │   ├── file_history.py
│   │   │   └── file_comment.py
│   │   └── services/         # Services métier
│   │       └── azure_blob.py  # Azure Blob Storage
│   ├── alembic/              # Migrations base de données
│   └── requirements.txt
│
└── docker-compose.yml        # Configuration Docker local
```

### Flux de Données

#### Flux d'Authentification

1. **Inscription/Connexion** :
   ```
   Frontend → POST /api/v1/auth/register|login
   Backend → Vérification credentials
   Backend → Génération JWT (access + refresh)
   Backend → Retour tokens
   Frontend → Stockage tokens (localStorage)
   ```

2. **OAuth2** :
   ```
   Frontend → GET /api/v1/auth/{provider}/authorize
   Backend → Redirection vers provider OAuth
   Provider → Redirection callback avec code
   Backend → Échange code → token
   Backend → Création/compte utilisateur
   Backend → Génération JWT
   Backend → Redirection frontend avec tokens (fragment URL)
   Frontend → Extraction tokens → Stockage
   ```

#### Flux d'Upload de Fichier

1. **Upload** :
   ```
   Frontend → Validation fichier (taille, extension)
   Frontend → POST /api/v1/files/upload (FormData)
   Backend → Validation (taille, extension, permissions)
   Backend → Génération blob_name unique
   Backend → Upload vers Azure Blob Storage
   Backend → Sauvegarde métadonnées PostgreSQL
   Backend → Retour métadonnées fichier
   Frontend → Mise à jour liste fichiers
   ```

2. **Téléchargement** :
   ```
   Frontend → GET /api/v1/files/{id}/download
   Backend → Vérification permissions
   Backend → Récupération fichier Azure
   Backend → Stream vers client
   Frontend → Téléchargement fichier
   ```

#### Flux de Gestion de Dossiers

1. **Création** :
   ```
   Frontend → POST /api/v1/folders
   Backend → Validation nom, parent_id
   Backend → Création dossier PostgreSQL
   Backend → Retour métadonnées dossier
   Frontend → Mise à jour arborescence
   ```

2. **Navigation** :
   ```
   Frontend → GET /api/v1/files?folder_id={id}
   Backend → Filtrage fichiers par folder_id
   Backend → Retour liste fichiers
   Frontend → Affichage avec breadcrumbs
   ```

#### Flux de Corbeille

1. **Suppression (Soft Delete)** :
   ```
   Frontend → DELETE /api/v1/files/{id}
   Backend → Mise à jour deleted_at = timestamp
   Backend → Fichier masqué des listes normales
   Frontend → Mise à jour liste
   ```

2. **Restauration** :
   ```
   Frontend → POST /api/v1/files/{id}/restore
   Backend → Mise à jour deleted_at = NULL
   Backend → Fichier réapparaît dans listes
   Frontend → Mise à jour corbeille + listes
   ```

3. **Suppression définitive** :
   ```
   Frontend → DELETE /api/v1/files/{id}/permanent
   Backend → Suppression fichier Azure Blob
   Backend → Suppression enregistrement PostgreSQL
   Frontend → Mise à jour corbeille
   ```

---

## 🎯 Fonctionnalités Détaillées

### 🔐 Authentification

#### Authentification JWT

- **Inscription** : Email, username, password avec validation
- **Connexion** : Username/password avec génération JWT
- **Tokens** : Access token (30 min) + Refresh token (7 jours)
- **Sécurité** : Hachage bcrypt (12 rounds), validation Pydantic

#### OAuth2 (Google, GitHub, Microsoft)

- **Flux** : Authorization Code Flow
- **Cache** : Protection contre appels multiples (codes OAuth)
- **Redirection** : JavaScript redirect pour éviter ERR_INVALID_REDIRECT
- **Configuration** : Variables d'environnement + URLs de callback

### 📤 Gestion des Fichiers

#### Upload

- **Validation** : Taille max (100MB), extensions autorisées
- **Stockage** : Azure Blob Storage avec nom unique
- **Métadonnées** : PostgreSQL (nom, taille, type, région)
- **Interface** : Drag & drop avec barre de progression

#### Téléchargement

- **Streaming** : Téléchargement direct depuis Azure
- **Permissions** : Vérification propriétaire
- **Headers** : Content-Disposition pour nom original

#### Prévisualisation

- **Types supportés** : Images, PDF, texte
- **URL signée** : SAS URL Azure avec expiration
- **Viewer** : Composant React pour affichage

#### Recherche

- **Critères** : Nom de fichier, type MIME
- **Filtres** : Par dossier, par type
- **Pagination** : skip/limit pour performance

### 📁 Gestion des Dossiers

#### Structure

- **Hiérarchie** : Relations parent-enfant (parent_id)
- **Navigation** : Breadcrumbs automatiques
- **Racine** : folder_id = NULL pour fichiers racine

#### Opérations

- **Création** : Dossiers imbriqués
- **Renommage** : Modification nom
- **Déplacement** : Changement parent_id
- **Suppression** : Soft delete (comme fichiers)

### 🗑️ Gestion de la Corbeille

#### Soft Delete

- **Marquage** : `deleted_at` timestamp
- **Masquage** : Fichiers non visibles dans listes normales
- **Récupération** : Restauration possible

#### Suppression Définitive

- **Processus** : Suppression Azure + PostgreSQL
- **Irréversible** : Confirmation requise
- **Nettoyage** : Libération espace de stockage

### 🔗 Partage de Fichiers

#### Liens Publics

- **Génération** : Token UUID unique
- **Expiration** : Date d'expiration optionnelle
- **Protection** : Mot de passe optionnel
- **Accès** : Sans authentification

#### Flux de Partage

1. Création lien → Génération token
2. Partage URL → `/share/{token}`
3. Accès → Vérification token + expiration + mot de passe
4. Téléchargement → URL signée Azure

### 📊 Dashboard

#### Statistiques

- **Fichiers** : Nombre total, espace utilisé
- **Stockage** : Utilisé / Disponible / Pourcentage
- **Graphiques** : Visualisation espace (Chart.js)

#### Fichiers Récents

- **Tri** : Par date de modification
- **Limite** : 10 fichiers récents
- **Affichage** : Liste avec métadonnées

---

## ⚙️ Installation et Configuration

### Prérequis

- **Python** 3.11+
- **Node.js** 18+
- **Docker** & Docker Compose (recommandé)
- **PostgreSQL** (ou via Docker)
- **Compte Azure** (pour Blob Storage)

### Installation avec Docker

1. **Cloner le projet** :
   ```bash
   git clone https://github.com/MEVENGUE/SUPFile-Vercel-App.git
   cd SUPFile-Vercel-App
   ```

2. **Configurer les variables d'environnement** :
   
   Créer `.env` à la racine :
   ```env
   # Backend
   DATABASE_URL=postgresql://supfile_user:supfile_password@postgres:5432/supfile
   SECRET_KEY=votre-secret-key-minimum-32-caracteres
   JWT_SECRET_KEY=votre-jwt-secret-key-minimum-32-caracteres
   
   # Azure Blob Storage
   AZURE_STORAGE_ACCOUNT_NAME=votre-compte-azure
   AZURE_STORAGE_ACCOUNT_KEY=votre-cle-azure
   AZURE_STORAGE_CONTAINER_NAME=supfile-files
   AZURE_STORAGE_CONNECTION_STRING=votre-connection-string
   
   # Frontend
   VITE_API_URL=http://localhost:8000/api/v1
   ```

3. **Démarrer les services** :
   ```bash
   docker-compose up -d
   ```

4. **Initialiser la base de données** :
   ```bash
   docker exec -it supfile-backend bash
   alembic upgrade head
   ```

5. **Accéder à l'application** :
   - Frontend : http://localhost:3000
   - Backend : http://localhost:8000
   - API Docs : http://localhost:8000/docs

### Installation Manuelle

#### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Créer .env
cp .env.example .env
# Éditer .env avec vos configurations

# Configurer PostgreSQL localement
# Puis exécuter les migrations
alembic upgrade head

# Démarrer le serveur
uvicorn app.main:app --reload
```

#### Frontend

```bash
cd frontend
npm install

# Créer .env
echo "VITE_API_URL=http://localhost:8000/api/v1" > .env

# Démarrer le serveur de développement
npm run dev
```

---

## 🌐 Déploiement

### Frontend (Vercel)

1. **Connecter le dépôt GitHub** :
   - Allez sur [vercel.com](https://vercel.com)
   - Connectez-vous avec GitHub
   - Importez le dépôt `SUPFile-Vercel-App`

2. **Configurer le projet** :
   - **Root Directory** : `frontend`
   - **Framework Preset** : Vite
   - **Build Command** : `npm run build`
   - **Output Directory** : `dist`

3. **Variables d'environnement** :
   ```env
   VITE_API_URL=https://votre-backend.railway.app/api/v1
   VITE_OPENAI_API_KEY=votre-cle-openai (optionnel)
   ```

4. **Déployer** :
   - Cliquez sur "Deploy"
   - Vercel déploie automatiquement

### Backend (Railway)

1. **Créer un projet Railway** :
   - Allez sur [railway.app](https://railway.app)
   - Connectez-vous avec GitHub
   - Créez un nouveau projet

2. **Ajouter PostgreSQL** :
   - Cliquez sur "+ New" > "Database" > "Add PostgreSQL"
   - Notez la variable `DATABASE_URL`

3. **Déployer le backend** :
   - Cliquez sur "+ New" > "GitHub Repo"
   - Sélectionnez `SUPFile-Vercel-App`
   - Railway détecte automatiquement le Dockerfile

4. **Variables d'environnement** :
   ```env
   # Base de données (OBLIGATOIRE)
   DATABASE_URL=${{Postgres.DATABASE_URL}}
   
   # Clés secrètes
   SECRET_KEY=votre-secret-key-32-caracteres
   JWT_SECRET_KEY=votre-jwt-secret-key-32-caracteres
   
   # Azure Blob Storage
   AZURE_STORAGE_ACCOUNT_NAME=votre-compte
   AZURE_STORAGE_ACCOUNT_KEY=votre-cle
   AZURE_STORAGE_CONTAINER_NAME=supfile-files
   AZURE_STORAGE_CONNECTION_STRING=votre-connection-string
   
   # CORS
   CORS_ORIGINS=https://votre-frontend.vercel.app
   
   # OAuth (optionnel)
   OAUTH_GOOGLE_CLIENT_ID=...
   OAUTH_GOOGLE_CLIENT_SECRET=...
   OAUTH_GITHUB_CLIENT_ID=...
   OAUTH_GITHUB_CLIENT_SECRET=...
   OAUTH_MICROSOFT_CLIENT_ID=...
   OAUTH_MICROSOFT_CLIENT_SECRET=...
   OAUTH_REDIRECT_BASE_URL=https://votre-frontend.vercel.app
   OAUTH_CALLBACK_BASE_URL=https://votre-backend.railway.app
   
   # Configuration serveur
   HOST=0.0.0.0
   PORT=${{PORT}}
   DEBUG=False
   APP_ENV=production
   ```

5. **Exécuter les migrations** :
   - Ouvrez la console Railway
   - Exécutez : `alembic upgrade head`

6. **Activer le domaine** :
   - Allez dans "Settings"
   - Activez "Generate Domain"
   - Notez l'URL (ex: `https://supfile-backend.railway.app`)

### Configuration Azure Blob Storage

1. **Créer un Storage Account** :
   - Allez sur [portal.azure.com](https://portal.azure.com)
   - Créez un nouveau Storage Account

2. **Créer un conteneur** :
   - Nom : `supfile-files`
   - Niveau d'accès : Private (recommandé)

3. **Obtenir les clés** :
   - Allez dans "Access keys"
   - Copiez la Connection string
   - Ajoutez dans Railway

---

## 🔐 Authentification et OAuth2

### Authentification JWT

#### Inscription

```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "username": "username",
  "password": "password123",
  "full_name": "John Doe"
}
```

#### Connexion

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "username",
  "password": "password123"
}
```

**Réponse** :
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

### OAuth2 Configuration

#### Google OAuth

1. **Console Google Cloud** :
   - Allez sur [console.cloud.google.com](https://console.cloud.google.com)
   - Créez un projet OAuth 2.0
   - Ajoutez l'URL de callback :
     ```
     https://votre-backend.railway.app/api/v1/auth/google/callback
     ```

2. **Variables Railway** :
   ```env
   OAUTH_GOOGLE_CLIENT_ID=votre-client-id
   OAUTH_GOOGLE_CLIENT_SECRET=votre-secret
   ```

#### GitHub OAuth

1. **GitHub Settings** :
   - Allez sur [github.com/settings/developers](https://github.com/settings/developers)
   - Créez une OAuth App
   - Authorization callback URL :
     ```
     https://votre-backend.railway.app/api/v1/auth/github/callback
     ```

2. **Variables Railway** :
   ```env
   OAUTH_GITHUB_CLIENT_ID=votre-client-id
   OAUTH_GITHUB_CLIENT_SECRET=votre-secret
   ```

#### Microsoft OAuth

1. **Azure Portal** :
   - Allez sur [portal.azure.com](https://portal.azure.com)
   - Azure Active Directory > App registrations
   - Créez une application
   - Ajoutez Redirect URI :
     ```
     https://votre-backend.railway.app/api/v1/auth/microsoft/callback
     ```

2. **Variables Railway** :
   ```env
   OAUTH_MICROSOFT_CLIENT_ID=votre-client-id
   OAUTH_MICROSOFT_CLIENT_SECRET=votre-secret
   ```

3. **Configuration via Azure CLI** :
   ```bash
   az login
   az ad app update --id VOTRE_APP_ID \
     --web-redirect-uris "https://votre-backend.railway.app/api/v1/auth/microsoft/callback"
   ```

### Variables OAuth Requises

```env
# URLs
OAUTH_REDIRECT_BASE_URL=https://votre-frontend.vercel.app
OAUTH_CALLBACK_BASE_URL=https://votre-backend.railway.app

# Google
OAUTH_GOOGLE_CLIENT_ID=...
OAUTH_GOOGLE_CLIENT_SECRET=...

# GitHub
OAUTH_GITHUB_CLIENT_ID=...
OAUTH_GITHUB_CLIENT_SECRET=...

# Microsoft
OAUTH_MICROSOFT_CLIENT_ID=...
OAUTH_MICROSOFT_CLIENT_SECRET=...
```

---

## 📡 API Reference

### Authentification

#### POST `/api/v1/auth/register`
Inscription d'un nouvel utilisateur

**Body** :
```json
{
  "email": "user@example.com",
  "username": "username",
  "password": "password123",
  "full_name": "John Doe"
}
```

#### POST `/api/v1/auth/login`
Connexion utilisateur

**Body** :
```json
{
  "username": "username",
  "password": "password123"
}
```

**Response** :
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

#### GET `/api/v1/auth/{provider}/authorize`
Initie le flux OAuth2 (Google, GitHub, Microsoft)

**Redirection** : Vers le provider OAuth

#### GET `/api/v1/auth/{provider}/callback`
Callback OAuth2 (géré automatiquement)

### Fichiers

#### GET `/api/v1/files`
Liste des fichiers

**Query Parameters** :
- `folder_id` (optional) : ID du dossier parent
- `skip` (optional) : Pagination offset
- `limit` (optional) : Nombre de résultats (max 1000)

#### POST `/api/v1/files/upload`
Upload d'un fichier

**Body** : FormData avec champ `file` et `folder_id` (optionnel)

**Response** :
```json
{
  "id": 1,
  "filename": "document.pdf",
  "original_filename": "document.pdf",
  "file_size": 1024,
  "content_type": "application/pdf",
  "created_at": "2024-01-01T00:00:00",
  "upload_region": "eastus"
}
```

#### GET `/api/v1/files/{id}/download`
Téléchargement d'un fichier

**Headers** : `Authorization: Bearer {token}`

#### GET `/api/v1/files/{id}/preview`
URL de prévisualisation (SAS URL Azure)

#### DELETE `/api/v1/files/{id}`
Suppression (soft delete - corbeille)

#### POST `/api/v1/files/{id}/restore`
Restauration depuis la corbeille

#### DELETE `/api/v1/files/{id}/permanent`
Suppression définitive

#### GET `/api/v1/files/trash`
Liste des fichiers dans la corbeille

#### GET `/api/v1/files/search`
Recherche de fichiers

**Query Parameters** :
- `q` : Terme de recherche
- `content_type` (optional) : Filtrer par type
- `folder_id` (optional) : Filtrer par dossier

### Dossiers

#### GET `/api/v1/folders`
Liste des dossiers

**Query Parameters** :
- `parent_id` (optional) : ID du dossier parent
- `skip`, `limit` : Pagination

#### POST `/api/v1/folders`
Création d'un dossier

**Body** :
```json
{
  "name": "Mon Dossier",
  "parent_id": null
}
```

#### PATCH `/api/v1/folders/{id}/rename`
Renommage d'un dossier

**Body** :
```json
{
  "name": "Nouveau Nom"
}
```

#### PATCH `/api/v1/folders/{id}/move`
Déplacement d'un dossier

**Body** :
```json
{
  "parent_id": 2
}
```

#### DELETE `/api/v1/folders/{id}`
Suppression d'un dossier (soft delete)

#### GET `/api/v1/folders/trash`
Liste des dossiers dans la corbeille

### Partage

#### POST `/api/v1/share`
Création d'un lien de partage

**Body** :
```json
{
  "file_id": 1,
  "expires_at": "2024-12-31T23:59:59",
  "password": "optional_password"
}
```

**Response** :
```json
{
  "token": "uuid-token",
  "url": "https://app.com/share/uuid-token"
}
```

#### GET `/api/v1/share/{token}`
Accès à un fichier partagé

**Query Parameters** :
- `password` (optional) : Mot de passe si requis

### Dashboard

#### GET `/api/v1/dashboard/stats`
Statistiques du dashboard

**Response** :
```json
{
  "total_files": 100,
  "total_size": 1073741824,
  "storage_used": 536870912,
  "storage_available": 107374182400,
  "storage_percentage": 0.5
}
```

---

## 🔒 Sécurité

### Mesures Implémentées

1. **Authentification** :
   - JWT tokens sécurisés avec expiration
   - Refresh tokens pour renouvellement
   - Hachage bcrypt (12 rounds)

2. **Autorisation** :
   - Contrôle d'accès par utilisateur
   - Vérification des permissions sur chaque requête
   - Middleware d'authentification

3. **Validation** :
   - Validation des entrées (Pydantic)
   - Validation des fichiers (taille, extension)
   - Protection injection SQL (SQLAlchemy)

4. **Réseau** :
   - HTTPS (en production)
   - CORS configuré
   - Headers de sécurité

5. **Stockage** :
   - Secrets dans variables d'environnement
   - Pas de secrets en clair dans le code
   - Azure Blob Storage sécurisé

### Bonnes Pratiques

- ✅ Aucun secret en clair dans le code
- ✅ Validation de toutes les entrées
- ✅ Protection contre injection SQL
- ✅ HTTPS partout en production
- ✅ CORS restreint aux domaines autorisés

---

## 🐛 Dépannage

### Problème : Le frontend ne peut pas se connecter au backend

**Solutions** :
- Vérifier `VITE_API_URL` dans Vercel
- Vérifier `CORS_ORIGINS` dans Railway
- Vérifier que le backend est accessible

### Problème : Erreur de base de données

**Solutions** :
- Vérifier `DATABASE_URL` dans Railway
- Exécuter les migrations : `alembic upgrade head`
- Vérifier les logs Railway

### Problème : Erreur Azure Blob Storage

**Solutions** :
- Vérifier toutes les variables Azure
- Vérifier que le conteneur existe
- Vérifier les permissions du Storage Account

### Problème : OAuth ne fonctionne pas

**Solutions** :
- Vérifier les URLs de callback dans les providers
- Vérifier les variables d'environnement OAuth
- Vérifier les logs backend pour les erreurs

### Problème : Page blanche après actualisation

**Solutions** :
- Vider le cache du navigateur (Ctrl+Shift+R)
- Vérifier le Service Worker (DevTools > Application)
- Désinscrire le Service Worker si nécessaire

---

## 🗺️ Roadmap

### Phase 1 : Fondations ✅
- Authentification JWT
- Upload/téléchargement
- Gestion fichiers de base

### Phase 2 : Organisation ✅
- Gestion dossiers
- Navigation breadcrumbs
- Recherche

### Phase 3 : Partage ✅
- Liens publics
- Partage sécurisé

### Phase 4 : Améliorations UX ✅
- Thème clair/sombre
- Animations
- Responsive design

### Phase 5 : Fonctionnalités Avancées ✅
- Historique modifications
- Commentaires
- OAuth2 (Google, GitHub, Microsoft)
- Corbeille avec restauration

### Phase 6 : À Venir
- Application mobile (React Native)
- Chiffrement fichiers
- Versioning complet
- Synchronisation temps réel (WebSocket)
- Collaboration en temps réel

---

**Dernière mise à jour** : Janvier 2025
