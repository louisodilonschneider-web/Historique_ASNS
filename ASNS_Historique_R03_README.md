# 🏊 Suivi Aquatique EPS

Application web de suivi des diplômes aquatiques (**ASNS** et **Aisance aquatique**) pour l'équipe EPS.  
Développée en HTML / JavaScript vanilla, connectée à **Firebase Firestore** pour le partage des données entre tous les enseignants.

---

## Sommaire

1. [Démarrage rapide](#démarrage-rapide)
2. [Firebase — projet utilisé](#firebase--projet-utilisé)
3. [Workflow annuel](#workflow-annuel)
4. [Format CSV attendu](#format-csv-attendu)
5. [Fonctionnalités détaillées](#fonctionnalités-détaillées)
6. [Corrélation inter-années](#corrélation-inter-années)
7. [Enseignants configurés](#enseignants-configurés)
8. [Export PDF](#export-pdf)

---

## Démarrage rapide

1. Ouvrir `index.html` dans un navigateur (Chrome, Firefox, Edge) **avec connexion internet**
2. Patienter quelques secondes pendant la connexion à Firebase (écran de chargement 🏊)
3. Créer une année scolaire : taper `2024-2025` → **+ Créer**
4. Ajouter les classes : taper `6e A` → **+ Ajouter une classe**
5. Importer un CSV par classe via le bouton **⬆ CSV** sur la ligne de la classe
6. Affecter les enseignants → onglet **Affectation enseignants**
7. Renseigner les diplômes → bouton **✏️ Saisie rapide** ou fiche individuelle ✏️

> Les données sont partagées en temps réel entre tous les enseignants qui ouvrent le fichier.  
> Une copie de secours est conservée localement dans le navigateur en cas de perte de connexion.

---

## Firebase — projet utilisé

L'application utilise **Firebase Realtime Database** (pas Firestore).

| Paramètre           | Valeur                                                                          |
|---------------------|---------------------------------------------------------------------------------|
| **Projet**          | `inventaire-materiel-a7b8c`                                                     |
| **Base de données** | Realtime Database — `europe-west1` (Belgique)                                   |
| **Nœud de données** | `/eps_aquatique/{ annees, eleves }`                                              |
| **URL**             | `https://inventaire-materiel-a7b8c-default-rtdb.europe-west1.firebasedatabase.app` |

### Configuration intégrée dans `index.html`

```javascript
const firebaseConfig = {
  apiKey:            "AIzaSyBKlBfOOQN2mlH2GYyYhWMvo6Ozw4m8TzI",
  authDomain:        "inventaire-materiel-a7b8c.firebaseapp.com",
  databaseURL:       "https://inventaire-materiel-a7b8c-default-rtdb.europe-west1.firebasedatabase.app",
  projectId:         "inventaire-materiel-a7b8c",
  storageBucket:     "inventaire-materiel-a7b8c.firebasestorage.app",
  messagingSenderId: "708986692713",
  appId:             "1:708986692713:web:68b8dd9ab9f86178ac6ef8",
  measurementId:     "G-GEB31HYEXC"
};
```

### Règles Realtime Database

Dans la console Firebase → **Realtime Database → Règles**, vos règles actuelles sont :

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

Cela autorise l'accès à tous — suffisant pour une utilisation en réseau scolaire fermé.  
Pour restreindre aux seuls enseignants connectés, remplacer `true` par `"auth != null"` et activer Firebase Authentication.

> ⚠️ Les données du projet (`eps` et `inventaire`) coexistent dans la même Realtime Database. Les données EPS sont stockées sous le nœud `/eps_aquatique` pour ne pas interférer avec l'existant.

---

## Workflow annuel

### En début d'année

```
1. Créer l'année (ex : 2025-2026)
2. Ajouter les classes manuellement : 6e A, 6e B, 6e C…
3. Pour chaque classe : cliquer ⬆ CSV sur sa ligne → sélectionner le fichier
4. Renseigner le jour piscine de chaque classe (Mardi ou Jeudi)
5. Aller dans l'onglet "Affectation enseignants"
   → Cocher les élèves d'un groupe de niveau (plusieurs classes possibles)
   → Choisir l'enseignant → Affecter
6. Renseigner les diplômes déjà acquis (champ "Diplôme à l'entrée")
```

### En cours d'année

```
- Modifier classe / enseignant d'un élève via ✏️
- Cocher "Élève inactif" pour un départ (historique conservé)
- Saisir les notes trimestrielles T1 / T2 / T3
- Valider les diplômes en mode saisie rapide (bouton vert "✏️ Saisie rapide")
  → Cliquer ASNS / Aisance / Rien sur chaque ligne
  → Si Rien : choisir la raison directement sur la ligne
```

### En fin d'année

```
- Consulter les statistiques (onglet Statistiques)
- Exporter le PDF de synthèse (bouton 📄 PDF)
- L'année suivante : créer 2026-2027, réimporter les élèves devenus 5e
  → Leur historique est automatiquement retrouvé
```

---

## Format CSV attendu

L'application lit le **format d'export de votre logiciel de gestion scolaire**.

### Colonnes reconnues

| Colonne CSV                  | Utilisation                                              |
|------------------------------|----------------------------------------------------------|
| `Élèves`                     | ✅ **Obligatoire** — format `NOM Prénom` (nom en MAJUSCULES) |
| `Né(e) le`                   | ✅ Recommandé — date de naissance pour éviter les homonymes |
| `Sexe`                       | Optionnel — `M`, `F`, `Masculin`, `Féminin` acceptés    |
| `Classe de rattachement`     | Optionnel si la classe est forcée au moment de l'import  |
| Toutes les autres colonnes   | Ignorées silencieusement                                 |

> La colonne **Enseignant** n'est pas dans le CSV — l'affectation se fait manuellement car les classes sont réparties par groupes de niveau.

### Séparateur

`;` ou `,` — détecté automatiquement.

### Exemple

```csv
Élèves;Encouragement/Valorisation;Né(e) le;Sexe;Adresse E-mail;Entrée;Sortie;;Classe de rattachement
DUPONT Lucas;;12/03/2013;M;lucas@mail.fr;01/09/2024;;;6e A
MARTIN Emma;;04/07/2013;F;;;01/09/2024;;;6e B
LEROY Nathan;;22/11/2012;M;;;01/09/2024;;;6e A
```

### Import par classe

Dans le tableau des classes, le bouton **⬆ CSV** de chaque ligne importe le fichier **exclusivement pour cette classe**, quelle que soit la colonne `Classe de rattachement` dans le fichier.  
Cela permet d'utiliser des exports bruts classe par classe.

---

## Fonctionnalités détaillées

### Tableau des classes

Chaque classe dispose de :
- Compteur d'élèves importés
- Statut du CSV importé (nom du fichier + date)
- Sélecteur **Jour piscine** : Mardi ou Jeudi
- Bouton **⬆ CSV** propre à la classe
- Bouton 🗑️ suppression (les élèves sont conservés)

### Onglet Suivi élèves

- **Couleurs de fond** : 🟢 vert = ASNS · 🟡 jaune = Aisance · ⬜ neutre = Non obtenu
- **★** sur le nom = diplôme aquatique déjà détenu à l'entrée de l'année
- **Filtres** : classe, enseignant, statut, recherche texte
- **Mode saisie rapide** (bouton vert) : ASNS / Aisance / Rien en un clic par ligne, avec les raisons de non-obtention directement accessibles sur la même ligne
- **Affectation rapide** : cocher plusieurs élèves → barre violette → choisir l'enseignant

### Raisons de non-obtention

| Raison              | Description                              |
|---------------------|------------------------------------------|
| Absence             | Élève absent lors des séances            |
| Dispense ponctuelle | Dispense médicale temporaire             |
| Dispense annuelle   | Dispense médicale sur toute l'année      |
| Matériel            | Problème d'équipement                    |
| Autre               | Champ libre pour préciser                |

### Onglet Affectation enseignants

- Vue par classe, sélection individuelle ou **"Tout sélectionner"** par classe
- Sélection inter-classes pour les groupes de niveau
- Modification possible en cours d'année

### Onglet Statistiques

**Global :**
- Nombre et % ASNS / Aisance / Non obtenu
- Élèves sans diplôme à l'entrée → diplôme validé en fin d'année

**Par classe :**
- Tableau avec totaux et % de diplômés (vert ≥ 50%, rouge < 50%)

### Recherche globale (barre en haut)

- Cherche dans **toutes les années et toutes les promotions**
- Résultats dès 2 caractères
- Clic → fiche historique complète de l'élève (toutes ses années)

---

## Corrélation inter-années

Chaque élève a un **identifiant unique** : `nom + prénom + date de naissance`.

Quand un élève de 6e est réimporté en 5e l'année suivante, l'application le reconnaît et conserve tout son historique. Seules les données de la nouvelle année sont créées.

### Exemple de suivi sur 4 ans

| Année     | Classe | Enseignant   | Diplôme           |
|-----------|--------|--------------|-------------------|
| 2024-2025 | 6e A   | Mme Allard   | Aisance aquatique |
| 2025-2026 | 5e B   | M. Bros      | ASNS              |
| 2026-2027 | 4e A   | M. Jaccaz    | —                 |
| 2027-2028 | 3e C   | Mme Wattron  | —                 |

> **Conseil** : inclure la colonne `Né(e) le` dans le CSV pour différencier les homonymes.

---

## Enseignants configurés

| Enseignant     |
|----------------|
| Mme Allard     |
| Mme Wattron    |
| M. Bros        |
| M. Jaccaz      |
| M. Schneider   |

Pour modifier, rechercher `Mme Allard` dans `index.html` (4 occurrences) et remplacer.

---

## Export PDF

Le bouton **📄 PDF** génère `suivi_aquatique_ANNEE.pdf` contenant :

**Tableau des élèves** (respecte les filtres actifs) :
- Nom, prénom, classe, enseignant EPS
- Notes T1 / T2 / T3
- Diplôme obtenu ou non
- Date d'obtention
- Raison de non-obtention

**Page de synthèse statistique** :
- Totaux et pourcentages globaux
- Compteur "sans diplôme à l'entrée → validé en fin d'année"

---

## Compatibilité

| Navigateur | Support |
|------------|---------|
| Chrome 90+ | ✅      |
| Firefox 88+ | ✅     |
| Edge 90+   | ✅      |
| Safari 14+ | ✅      |
| Internet Explorer | ❌ |

**Connexion internet requise** pour synchroniser avec Firebase.  
En cas de déconnexion, les données restent accessibles et modifiables localement — elles se resynchronisent automatiquement au retour de la connexion.

---

*Application EPS — Suivi aquatique ASNS / Aisance aquatique*
