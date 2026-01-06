# 🚀 SUPFile - Secure Cloud File Storage System

<div align="center">

![Version](https://img.shields.io/badge/version-2.0-blue.svg)
![License](https://img.shields.io/badge/license-Academic-red.svg)
![Python](https://img.shields.io/badge/python-3.11+-blue.svg)
![React](https://img.shields.io/badge/react-18.2+-61dafb.svg)
![FastAPI](https://img.shields.io/badge/fastapi-0.104+-009688.svg)

**Système de stockage de fichiers cloud sécurisé** - Projet académique SUPINFO

[🌐 Application Live](https://supfile-webapp.vercel.app) • [📚 Documentation](./Docs_Projet/) • [📄 Guide PDF](./Docs_Projet/Guide%20Dessin%20SUPFile_Architecture_and_Security.pdf) • [📊 Diagrammes](./Images)

</div>

🌍 Vos fichiers toujours à portée de main : que vous soyez sur PC 💻 ou sur mobile 📱, SUPFile vous offre une expérience continue, sécurisée et intuitive, où que la vie vous mène.

### 🖼️ Aperçu de l'Application

<div align="center">

**Interface SUPFile**

*Vue d'ensemble simplifiée de l'architecture SUPFile*

</div>

<img width="1141" height="932" alt="Capture d&#39;écran 2026-01-04 231103" src="https://github.com/user-attachments/assets/10434822-916a-4dab-9656-456d0275edd8" />

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

> 📄 **Guide Visuel** : Pour une explication détaillée de l'architecture et de la sécurité de SUPFile sous forme de dessins, consultez le [Guide Dessin SUPFile](./Docs_Projet/Guide%20Dessin%20SUPFile_Architecture_and_Security.pdf).

### 📸 Interface Utilisateur

<div align="center">

![Interface Dashboard](./Images/Graphique%20Projet.png)

*Dashboard interactif avec statistiques et visualisations*

</div>

<img width="1893" height="992" alt="Capture d&#39;écran 2026-01-04 231411" src="https://github.com/user-attachments/assets/956fac15-0330-4e25-9a1c-b6e57627d762" />



### ✨ Points Forts


<img width="1340" height="730" alt="image" src="https://github.com/user-attachments/assets/0e0406f2-f391-4863-9b79-c175e0a5f0ee" />


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

   <img width="1904" height="988" alt="image" src="https://github.com/user-attachments/assets/e02bae95-f035-436f-a5d0-26b7e6cebade" />


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

   <img width="1909" height="995" alt="image" src="https://github.com/user-attachments/assets/7863f258-000b-4f21-b386-84a1b8c9551e" />


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

   <img width="1919" height="986" alt="image" src="https://github.com/user-attachments/assets/d6f9a306-6573-4221-8eab-46ac9e982b19" />


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


   <img width="1919" height="988" alt="image" src="https://github.com/user-attachments/assets/50141f41-bc6f-4c58-a7ea-e3c0014a3075" />


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

<div align="center">

![Architecture Globale SUPFile](./Images/1-Architecture%20Globale.png)

*Architecture complète du système SUPFile*

</div>

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
│   ├── README.md             # Documentation technique complète
│   ├── Guide Dessin SUPFile_Architecture_and_Security.pdf  # Guide visuel
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

Voir [Documentation Technique](./Docs_Projet/README.md#-installation-et-configuration) pour les instructions détaillées.

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
4. Configurez les variables d'environnement (voir [Documentation Technique](./Docs_Projet/README.md#-déploiement))
5. Exécutez les migrations : `alembic upgrade head`

📖 **Guide complet** : [Documentation Technique](./Docs_Projet/README.md#-déploiement)

---

## 📚 Documentation

La documentation complète du projet se trouve dans le dossier **[`Docs_Projet/`](./Docs_Projet/)** :

### 📁 Contenu du Dossier Documentation

- **[📖 README.md](./Docs_Projet/README.md)** : Documentation technique complète expliquant le projet, l'architecture, les fonctionnalités, l'installation, le déploiement, l'API, la sécurité, et tous les diagrammes de flux détaillés

- **[📄 Guide Dessin SUPFile](./Docs_Projet/Guide%20Dessin%20SUPFile_Architecture_and_Security.pdf)** : Guide visuel expliquant l'architecture et la sécurité de l'application SUPFile sous forme de dessins et schémas

### 📊 Ressources Complémentaires

- **[📊 Diagrammes d'Architecture](./Images/)** : Tous les diagrammes d'architecture et de flux (PNG) disponibles à la racine du projet
- **[🔧 Configuration OAuth](./Docs_Projet/README.md#-authentification-et-oauth2)** : Guide de configuration OAuth2 pour Google, GitHub, Microsoft

### 📊 Diagrammes d'Architecture

> 💡 **Note** : Pour une documentation technique complète avec tous les diagrammes détaillés, consultez le [README de documentation](./Docs_Projet/README.md) et le [Guide Dessin SUPFile](./Docs_Projet/Guide%20Dessin%20SUPFile_Architecture_and_Security.pdf).

Le dossier `Images/` contient les schémas visuels (PNG) des diagrammes d'architecture du projet. Ces images illustrent les différents flux et processus de l'application.

#### 🏗️ Architecture Globale

Vue d'ensemble du système avec tous les composants (Frontend, Backend, Base de données, Stockage cloud, OAuth providers).

<div align="center">

![Architecture Globale](./Images/1-Architecture%20Globale.png)

</div>

#### 🔐 Flux d'Authentification

**Inscription** : Processus complet d'inscription d'un nouvel utilisateur.

<div align="center">

![Flux d'Authentification - Inscription](./Images/2-Flux%20d'Authentification%20-%20Inscription.png)

</div>

**Authentification JWT** : Processus de connexion et génération de tokens JWT.

<div align="center">

![Flux d'Authentification - Authentification JWT](./Images/2-Flux%20d'Authentification%20-%20Authentification%20JWT.png)

</div>

#### 🔑 Flux OAuth2 Complet

Flux complet pour les trois providers OAuth2 (Google, GitHub, Microsoft) avec gestion des tokens temporaires.

<div align="center">

![Flux OAuth2 Complet](./Images/3-Flux%20OAuth2%20Complet%20-Google_GitHub_Microsoft.png)

</div>

**Séquence OAuth2 Détaillée - Google** : Séquence détaillée du flux OAuth2 pour Google.

<div align="center">

![Séquence OAuth2 Détaillée - Google](./Images/Séquence%20OAuth2%20Détaillée%20-%20Google%20OAuth2.png)

</div>  

#### 📤 Flux d'Upload de Fichier

Processus complet de téléchargement de fichiers : validation, upload vers Azure Blob Storage, enregistrement des métadonnées.

<div align="center">

![Flux d'Upload de Fichier](./Images/Flux%20d'Upload%20de%20Fichier.png)

</div>

**Validation de Fichier** : Processus de validation des fichiers avant upload.

<div align="center">

![Validation de Fichier](./Images/Validation%20de%20Fichier.png)

</div>

#### 📁 Flux de Gestion de Dossiers

**Création et Navigation** : Processus de création de dossiers et navigation dans l'arborescence.

<div align="center">

![Flux de Gestion de Dossiers - Création et Navigation](./Images/Flux%20de%20Gestion%20de%20Dossiers%20-%20Création%20et%20Navigation.png)

</div>

**Breadcrumbs** : Système de navigation avec fil d'Ariane.

<div align="center">

![Flux de Gestion de Dossiers - Breadcrumbs](./Images/Flux%20de%20Gestion%20de%20Dossiers%20-%20Breadcrumbs.png)

</div>

#### 🗑️ Flux de Corbeille

**États d'un Fichier** : Cycle de vie d'un fichier (actif, supprimé, restauré, supprimé définitivement).

<div align="center">

![Flux de Corbeille - États d'un Fichier](./Images/Flux%20de%20Corbeille%20-%20%20États%20d'un%20Fichier.png)

</div>

**Soft Delete et Restauration** : Processus de suppression réversible et restauration.

<div align="center">

![Flux de Corbeille - Soft Delete et Restauration](./Images/Flux%20de%20Corbeille%20-%20Soft%20Delete%20et%20Restauration.png)

</div>

#### 🔗 Flux de Partage

Processus de partage de fichiers avec génération de liens publics, expiration et protection par mot de passe.

<div align="center">

![Flux de Partage](./Images/Flux%20de%20Partage.png)

</div>

**Modèle de Partage** : Modèle de données pour le système de partage.

<div align="center">

![Modèle de Partage](./Images/Modèle%20de%20Partage.png)

</div>

#### 💾 Architecture de Stockage

Architecture multi-niveaux du stockage (PostgreSQL pour métadonnées, Azure Blob Storage pour fichiers binaires).

<div align="center">

![Architecture de Stockage](./Images/Architecture%20de%20Stockage%20-%20Stockage%20Multi-Niveaux.png)

</div>

#### 🗄️ Modèle de Données

**Schéma Entité-Relation (ERD)** : Modèle complet de la base de données avec toutes les relations.

<div align="center">

![Modèle de Données - Schéma Entité-Relation](./Images/Modèle%20de%20Données%20-%20Schéma%20Entité-Relation.png "Modèle de Données - Schéma Entité-Relation")

</div>

#### 🔒 Sécurité

**Protection contre les Codes Dupliqués** : Mécanisme de protection contre la réutilisation de codes OAuth2.

<div align="center">

![Protection contre les Codes Dupliqués](./Images/Protection%20contre%20les%20Codes%20Dupliqués.png "Protection contre les Codes Dupliqués")

</div>

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

[🌐 Application Live](https://supfile-webapp.vercel.app) • [📚 Documentation](./Docs_Projet/) • [📄 Guide PDF](./Docs_Projet/Guide%20Dessin%20SUPFile_Architecture_and_Security.pdf) • [📊 Diagrammes](./Images)

</div>
