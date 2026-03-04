# 🟠 TICKETING VEDEM V1.02

> Application web de gestion des incidents IT — fichier HTML autonome, sans installation.

---

## 📋 Présentation

**TICKETING VEDEM V1.02** est une plateforme de ticketing interne permettant de centraliser, suivre et résoudre les incidents informatiques et les réclamations clients de l'organisation VEDEM.

L'application fonctionne comme un **fichier HTML unique** (`IncidentDesk.html`) — aucun serveur, aucune base de données externe, aucune installation requise. Il suffit de l'ouvrir dans un navigateur moderne.

---

## 🚀 Démarrage rapide

```bash
# Aucune installation nécessaire.
# Ouvrir simplement le fichier dans un navigateur :
open IncidentDesk.html
```

### Comptes de démonstration

| Rôle | Login | Mot de passe | Accès |
|------|-------|-------------|-------|
| **S-Admin** | `SADMIN` | `0000` | Accès total — gère tous les comptes |
| **Admin** | `ADMIN` | `1234` | Tickets + utilisateurs (hors S-Admin) |
| **User** | `USER01` | `User@2024` | Crée et suit ses propres tickets |
| **Reader** | `READER` | `9999` | Lecture seule + commentaires |

---

## ✨ Fonctionnalités

### 🎫 Gestion des tickets
- Création de tickets **incident interne** ou **réclamation client**
- 9 statuts : Reçu → Ouvert → Pris en charge → En cours → Résolu → Fermé (+ Non ouvert, Ajourné, Abandonné)
- 3 niveaux de priorité avec **SLA visuels** en temps réel (vert / orange / rouge)
- Timeline de traçabilité complète sur chaque ticket
- Ajout de commentaires horodatés par rôle

### 👥 Gestion des utilisateurs
- Création de comptes avec 4 rôles distincts
- Validation du mot de passe en temps réel (score de force 1–5)
- Activation / désactivation / suppression des comptes
- Permissions granulaires : accès rapports, export CSV/PDF

### 🚨 Alertes & Escalades automatiques
- **Alerte toutes les 10mn** si un ticket reste non ouvert
- **Escalade après 2h** : Email + WhatsApp automatiques aux responsables
- Bannière d'escalade visible sur le tableau de bord (Admin/S-Admin)

### 📊 Rapports & Analyses
- Graphiques temps réel : donut statuts, barres priorités, courbe mensuelle (Chart.js)
- Filtres par période, service, priorité
- Export CSV et PDF (selon permissions)

### 📋 Journal d'activité
- Traçabilité de toutes les actions : connexions, modifications, créations, suppressions
- Horodatage et attribution par utilisateur/rôle
- Accessible aux Admin et S-Admin uniquement

---

## 🏗️ Architecture technique

```
IncidentDesk.html
├── <style>          CSS variables + thème orangé VEDEM
├── #login           Écran de connexion
├── #app
│   ├── .topbar      Barre de navigation (nom app + rôle + user)
│   ├── .sidebar     Menu dynamique selon rôle
│   └── .content     Zone de rendu des pages (générée par JS)
├── #ov              Modal détail/édition ticket
└── <script>
    ├── DB           Base de données en mémoire
    ├── Auth         Authentification + gestion sessions
    ├── Router       renderPage() — navigation SPA
    ├── Pages        pageDashboard, pageTickets, pageUsers…
    └── Utils        Formatage, SLA, toasts, journal
```

### Stack technique

| Composant | Technologie |
|-----------|-------------|
| Structure | HTML5 |
| Style | CSS3 (variables, grid, flexbox) |
| Logique | JavaScript ES6+ (Vanilla, sans framework) |
| Graphiques | Chart.js 4.4.0 (CDN) |
| Polices | DM Sans + JetBrains Mono (Google Fonts) |
| Stockage | Mémoire vive (objet `DB` en session) |

---

## 👤 Rôles et permissions

| Fonctionnalité | S-Admin | Admin | User | Reader |
|---|:---:|:---:|:---:|:---:|
| Tableau de bord | ✅ | ✅ | ✅ (limité) | ✅ (limité) |
| Voir tous les tickets | ✅ | ✅ | ❌ | ✅ |
| Voir ses tickets | ✅ | ✅ | ✅ | — |
| Créer un ticket | ✅ | ✅ | ✅ | ❌ |
| Modifier statut/priorité | ✅ | ✅ | ❌ | ❌ |
| Réassigner un service | ✅ | ✅ | ❌ | ❌ |
| Ajouter un commentaire | ✅ | ✅ | ✅ | ✅ |
| Gérer les utilisateurs | ✅ | ✅ (partiel) | ❌ | ❌ |
| Créer un Admin/S-Admin | ✅ | ❌ | ❌ | ❌ |
| Supprimer un utilisateur | ✅ | ❌ | ❌ | ❌ |
| Rapports & export | ✅ | ✅ | Configurable | Configurable |
| Journal d'activité | ✅ | ✅ | ❌ | ❌ |
| Configuration système | ✅ | ✅ | ❌ | ❌ |

---

## ⏱️ Niveaux SLA

| Priorité | Prise en charge | Résolution | Couleur |
|----------|----------------|------------|---------|
| **P1 — Critique** | 30 minutes | 4 heures | 🔴 Rouge |
| **P2 — Majeur** | 2 heures | 8 heures | 🟠 Orange |
| **P3 — Mineur** | 24 heures | 72 heures | 🟢 Vert |

---

## 📁 Structure des données (objet DB)

```javascript
const DB = {
  users: [
    {
      id, login, pw, role,        // Identifiants
      nm, prenom, nom, mat,       // Identité
      email, svc,                 // Contact & service
      active,                     // Statut du compte
      accesRapport, exportRapport,// Permissions spécifiques
      createdAt, createdBy        // Audit
    }
  ],
  tickets: [
    {
      id, type,                   // 'interne' | 'client'
      catInterne, catClient,      // Catégorie selon type
      clientNom, produit,         // Infos client (si réclamation)
      desc, prio, statut,         // Contenu et état
      svcAssigne,                 // Service responsable
      emetteurId, emetteurNm,     // Auteur
      dt, dtDetect, slaMax, wait, // Dates et SLA
      timeline: [],               // Historique des événements
      comments: []                // Commentaires
    }
  ],
  logs: [],           // Journal d'activité (max 200 entrées)
  nextUserId: 7,      // Compteur auto-incrémental
  nextTicketId: 7,
};
```

---

## 🔧 Personnalisation

### Modifier les services assignataires
Dans le script JS, modifier le tableau `SVCS` :
```javascript
const SVCS = ['DSI', 'Boldcode', 'Direction Opérations', 'Relation Clientèle', 'Autres'];
```

### Modifier les SLA
Dans `PRIOS`, ajuster `slaMax` (en minutes) :
```javascript
const PRIOS = {
  P1: { slaMax: 30   },   // 30 minutes
  P2: { slaMax: 120  },   // 2 heures
  P3: { slaMax: 1440 },   // 24 heures
};
```

### Modifier les catégories d'incidents
```javascript
const CAT_INT = ['Accès comptes', 'Plateforme — Bugs / Lenteurs', ...];
const CAT_CLI = ['Retard de réception', 'Non réception', ...];
```

---

## ⚠️ Limitations connues

| Limitation | Impact | Solution recommandée |
|------------|--------|---------------------|
| Données en mémoire uniquement | Perte des données à la fermeture | Intégrer un backend (API REST + base SQL/NoSQL) |
| Accès monoutilisateur | Pas de collaboration simultanée | Héberger sur un serveur web |
| Alertes email/WhatsApp simulées | Pas d'envoi réel | Connecter à SMTP / API WhatsApp Business |
| Export CSV/PDF simulé | Pas de fichier réel généré | Implémenter côté serveur (Node.js, Python…) |
| Pas de chiffrement des mots de passe | Sécurité réduite | Hacher les mots de passe côté serveur (bcrypt) |

---

## 🗺️ Évolutions prévues (Roadmap)

- [ ] **Backend REST** — Node.js/Express + PostgreSQL ou MongoDB
- [ ] **Authentification sécurisée** — JWT + refresh tokens
- [ ] **Notifications réelles** — SMTP (nodemailer) + API WhatsApp Business
- [ ] **Export PDF** — Puppeteer ou WeasyPrint côté serveur
- [ ] **Hébergement intranet** — Docker + reverse proxy Nginx
- [ ] **Application mobile (PWA)** — Accès nomade depuis smartphone
- [ ] **Tableau de bord KPI avancé** — Temps moyen de résolution, taux SLA

---

## 📄 Licence & Contact

```
Éditeur   : VEDEM — Direction des Services Informatiques
Version   : V1.02
Date      : Mars 2026
Statut    : Prototype fonctionnel — Prêt pour intégration backend
Document  : À usage interne — Confidentiel
```

---

*TICKETING VEDEM V1.02 — © 2026 VEDEM. Tous droits réservés.*
