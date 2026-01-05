# 🚀 SUPFile - Secure Cloud File Storage System

<div align="center">

![Version](https://img.shields.io/badge/version-2.0-blue.svg)
![License](https://img.shields.io/badge/license-Academic-red.svg)
![Python](https://img.shields.io/badge/python-3.11+-blue.svg)
![React](https://img.shields.io/badge/react-18.2+-61dafb.svg)
![FastAPI](https://img.shields.io/badge/fastapi-0.104+-009688.svg)

**Système de stockage de fichiers cloud sécurisé** - Projet académique SUPINFO

[🌐 Application Live](https://supfile-webapp.vercel.app) • [📚 Documentation Complète](./Docs_Projet/DOCUMENTATION.md) • [📊 Diagrammes](./Images)

</div>

---

## 📋 Table des Matières

- [Vue d'ensemble](#-vue-densemble)
- [Fonctionnalités](#-fonctionnalités)
- [Architecture](#-architecture)
- [Installation](#-installation)
- [Déploiement](#-déploiement)
- [Documentation](#-documentation)
- [Technologies](#-technologies)
- [Contributeurs](#-contributeurs)

---

## 🎯 Vue d'ensemble

**SUPFile** est une application web moderne de stockage de fichiers cloud sécurisée, inspirée de Dropbox. Elle permet aux utilisateurs de stocker, organiser, partager et gérer leurs fichiers de manière sécurisée dans le cloud.

### ✨ Points Forts

- 🔐 **Authentification multi-providers** : JWT, OAuth2 (Google, GitHub, Microsoft)
- 📁 **Gestion complète** : Fichiers, dossiers, corbeille, recherche
- 🔗 **Partage sécurisé** : Liens publics avec expiration et mot de passe
- 📊 **Dashboard interactif** : Statistiques et visualisations
- 🌍 **Multi-régions** : Stockage Azure Blob Storage
- 📱 **Responsive** : Interface adaptative mobile/desktop
- 🎨 **Thème clair/sombre** : Personnalisation de l'interface

---

## 🎨 Fonctionnalités

### 🔐 Authentification

- ✅ **Inscription/Connexion** : Email et mot de passe avec validation
- ✅ **OAuth2** : Connexion via Google, GitHub, Microsoft
- ✅ **JWT Tokens** : Access token + Refresh token sécurisés
- ✅ **Sessions persistantes** : Gestion automatique des tokens
- ✅ **Sécurité** : Hachage bcrypt (12 rounds), protection CSRF

**Fonctionnement** :
1. L'utilisateur s'inscrit ou se connecte via OAuth
2. Le backend génère des tokens JWT (access + refresh)
3. Les tokens sont stockés dans le localStorage (frontend)
4. Chaque requête API inclut le token dans le header `Authorization`
5. Le backend valide le token et identifie l'utilisateur

### 📤 Gestion des Fichiers

- ✅ **Upload** : Drag & drop, barre de progression, validation
- ✅ **Téléchargement** : Téléchargement individuel ou en lot
- ✅ **Prévisualisation** : Images, PDF, texte directement dans le navigateur
- ✅ **Recherche** : Recherche par nom, type, contenu
- ✅ **Renommage** : Modification du nom de fichier
- ✅ **Déplacement** : Déplacement entre dossiers
- ✅ **Métadonnées** : Taille, date, type, région de stockage

**Fonctionnement** :
1. **Upload** : Fichier → Validation (taille, extension) → Azure Blob Storage → Métadonnées PostgreSQL
2. **Téléchargement** : Requête → Vérification permissions → Récupération Azure → Stream vers client
3. **Prévisualisation** : Génération URL signée Azure → Affichage dans viewer
4. **Recherche** : Requête SQL avec filtres → Retour liste filtrée

### 📁 Gestion des Dossiers

- ✅ **Création** : Dossiers imbriqués avec arborescence
- ✅ **Navigation** : Breadcrumbs, navigation intuitive
- ✅ **Organisation** : Déplacement, renommage, suppression
- ✅ **Structure** : Arborescence complète avec relations parent-enfant

**Fonctionnement** :
1. Création d'un dossier → Enregistrement dans PostgreSQL avec `parent_id`
2. Navigation → Filtrage des fichiers par `folder_id`
3. Breadcrumbs → Reconstruction du chemin depuis la racine
4. Déplacement → Mise à jour du `folder_id` des fichiers/dossiers enfants

### 🗑️ Gestion de la Corbeille

- ✅ **Soft Delete** : Suppression réversible (marquage `deleted_at`)
- ✅ **Restauration** : Récupération des fichiers supprimés
- ✅ **Suppression définitive** : Suppression du fichier Azure + base de données
- ✅ **Vue dédiée** : Page corbeille avec filtres

**Fonctionnement** :
1. **Suppression** : `deleted_at` = timestamp → Fichier masqué des listes normales
2. **Corbeille** : Filtrage `WHERE deleted_at IS NOT NULL`
3. **Restauration** : `deleted_at` = NULL → Fichier réapparaît
4. **Suppression définitive** : Suppression Azure Blob + Suppression DB

### 🔗 Partage de Fichiers

- ✅ **Liens publics** : Génération de tokens uniques
- ✅ **Expiration** : Liens avec date d'expiration
- ✅ **Protection** : Mot de passe optionnel
- ✅ **Accès anonyme** : Téléchargement sans authentification

**Fonctionnement** :
1. Génération d'un token UUID unique
2. Création d'un `ShareLink` avec expiration et mot de passe (optionnel)
3. URL publique : `/share/{token}`
4. Accès : Vérification token + expiration + mot de passe si requis
5. Téléchargement via URL signée Azure

### 📊 Dashboard

- ✅ **Statistiques** : Nombre de fichiers, espace utilisé/disponible
- ✅ **Graphiques** : Visualisation de l'espace de stockage
- ✅ **Fichiers récents** : Liste des derniers fichiers modifiés
- ✅ **Activité** : Vue d'ensemble de l'activité utilisateur

**Fonctionnement** :
1. Agrégation SQL : `COUNT`, `SUM(file_size)` par utilisateur
2. Calcul du pourcentage : `(storage_used / storage_available) * 100`
3. Fichiers récents : `ORDER BY updated_at DESC LIMIT 10`
4. Affichage avec graphiques React (Chart.js)

### 💬 Commentaires et Historique

- ✅ **Commentaires** : Commentaires sur fichiers et dossiers
- ✅ **Historique** : Historique des modifications de fichiers
- ✅ **Versions** : Gestion des versions de fichiers (en développement)

### 🎨 Ressources Visuelles

L'application utilise des ressources visuelles organisées dans le dossier `frontend/public/` :

**Images principales** (racine de `public/`) :
- `logo.jpg` : Logo principal SUPFile (utilisé dans login, sidebar, header)
- `Fichier.jpg` : Icône générique pour les fichiers (fallback pour tous les types)
- `Espace de Stockage.jpg` : Icône pour l'upload et le stockage

**Dossier `Images app/`** (ressources spécifiques) :

**Organisation** :
- Les images principales sont à la racine de `public/` pour un accès direct
- Les ressources spécifiques sont dans `public/Images app/` pour une meilleure organisation
- Toutes les images sont servies statiquement via Vite et accessibles via `/nom-image.jpg` ou `/Images app/nom-image.jpg`

---

## 🏗️ Architecture

### Structure Globale

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
├── Docs_Projet/              # Documentation du projet
│   ├── DOCUMENTATION.md      # Documentation complète
│   └── Images/               # Diagrammes d'architecture (PNG)
│       ├── 1-Architecture Globale.png
│       ├── 2-Flux d'Authentification - *.png
│       ├── 3-Flux OAuth2 Complet -*.png
│       ├── Flux d'Upload de Fichier.png
│       ├── Flux de Gestion de Dossiers - *.png
│       ├── Flux de Corbeille - *.png
│       ├── Flux de Partage.png
│       ├── Modèle de Données - *.png
│       └── ...                # Autres schémas d'architecture
│
├── frontend/                 # Application React
│   ├── src/
│   │   ├── components/       # Composants réutilisables
│   │   ├── pages/            # Pages de l'application
│   │   ├── services/         # Services API
│   │   ├── contexts/         # Contextes React (Auth, Theme)
│   │   ├── hooks/            # Hooks personnalisés
│   │   └── utils/            # Utilitaires
│   ├── public/               # Fichiers statiques + Service Worker
│   │   ├── Images app/       # Ressources visuelles de l'application
│   │   │   ├── Partagés.jpg           # Icône fichiers partagés
│   │   │   ├── SUPINFO Paris Logo.png # Logo SUPINFO
│   │   │   └── ...                     # Autres icônes (notifications, paramètres, etc.)
│   │   ├── logo.jpg          # Logo principal SUPFile
│   │   ├── Fichier.jpg       # Icône fichier générique
│   │   ├── Espace de Stockage.jpg # Icône upload/storage
│   │   ├── sw.js             # Service Worker (PWA)
│   │   └── manifest.json     # Manifest PWA
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
│   │   │   └── ...
│   │   ├── core/             # Configuration
│   │   │   ├── config.py     # Settings (Pydantic)
│   │   │   ├── database.py   # SQLAlchemy
│   │   │   ├── security.py    # JWT, bcrypt
│   │   │   └── middleware.py # CORS, auth
│   │   ├── models/           # Modèles SQLAlchemy
│   │   │   ├── user.py
│   │   │   ├── file.py
│   │   │   ├── folder.py
│   │   │   └── ...
│   │   └── services/         # Services métier
│   │       └── azure_blob.py # Azure Blob Storage
│   ├── alembic/             # Migrations base de données
│   └── requirements.txt
│
└── docker-compose.yml        # Configuration Docker local
```

---

## 🚀 Installation

### Prérequis

- **Python** 3.11+
- **Node.js** 18+
- **Docker** & Docker Compose (recommandé)
- **PostgreSQL** (ou via Docker)
- **Compte Azure** (pour Blob Storage)

### Installation Rapide avec Docker en Local

```bash
# 1. Cloner le projet
git clone https://github.com/MEVENGUE/SUPFile......git
cd SUPFile-Vercel-App

# 2. Créer le fichier .env
cp .env.example .env
# Éditer .env avec vos configurations

# 3. Démarrer les services
docker-compose up -d

# 4. Initialiser la base de données
docker exec -it supfile-backend bash
alembic upgrade head

# 5. Accéder à l'application
# Frontend: http://localhost:3000
# Backend: http://localhost:8000
# API Docs: http://localhost:8000/docs
```

### Installation Manuelle

Voir [DOCUMENTATION.md](./Docs_Projet/DOCUMENTATION.md#installation) pour les instructions détaillées.

---

## 🌐 Déploiement

### Frontend (Vercel)

1. Connectez votre dépôt GitHub à Vercel
2. Configurez le **Root Directory** : `frontend`
3. Ajoutez les variables d'environnement :
   ```env
   VITE_API_URL=https://votre-backend.railway.app/api/v1
   ```
4. Déployez !

### Backend (Railway)

1. Créez un projet Railway
2. Ajoutez PostgreSQL
3. Déployez le backend depuis GitHub
4. Configurez les variables d'environnement (voir [DOCUMENTATION.md](./Docs_Projet/DOCUMENTATION.md#déploiement))
5. Exécutez les migrations : `alembic upgrade head`

📖 **Guide complet** : [DOCUMENTATION.md](./Docs_Projet/DOCUMENTATION.md#déploiement)

---

## 📚 Documentation

- **[📖 Documentation Complète](./Docs_Projet/DOCUMENTATION.md)** : Architecture, API, sécurité, déploiement
- **[📊 Diagrammes Mermaid](./Images)** : Diagrammes d'architecture et de flux
- **[🖼️ Images d'Architecture](./Docs_Projet/Images/)** : Schémas visuels des diagrammes (PNG)
- **[🔧 Configuration OAuth](./Docs_Projet/DOCUMENTATION.md#oauth2)** : Configuration Google, GitHub, Microsoft

### 📊 Diagrammes d'Architecture

Le dossier `Docs_Projet/Images/` contient les schémas visuels (PNG) des diagrammes Mermaid du projet. Ces images sont des exports des diagrammes Mermaid présents dans [DIAGRAMS.md](./Images) et peuvent être utilisées pour la documentation, les présentations ou la compréhension visuelle de l'architecture.

#### 🏗️ Architecture Globale

Vue d'ensemble du système avec tous les composants (Frontend, Backend, Base de données, Stockage cloud, OAuth providers).

![Architecture Globale](./Images/1-Architecture%20Globale.png)

#### 🔐 Flux d'Authentification

**Inscription** : Processus complet d'inscription d'un nouvel utilisateur.

![Flux d'Authentification - Inscription](./Images/2-Flux%20d'Authentification%20-%20Inscription.png)

**Authentification JWT** : Processus de connexion et génération de tokens JWT.

![Flux d'Authentification - Authentification JWT](./Images/2-Flux%20d'Authentification%20-%20Authentification%20JWT.png)

#### 🔑 Flux OAuth2 Complet

Flux complet pour les trois providers OAuth2 (Google, GitHub, Microsoft) avec gestion des tokens temporaires.

![Flux OAuth2 Complet](./Images/3-Flux%20OAuth2%20Complet%20-Google_GitHub_Microsoft.png)    

**Séquence OAuth2 Détaillée - Google** : Séquence détaillée du flux OAuth2 pour Google.

![Séquence OAuth2 Détaillée - Google](./Images/Séquence%20OAuth2%20Détaillée%20-%20Google%20OAuth2.png)  

#### 📤 Flux d'Upload de Fichier

Processus complet de téléchargement de fichiers : validation, upload vers Azure Blob Storage, enregistrement des métadonnées.

![Flux d'Upload de Fichier](./Images/Flux%20d'Upload%20de%20Fichier.png)

**Validation de Fichier** : Processus de validation des fichiers avant upload.

![Validation de Fichier](./Images/Validation%20de%20Fichier.png)

#### 📁 Flux de Gestion de Dossiers

**Création et Navigation** : Processus de création de dossiers et navigation dans l'arborescence.

![Flux de Gestion de Dossiers - Création et Navigation](./Images/Flux%20de%20Gestion%20de%20Dossiers%20-%20Création%20et%20Navigation.png)

**Breadcrumbs** : Système de navigation avec fil d'Ariane.

![Flux de Gestion de Dossiers - Breadcrumbs](./Images/Flux%20de%20Gestion%20de%20Dossiers%20-%20Breadcrumbs.png)

#### 🗑️ Flux de Corbeille

**États d'un Fichier** : Cycle de vie d'un fichier (actif, supprimé, restauré, supprimé définitivement).

![Flux de Corbeille - États d'un Fichier](./Images/Flux%20de%20Corbeille%20-%20%20États%20d'un%20Fichier.png)

**Soft Delete et Restauration** : Processus de suppression réversible et restauration.

![Flux de Corbeille - Soft Delete et Restauration](./Images/Flux%20de%20Corbeille%20-%20Soft%20Delete%20et%20Restauration.png)

#### 🔗 Flux de Partage

Processus de partage de fichiers avec génération de liens publics, expiration et protection par mot de passe.

![Flux de Partage](./Images/Flux%20de%20Partage.png)

**Modèle de Partage** : Modèle de données pour le système de partage.

![Modèle de Partage](./Images/Modèle%20de%20Partage.png)

#### 💾 Architecture de Stockage

Architecture multi-niveaux du stockage (PostgreSQL pour métadonnées, Azure Blob Storage pour fichiers binaires).

![Architecture de Stockage](./Images/Architecture%20de%20Stockage%20-%20Stockage%20Multi-Niveaux.png)

#### 🗄️ Modèle de Données

**Schéma Entité-Relation (ERD)** : Modèle complet de la base de données avec toutes les relations.

![Modèle de Données - Schéma Entité-Relation](./Images/Modèle%20de%20Données%20-%20Schéma%20Entité-Relation.png "Modèle de Données - Schéma Entité-Relation")

#### 🔒 Sécurité

**Protection contre les Codes Dupliqués** : Mécanisme de protection contre la réutilisation de codes OAuth2.

![Protection contre les Codes Dupliqués](./Images/Protection%20contre%20les%20Codes%20Dupliqués.png "Protection contre les Codes Dupliqués")

---

## 🛠️ Technologies

### Frontend
- **React** 18.2.0 - Bibliothèque UI
- **TypeScript** 5.3.3 - Typage statique
- **Vite** 5.0.8 - Build tool ultra-rapide
- **React Router** 6.20.0 - Navigation
- **React Query** 5.12.2 - Gestion d'état serveur
- **Axios** 1.6.2 - Requêtes HTTP
- **React Dropzone** 14.2.3 - Upload drag & drop
- **React Toastify** 9.1.3 - Notifications

### Backend
- **FastAPI** 0.104.1 - Framework Python moderne
- **Python** 3.11 - Langage de programmation
- **SQLAlchemy** 2.0.23 - ORM
- **Alembic** 1.12.1 - Migrations
- **PostgreSQL** 15 - Base de données
- **Azure Blob Storage** 12.19.0 - Stockage fichiers
- **Python-JOSE** 3.3.0 - JWT
- **Passlib** + **Bcrypt** - Hachage mots de passe
- **Pydantic** 2.5.0 - Validation

### Infrastructure
- **Docker** & **Docker Compose** - Containerisation
- **Vercel** - Hébergement frontend
- **Railway** - Hébergement backend
- **Azure Blob Storage** - Stockage cloud

---

## 👥 Contributeurs

- **MEVENGUE Franck** - Développeur principal
- **Nadia Loukdache** - Co-développeuse

---

## 📄 Licence

Projet académique SUPINFO - Tous droits réservés

---

<div align="center">

**Fait avec ❤️ par l'équipe SUPFile**

[🌐 Application Live](https://supfile-webapp.vercel.app) • [📚 Documentation](./Docs_Projet/DOCUMENTATION.md) • [📊 Diagrammes](./Images)

</div>
