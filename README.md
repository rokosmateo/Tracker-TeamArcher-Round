# Tracker-TeamArcher-Round 🎯

Application web de **suivi de match de tir à l'arc par équipe** (rencontres D1 / Team, par division).
Tout tient dans un seul fichier (`index.html`) : pas de serveur à installer, pas d'étape de
« build ». Les scores se synchronisent en temps réel entre tous les appareils grâce à **Firebase
Firestore** (un téléphone met un score à jour → les autres écrans se rafraîchissent en moins d'une
seconde).

> Ce dépôt est un **modèle public** : n'importe quel club peut le copier, créer sa propre base Firebase
> gratuite et héberger sa propre version. Ce README explique comment faire, pas à pas, **sans
> connaissances techniques particulières**.

---

## 📋 Sommaire

1. [Comment ça marche (en bref)](#-comment-ça-marche-en-bref)
2. [Étape 1 — Récupérer le projet](#-étape-1--récupérer-le-projet)
3. [Étape 2 — Créer votre base Firebase (gratuit)](#-étape-2--créer-votre-base-firebase-gratuit)
4. [Étape 3 — Personnaliser le fichier (nom, mot de passe, Firebase)](#-étape-3--personnaliser-le-fichier-nom-mot-de-passe-firebase)
5. [Étape 4 — Héberger le site (Netlify)](#-étape-4--héberger-le-site-netlify)
6. [Tester en local (optionnel)](#-tester-en-local-optionnel)
7. [Plusieurs équipes sur la même base](#-plusieurs-équipes-sur-la-même-base)
8. [Questions fréquentes](#-questions-fréquentes)

---

## 🔧 Comment ça marche (en bref)

- **Une seule page HTML** contient toute l'application (interface + logique).
- L'état complet du tournoi (matchs, scores, équipes…) est enregistré :
  - dans le **navigateur** (`localStorage`) → affichage immédiat, fonctionne même hors ligne ;
  - dans **Firebase Firestore** → partage en temps réel entre tous les appareils.
- Dans Firestore, tout est stocké dans **un seul document** : la collection `teams`, document
  `equipe-principale`, avec un champ `payload` (le tournoi complet au format JSON) et un champ `updatedAt`.

Pour avoir votre propre version, vous devez :
1. créer **votre** projet Firebase,
2. coller **votre** configuration dans `index.html`,
3. activer la **connexion coach** et verrouiller les **règles** (section
   [Sécurité](#-sécurité--qui-peut-modifier-les-scores)) pour que vous seul puissiez saisir les scores.

---

## 📥 Étape 1 — Récupérer le projet

Vous avez deux options.

### Option A — « Fork » (recommandé, garde le lien avec l'original)
1. Connectez-vous sur [github.com](https://github.com).
2. Allez sur la page de ce dépôt et cliquez sur le bouton **Fork** (en haut à droite).
3. GitHub crée une copie **sur votre propre compte**. C'est elle que vous modifierez.

### Option B — Télécharger le fichier
1. Ouvrez `index.html` dans le dépôt.
2. Cliquez sur **Download raw file** (icône de téléchargement) pour récupérer le fichier seul.

---

## 🔥 Étape 2 — Créer votre base Firebase (gratuit)

Firebase est un service Google. L'offre gratuite (« plan Spark ») suffit très largement pour un club.

1. Allez sur **[console.firebase.google.com](https://console.firebase.google.com)** et connectez-vous
   avec un compte Google.
2. Cliquez sur **« Ajouter un projet »** (*Add project*).
   - Donnez-lui un nom, par ex. `suivi-mon-club`.
   - Vous pouvez **désactiver Google Analytics** (inutile ici) puis valider.
3. Une fois le projet créé, dans le menu de gauche ouvrez **Build → Firestore Database**.
   - Cliquez sur **« Créer une base de données »** (*Create database*).
   - Choisissez un emplacement (par ex. `eur3` ou `europe-west` si vous êtes en Europe).
   - Démarrez **en mode test** (*Start in test mode*) pour commencer simplement *(voir la note
     sécurité plus bas)*, puis **Activer**.
4. Toujours dans le menu de gauche, cliquez sur la **roue dentée ⚙️ → Paramètres du projet**
   (*Project settings*).
5. Faites défiler jusqu'à **« Vos applications »** (*Your apps*) et cliquez sur l'icône **Web**
   `</>`.
   - Donnez un surnom à l'application (par ex. `web`), **ne cochez pas** « Firebase Hosting ».
   - Cliquez sur **« Enregistrer l'application »**.
6. Firebase vous affiche un bloc `firebaseConfig` qui ressemble à ceci :

   ```js
   const firebaseConfig = {
     apiKey: "AIza...........................",
     authDomain: "suivi-mon-club.firebaseapp.com",
     projectId: "suivi-mon-club",
     storageBucket: "suivi-mon-club.firebasestorage.app",
     messagingSenderId: "000000000000",
     appId: "1:000000000000:web:xxxxxxxxxxxxxxxx"
   };
   ```

   **Gardez cet écran ouvert** : vous allez copier ces valeurs à l'étape suivante.

> 💡 Ces clés Firebase ne sont **pas secrètes** : elles sont prévues pour être mises dans le code
> d'un site web public. La protection des données se fait via les **règles de sécurité Firestore**
> (voir la section sécurité plus bas), pas en cachant la clé.

---

## 🔌 Étape 3 — Personnaliser le fichier (nom, Firebase)

Toute la personnalisation se fait dans le **même fichier**, `index.html`. Ouvrez-le (dans GitHub :
ouvrez le fichier puis cliquez sur l'icône **crayon ✏️** pour l'éditer).

### 3.1 — Nom du club et titre

Tout en haut du script, cherchez le bloc **`APP_CONFIG`** (vers la **ligne 1668**) :

```js
const APP_CONFIG = {
  // Nom affiché en haut de la page (sous-titre).
  clubName: 'Nom de votre club · Saison 2026',
  // Titre de l'onglet du navigateur.
  pageTitle: 'Suivi de match — Tir à l'arc'
};
```

- **`clubName`** → le nom de **votre** club, affiché en haut de l'application.
  Par ex. `'Compagnie des Archers de Lyon · Saison 2026'`.
- **`pageTitle`** → ce qui s'affiche dans l'onglet du navigateur.

> 🔐 **Et le mot de passe ?** Il n'est **plus dans le code**. L'accès « coach » (saisie des scores)
> se fait désormais par **Firebase Authentication** : on se connecte avec un email + mot de passe
> réels, vérifiés côté serveur. Voir la [section Sécurité](#-sécurité--qui-peut-modifier-les-scores)
> pour créer vos comptes coach. C'est une **vraie** protection, contrairement à un mot de passe écrit
> dans la page.

> ⚠️ Gardez bien les **apostrophes** `'…'` autour de chaque valeur. Ne modifiez que le texte entre
> apostrophes.

### 3.2 — Brancher votre Firebase

Toujours dans `index.html`, cherchez le bloc `FIREBASE_CONFIG` (vers la **ligne 1986**). Il
ressemble à ceci :

   ```js
   const FIREBASE_CONFIG = {
     apiKey: "VOTRE_API_KEY",
     authDomain: "VOTRE_PROJET.firebaseapp.com",
     projectId: "VOTRE_PROJET",
     storageBucket: "VOTRE_PROJET.firebasestorage.app",
     messagingSenderId: "VOTRE_SENDER_ID",
     appId: "VOTRE_APP_ID"
   };
   ```

**Remplacez ces six valeurs** par celles de **votre** projet (copiées à l'étape 2).

Juste en dessous (vers la **ligne 1994**), il y a l'identifiant de l'équipe :

```js
const TEAM_ID = 'equipe-principale';
```

Vous pouvez le laisser tel quel, ou le renommer (par ex. `'mon-club'`). C'est le nom du document
Firestore qui contiendra vos données.

### 3.3 — Enregistrer

**Enregistrez** vos modifications (sur GitHub : *Commit changes* ; sur votre ordinateur : sauvegardez
simplement le fichier).

C'est tout côté configuration : l'application applique automatiquement votre nom de club et votre
titre au chargement, et crée le document dans Firestore au premier lancement.

---

## 🌐 Étape 4 — Héberger le site (Netlify)

[Netlify](https://www.netlify.com) héberge votre page **gratuitement et sans limite de durée**, avec
une adresse `https://…` et un certificat sécurisé automatique. Deux méthodes au choix : la plus
simple (glisser-déposer) ou la connectée à GitHub (mise à jour automatique à chaque modification).

### Méthode A — Glisser-déposer (la plus rapide, sans compte GitHub)

1. Téléchargez le fichier `index.html` (déjà modifié avec **votre** config Firebase de l'étape 3) sur
   votre ordinateur, **dans un dossier seul** (par ex. un dossier `mon-club`).
   > ℹ️ Netlify publie un **dossier**. Mettez-y au minimum `index.html` ; le fichier d'accueil doit
   > obligatoirement s'appeler `index.html`.
2. Allez sur **[app.netlify.com/drop](https://app.netlify.com/drop)**.
3. **Glissez-déposez votre dossier** dans la zone indiquée sur la page.
4. En quelques secondes, Netlify met le site en ligne et affiche une adresse du type :

   ```
   https://nom-aleatoire-123456.netlify.app
   ```

5. Ouvrez ce lien : votre application est en ligne ! 🎉 (Créez un compte gratuit pour conserver le
   site et pouvoir le renommer.)

### Méthode B — Connecté à GitHub (mise à jour automatique)

Recommandé si vous avez fait un **fork** à l'étape 1 : chaque modification poussée sur GitHub
re-publie le site automatiquement.

1. Créez un compte gratuit sur **[netlify.com](https://www.netlify.com)** (vous pouvez vous
   connecter avec votre compte GitHub).
2. Cliquez sur **« Add new site »** → **« Import an existing project »**.
3. Choisissez **GitHub** et autorisez Netlify à accéder à vos dépôts.
4. Sélectionnez votre dépôt **Tracker-TeamArcher-Round**.
5. Réglages de build — laissez **tout vide** (c'est un simple fichier HTML, il n'y a rien à
   compiler) :
   - **Build command** : *(vide)*
   - **Publish directory** : `.` (le point, ou laissez vide)
6. Cliquez sur **« Deploy site »**. Après ~1 minute, votre site est en ligne à une adresse
   `https://….netlify.app`.

### Personnaliser l'adresse (les deux méthodes)

Dans Netlify : **Site configuration → Change site name** pour obtenir une adresse plus lisible, par
exemple :

```
https://suivi-mon-club.netlify.app
```

Partagez ce lien avec votre équipe ! 🎯

> **Autres hébergeurs possibles** (même principe — déposer le fichier `index.html`) :
> [Cloudflare Pages](https://pages.cloudflare.com), [Vercel](https://vercel.com), ou le **Firebase
> Hosting** de votre propre projet.

---

## 💻 Tester en local (optionnel)

Vous pouvez ouvrir `index.html` directement dans votre navigateur, mais Firebase fonctionne mieux
via un petit serveur local. Si Python est installé :

```bash
# Dans le dossier du projet
python3 -m http.server 8000
```

Puis ouvrez **http://localhost:8000** dans votre navigateur.

---

## 👥 Plusieurs équipes sur la même base

Tout le tournoi tient dans **un seul document** Firestore (`teams/<TEAM_ID>`). Si vous voulez faire
tourner **plusieurs instances séparées** depuis le même projet Firebase (par ex. deux clubs, ou
équipe homme / équipe femme), il suffit de **dupliquer le fichier** et de donner un `TEAM_ID`
différent à chacun :

```js
const TEAM_ID = 'club-A';   // dans la première copie
const TEAM_ID = 'club-B';   // dans la seconde copie
```

Chaque `TEAM_ID` crée un document indépendant : les données ne se mélangent pas.

---

## 🔒 Sécurité — qui peut modifier les scores ?

L'application sépare deux rôles :

- **Spectateur** (tout le monde) : voit les scores en **lecture seule**, sans rien installer ni se
  connecter. La synchronisation en temps réel fonctionne pour tous.
- **Coach** (vous et votre équipe) : se **connecte avec un compte** (email + mot de passe) pour
  **saisir** les scores.

Cette séparation est garantie **côté serveur** par Firebase : même si quelqu'un trouve l'adresse du
site et lit le code, il **ne peut pas écrire** dans la base sans être connecté avec un vrai compte
coach. C'est une **vraie** protection (contrairement à un mot de passe écrit dans la page).

Il y a **trois choses** à régler dans la console Firebase, une seule fois :

### A. Activer la connexion par email/mot de passe

1. Console Firebase → menu de gauche **Build → Authentication**.
2. Cliquez sur **« Get started »** (Commencer) si c'est la première fois.
3. Onglet **Sign-in method** (Mode de connexion) → **Email/Password** → **Activez** le premier
   interrupteur → **Enregistrer**. *(Laissez « Email link » désactivé.)*

### B. Créer un (ou plusieurs) compte coach

1. Toujours dans **Authentication**, onglet **Users** (Utilisateurs).
2. **« Add user »** (Ajouter un utilisateur) → saisissez un **email** et un **mot de passe** pour le
   coach → **Add user**.
3. Répétez pour chaque personne autorisée à saisir les scores.

> Ce sont ces identifiants que vous taperez dans l'app en cliquant sur **🔒 Spectateur → Mode coach**.
> Le mot de passe est stocké de façon sécurisée par Google, **jamais dans le code**.

### C. Verrouiller les règles Firestore

Console Firebase → **Firestore Database → Règles** (*Rules*) → collez ceci → **Publier** :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /teams/{teamId} {
      allow read: if true;                  // Lecture publique (spectateurs)
      allow write: if request.auth != null; // Écriture réservée aux coachs connectés
    }
  }
}
```

> ✅ Avec ces règles : **lecture pour tous**, **écriture seulement pour un compte connecté**. Un
> visiteur (ou un robot) qui tombe sur l'URL ne pourra **pas** modifier vos scores.

> 💡 Pour restreindre encore plus (n'autoriser QUE des comptes précis), remplacez la ligne d'écriture
> par : `allow write: if request.auth != null && request.auth.token.email in ['coach1@exemple.fr', 'coach2@exemple.fr'];`

---

## ❓ Questions fréquentes

**Est-ce que ça coûte quelque chose ?**
Non. Netlify et le plan gratuit de Firebase (Spark) suffisent pour un club. Vous ne payez que si vous
dépassez des quotas très élevés (des millions de lectures), ce qui n'arrivera pas.

**Un nouveau club démarre-t-il avec VOS données (archers, rencontres) ?**
Non. La version partagée démarre sur une **page blanche** : aucune rencontre pré-remplie, et des noms
d'archers génériques (`Archère 1`…`Archère 4`) que vous remplacez dans l'app. Vous créez vos
compétitions et vos matchs vous-même via le bouton **« + Ajouter »** (en mode coach). Pour
pré-remplir des rencontres dans le code, modifiez le bloc `DEFAULT_MATCHES` dans `index.html`
(le format est expliqué juste au-dessus de ce bloc).

**Les clés Firebase dans le code, c'est dangereux ?**
Non, elles sont faites pour être publiques. La sécurité repose sur les **règles Firestore** + la
**connexion coach** (section Sécurité), pas sur le secret de la clé.

**J'ai oublié le mot de passe d'un coach, comment faire ?**
Dans la console Firebase → **Authentication → Users**, cliquez sur les trois points à droite de
l'utilisateur → **Reset password** (ou supprimez le compte et recréez-le). Rien à toucher dans le
code.

**Un visiteur peut-il modifier mes scores en lisant le code source ?**
Non. Le code ne contient aucun mot de passe, et les **règles Firestore** interdisent toute écriture
sans être connecté avec un compte coach. Un visiteur reste en lecture seule.

**Mes données et celles du club original sont-elles mélangées ?**
Non, à condition d'utiliser **votre propre projet Firebase**. Chaque projet a sa propre base, isolée.

**Comment remettre les compteurs à zéro / supprimer un tournoi ?**
Depuis l'application elle-même (interface de gestion des matchs), ou en supprimant le document
`teams/<TEAM_ID>` dans la console Firestore.

**Comment saisir / modifier une flèche ?**
Cliquez sur une case **vide** pour saisir une flèche avec la cible tactile. Sur mobile, une flèche
**déjà saisie** se modifie par un **appui long** (pour éviter de l'écraser par un tap accidentel).
Sur la cible, **chaque archer a sa propre couleur** d'impact.

**Ça marche sur téléphone ?**
Oui, l'interface est pensée pour mobile et tablette autant que pour ordinateur.

---

## 📄 Licence

Ce projet est distribué sous licence **GNU AGPL-3.0** (voir le fichier [LICENSE](LICENSE)).

Concrètement, vous êtes **libre** d'utiliser, copier, modifier et héberger ce projet — y compris pour
votre club — **gratuitement**. En contrepartie, la licence impose une règle simple :

- 🔓 **Toute version modifiée et mise en ligne doit rester open source** : si vous adaptez l'appli et
  la rendez accessible à des utilisateurs (même via un site web), vous devez **mettre votre code
  source à disposition** sous cette même licence AGPL-3.0.
- 🚫 **Personne ne peut en faire un produit fermé et payant** : il est interdit de reprendre ce code,
  de le garder secret et de le revendre. Tout reste libre et partageable.

C'est volontaire : ce projet est offert à la communauté du tir à l'arc, et doit le rester pour tous.

© 2026 rokosmateo. Tir à l'arc — suivi de match par équipe.
