# 🏊 Suivi Aquatique EPS

Application web de suivi des diplômes aquatiques (ASNS et Aisance aquatique) pour l'équipe EPS.  
Développée en HTML / JavaScript vanilla + Firebase Firestore.

---

## Fonctionnalités

- **Gestion par année scolaire** — chaque promotion est isolée mais les élèves sont corrélés d'une année sur l'autre via un identifiant unique (nom + prénom)
- **Import CSV** — chargement des effectifs par classe en début d'année
- **Suivi individuel** — diplôme obtenu (ASNS / Aisance), date d'obtention, notes trimestrielles, raison de non-obtention, diplôme prealable
- **Filtres** — par classe, enseignant, statut diplôme, recherche libre
- **Statistiques** — taux de réussite, progression des élèves sans diplôme aquatique à l'entrée
- **Export PDF** — synthèse complète ou filtrée par classe / enseignant

---

## Prérequis

- Un projet **Firebase** (gratuit sur le plan Spark)
- Un navigateur moderne (Chrome, Firefox, Edge)
- Aucune installation serveur requise — l'application tourne entièrement côté client

---

## Installation

### 1. Créer le projet Firebase

1. Aller sur [https://console.firebase.google.com](https://console.firebase.google.com)
2. **Add project** → donner un nom (ex : `suivi-aquatique-eps`)
3. Désactiver Google Analytics (optionnel)
4. Dans le menu gauche : **Build → Firestore Database**
5. Cliquer **Create database** → choisir **Start in test mode** (à sécuriser plus tard)
6. Choisir la région **eur3 (europe-west)**

### 2. Récupérer la configuration

1. Dans Firebase Console : ⚙️ **Project settings** → onglet **General**
2. Descendre jusqu'à **Your apps** → cliquer l'icône **</>** (web)
3. Nommer l'app (ex : `suivi-eps`) → **Register app**
4. Copier l'objet `firebaseConfig`

### 3. Configurer `index.html`

Ouvrir `index.html` et remplacer le bloc en haut du script :

```javascript
const firebaseConfig = {
  apiKey:            "VOTRE_API_KEY",
  authDomain:        "VOTRE_PROJECT.firebaseapp.com",
  projectId:         "VOTRE_PROJECT_ID",
  storageBucket:     "VOTRE_PROJECT.appspot.com",
  messagingSenderId: "VOTRE_SENDER_ID",
  appId:             "VOTRE_APP_ID"
};
```

### 4. Déployer (optionnel)

L'application peut être hébergée sur **Firebase Hosting** (gratuit) :

```bash
npm install -g firebase-tools
firebase login
firebase init hosting        # sélectionner le projet, dossier public = .
firebase deploy
```

Ou simplement ouvrir `index.html` directement dans un navigateur.

---

## Format CSV attendu

Le fichier CSV doit contenir **une ligne d'en-tête** avec les colonnes suivantes (séparateur `;` ou `,`) :

| Colonne           | Obligatoire | Description                                      |
|-------------------|-------------|--------------------------------------------------|
| `nom`             | ✅          | Nom de famille                                   |
| `prenom`          | ✅          | Prénom                                           |
| `classe`          | ✗           | Ex : `6eA`, `5eB`                               |
| `enseignant`      | ✗           | Nom de l'enseignant EPS (doit correspondre à la liste) |
| `date_naissance`  | ✗           | Format `JJ/MM/AAAA` — améliore la corrélation si homonymes |

**Exemple :**

```csv
nom;prenom;classe;enseignant;date_naissance
Dupont;Lucas;6eA;Mme Allard;12/03/2013
Martin;Emma;6eB;M. Bros;04/07/2013
Leroy;Nathan;6eA;Mme Wattron;22/11/2012
```

> ℹ️ Les colonnes manquantes sont acceptées. L'import est **non-destructif** : si l'élève existe déjà (même identifiant), ses données de diplôme et de notes sont conservées ; seules les infos d'identité et d'affectation sont mises à jour.

---

## Workflow annuel

### En début d'année scolaire

1. **Créer l'année** dans le sélecteur (`2025-2026`)
2. **Importer le CSV** de la nouvelle promotion de 6ème
3. Pour les élèves **déjà connus** (anciens 5ème, etc.) : l'identifiant commun assure la continuité
4. Renseigner le **diplôme prealable** des élèves qui en ont déjà un (champ dans la fiche individuelle)

### En cours d'année

- Modifier la **classe** ou l'**enseignant** d'un élève via le bouton ✏️
- Cocher **Élève inactif** pour les départs en cours d'année (sans supprimer l'historique)
- Renseigner les **notes trimestrielles** au fur et à mesure
- Valider le **diplôme** dès qu'il est obtenu, avec la date

### En fin d'année

- Consulter les **statistiques** (taux ASNS / Aisance / rien, progression des non-diplômés)
- **Exporter le PDF** global ou filtré par classe / enseignant

---

## Corrélation inter-années

L'identifiant unique d'un élève est calculé à partir de **nom + prénom (+ date de naissance si fournie)**.  
Quand un élève est importé l'année suivante dans une nouvelle classe, l'application retrouve automatiquement son historique si l'identifiant correspond.

> Conseil : inclure `date_naissance` dans le CSV pour éviter les collisions en cas d'homonymes.

---

## Enseignants EPS configurés

| Enseignant     |
|----------------|
| Mme Allard     |
| Mme Wattron    |
| M. Bros        |
| M. Jaccaz      |
| M. Schneider   |

Pour modifier cette liste, éditer le tableau `ENSEIGNANTS` en haut du script dans `index.html`.

---

## Raisons de non-obtention disponibles

- Absence
- Dispense ponctuelle
- Dispense annuelle
- Matériel
- Autre *(champ libre pour préciser)*

---

## Sécurité Firestore

En mode **test**, toute personne avec l'URL peut lire et écrire.  
Pour un usage en établissement, configurer des règles Firestore dans la console Firebase :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Puis activer **Firebase Authentication** (Google Sign-In ou email/mot de passe) pour réserver l'accès à l'équipe EPS.

---

## Structure Firestore

```
annees/
  2024-2025/
    eleves/
      dupont_lucas_12032013/
        nom, prenom, classe, enseignant
        noteT1, noteT2, noteT3
        diplome, dateDiplome
        diplomePrealable
        raison, raisonAutre
        actif, notes
  2025-2026/
    eleves/ ...
```

---

## Technologies utilisées

| Brique           | Version  |
|------------------|----------|
| Firebase JS SDK  | 10.12.0  |
| jsPDF            | 2.5.1    |
| HTML / CSS / JS  | Vanilla  |

Aucun framework, aucun build tool. Un seul fichier `index.html`.
