# 🪙 KLAN NOSTRAD — Contexte complet du projet (à jour)

## INFORMATIONS GÉNÉRALES
- **Nom du groupe** : Klan Nostrad
- **Type** : Faction FiveM / GTA RP — thème "Prophetic Order", Los Santos
- **Fondateur** : Tony Diaz (membre `tony_diaz`, grade "Fondateur", role admin, accès total et protégé — non supprimable)
- **Site web** : https://antho6262.github.io/klan-nostrad-site/
- **Repo GitHub** : https://github.com/Antho6262/klan-nostrad-site
- **Dossier local** : `C:\Users\amalh\Desktop\Klan Nostrad\klan-nostrad-site`
- **Base de départ** : site cloné depuis le projet "La Sombra Del Jaguar" (même architecture), rendu vierge et généralisé, puis enrichi de plusieurs fonctionnalités spécifiques à Klan Nostrad (détail plus bas).

## STACK TECHNIQUE
- Frontend : HTML/CSS/JS vanilla — GitHub Pages (statique)
- Base de données : Firebase Realtime Database (région europe-west1)
- Hébergement : GitHub Pages (gratuit)

## FIREBASE CONFIG (déjà en place dans js/firebase-config.js)
```javascript
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyCDJiMzRhILU2vs1OvUowFW00Fn-of6p_k",
  authDomain: "klan-nostrad.firebaseapp.com",
  databaseURL: "https://klan-nostrad-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "klan-nostrad",
  storageBucket: "klan-nostrad.firebasestorage.app",
  messagingSenderId: "745141888334",
  appId: "1:745141888334:web:de8e93ee0eb2ae39876a1f",
  measurementId: "G-PCJ6QGNDTE"
};
```

## STRUCTURE DU SITE
```
klan-nostrad-site/
├── index.html          ← Page de connexion (logo + sélection membre + mot de passe)
├── setup.html          ← Initialisation Firebase (usage unique, crée Tony Diaz)
├── css/style.css        ← CSS global — thème doré + gris + noir (aucun rouge, retiré sur demande)
├── img/
│   ├── background.png   ← Sceau complet (fond du site, image envoyée par l'utilisateur)
│   └── logo.png          ← Médaillon recadré en cercle (logo sidebar + login)
├── js/
│   ├── firebase-config.js  ← Config Firebase + session + permissions + PAGES_DISPO
│   └── app.js                ← Sidebar (avec logo), NAV_ITEMS, shell commun (initShell)
└── pages/
    ├── dashboard.html     ← temps réel (listeners Firebase .on, pas de polling)
    ├── tracker.html
    ├── stats.html
    ├── stock.html          ← inclut désormais la gestion manuelle de l'argent (sale/propre)
    ├── consommation.html   ← consommation perso (ex: se piquer) — retire du stock sans vente
    ├── logs.html
    ├── quotas.html          ← LECTURE SEULE pour tout le monde (édition déplacée dans Admin)
    ├── objectifs.html
    ├── blanchiment.html
    ├── paye.html             ← paye hebdomadaire (taux drogue / autres actions)
    ├── transactions.html
    ├── taxes.html
    ├── sanctions.html
    ├── admin.html             ← onglets : Semaines / Membres / Stock / Actions / Quotas / Grades / Permissions / Config
    ├── profil.html
    └── tv.html
```

## PALETTE (évolution sur demande successive de l'utilisateur)
1. D'abord acier rgb(176,196,222) + or + noir.
2. Puis rouge rgb(196,30,33) + gris + blanc.
3. Puis écritures passées en doré (#d4af37), rouge gardé sur bordures/boutons.
4. **Rouge totalement retiré** : tout est doré (`--steel` et `--gold` = `#d4af37`) + gris/noir (`--bg-0..3`) + blanc (`--text`). Le "danger" (suppression) utilise un bronze/ambre (`#b8731f`), plus de rouge nulle part.
5. Boutons primaires/gold : fond doré dégradé sombre + texte doré clair qui "ressort" (text-shadow).
6. Logo : médaillon doré recadré depuis `background.png`, dans la sidebar (`.sidebar-coin`) et la page de connexion (`.login-coin`).

## FIREBASE — STRUCTURE DES DONNÉES ACTUELLE
```
membres/{id}        prenom, nom, grade, mot_de_passe, actif, role (admin/membre), quota
grades/{id}          nom, ordre
semaines/{id}        nom, bloquee, createdAt, closedAt (timestamp posé quand bloquée — sert à borner précisément les gains/blanchiments de la semaine)
actions_dispo/{id}   nom, montant_sale, montant_propre, recipe:[{catId,prodId,qty}], cat_variable (optionnel)
  → recipe = consommation fixe de stock (ex: ATM consomme toujours "Boîtier de piratage")
  → cat_variable = catégorie dont le PRODUIT EXACT est choisi par le membre au moment de l'action
    (ex: "Vente de Drogue" → cat_variable = id de la catégorie "Drogue" → le tracker demande "quelle drogue")
actions/{semaineId}/{id}  membre_id, prenom_membre, nom_membre, grade_membre, action, action_id,
                           quantite, date, heure, resultat, raison_echec,
                           argent_sale, argent_propre, gains_totaux, createdAt,
                           produit_drogue, produit_drogue_id (si action à cat_variable)
stock/{catId}        nom, emoji, produits/{id}: nom, prix, stock, seuil
argent/sale/{id}, argent/propre/{id}   date, description, type (Entrée/Sortie), montant, responsable, createdAt
  → alimenté par : actions réussies, blanchiment, transactions, taxes, paye, ET mouvement manuel (page Stock)
  → le "Solde" affiché (Dashboard, Stock) = somme cumulée de TOUTES les lignes depuis le début, pas seulement la semaine en cours
consommations/{id}   membre_id, prenom_membre, catId, prodId, produit_nom, quantite, notes, date, createdAt
blanchiments/{id}    date, description, montant_sale, montant_propre, frais, responsable, createdAt
payes/{semaineId}/{membreId}   montant, type_argent, date, par, createdAt
sanctions/{id}       membre_id, prenom, nom, niveau, motif, par, date
journal_stock/{id}   date, membre, action, quantite, produit
objectifs_config/{id}: cible          objectifs_membres/{membreId}/{id}: cible
quotas_categorie/{membreId}/{catId}: quota   ← quota GLOBAL par catégorie (ex: toutes drogues confondues), pas par produit individuel
transactions/{id}    type(achat/vente), groupe, telephone, produit_id, produit_nom, catId,
                      quantite, prix, type_argent, notes, date
taxes/{id}           groupe, telephone, membre_id, prenom_membre, taxe_id, taxe_nom,
                      montant, type_argent, notes, date
types_taxes/{id}     nom
permissions/{grade}/{page}   true/false (le Fondateur a toujours accès à tout)
config               blanchiment_taux (défaut 35), taux_paye_drogue (défaut 20), taux_paye_autres (défaut 45),
                      discord_webhook_semaine, nom_groupe
```
⚠️ Note historique : un ancien nœud `quotas_produits/{membreId}/{prodId}` (quota par produit individuel) a existé brièvement puis a été remplacé par `quotas_categorie` (quota global toutes drogues confondues) à la demande de l'utilisateur — ignorer `quotas_produits` s'il en reste des traces dans la base.

## ACTIONS IMPORTÉES (bouton "Importer les actions classiques" dans Admin → Actions)
| Action | Effet stock | Type |
|---|---|---|
| Vente de Drogue | -1 du produit choisi (catégorie "Drogue" : Meth, Tranq, Mexicana — noms ajustés par l'utilisateur) | produit variable |
| ATM | -1 Boîtier de piratage | recette fixe |
| Cambu | -1 Outil de crochetage | recette fixe |
| Plantation | -1 Eau, -1 Fertilisant, -1 Graine, -1 Pot | recette fixe |
| Vente de Branches | -1 Branche de cannabis | recette fixe |

Catégories de stock créées par l'import : Drogue 💊, Matériel 🛠️, Plantation 🌱, Branches 🌿.
L'utilisateur a aussi ajouté manuellement "Carte Fleeca" et "Perceuse" dans Matériel.

## MEMBRES ACTUELS (vus dans les dernières captures — vérifier dans Admin, peut changer)
Tony Diaz (Fondateur, admin), Silver (Fondateur, admin), Baba (Oracle, admin), Amir (Lieutenant),
Rick (Officier), Sam (Sergent), Michael (Officier), Robin (Officier), Jack (Disciple).
Quota global = 20 actions/semaine pour tous par défaut.

## PERMISSIONS
- Gérées depuis Admin → onglet Permissions (matrice grade × page).
- Le Fondateur (`role = admin`) a toujours accès à tout, même si rien n'est configuré.
- Suppression (tracker, transactions, taxes, sanctions, consommation, lignes d'argent) réservée à `role = admin`.

## FONCTIONNALITÉS AJOUTÉES PAR RAPPORT À LA BASE VIERGE INITIALE
1. **Produit variable** (`cat_variable`) pour les actions type Vente de Drogue : le tracker demande quel produit exact au moment de l'action.
2. **Import rapide** des 5 actions classiques + catégories de stock associées (Admin → Actions).
3. **Édition des produits de stock** directement dans Admin → Stock (nom/prix/stock/seuil + bouton Enregistrer), en plus de la suppression.
4. **Logo** (médaillon doré) dans la sidebar et la page de connexion.
5. **Page "Consommation"** (menu principal, icône 💉) : trace la consommation perso d'un produit (ex: se piquer) sans vente — retire du stock, historique par membre, suppression admin avec rollback. `consommations/{id}`.
6. **Quotas centralisés dans Admin → onglet Quotas** : quota global (nb d'actions) ET quota drogue (toutes confondues, `quotas_categorie`) édités uniquement par un admin. La page "Quotas" du menu principal est 100% lecture pour tout le monde.
7. **Page "Paye"** (menu principal, icône 💰) : par semaine, calcule les gains de chaque membre séparés en deux : Vente de Drogue (taux `taux_paye_drogue`, défaut 20%) et autres actions + blanchiment dont il est responsable (taux `taux_paye_autres`, défaut 45%) — taux réglables dans Admin → Config. Bouton "Payer" → débite `argent/sale` ou `argent/propre` et journalise dans `payes/{semaineId}/{membreId}`.
8. **Mouvement manuel d'argent** intégré à la page Stock (comme dans Sombra) : soldes sale/propre affichés, formulaire Compte/Sens/Montant/Description/Responsable, historique des mouvements avec suppression admin.
9. **Dashboard en temps réel** : remplace l'ancien polling 60s par des listeners Firebase `.on('value')` sur semaines/argent/stock/membres/actions — tout se met à jour instantanément sans recharger la page.
10. **Actions complètes sur les membres** (Admin → Membres) : prénom/nom/grade/quota directement éditables en ligne (bouton 💾 Enregistrer), + icônes avec tooltip au survol : 🔑 changer le mot de passe, 🛡️ passer/retirer admin, ⏸️/▶️ activer/désactiver, 🗑️ supprimer (sauf Tony Diaz, protégé).

## AJOUTER UNE NOUVELLE PAGE — CHECKLIST
1. Créer `pages/ma-page.html` (copier structure existante, shell `await initShell('ma-page', 'Titre')`).
2. `js/app.js` → ajouter dans `NAV_ITEMS` : `{ page, icon, label, file }`.
3. `js/firebase-config.js` → ajouter dans `PAGES_DISPO` : `{ page, label }`.
4. Configurer les accès dans Admin → Permissions.

## BUGS / POINTS D'ATTENTION
1. `orderByChild` Firebase nécessite un index → toujours trier en JS (déjà fait partout).
2. `snap.forEach` ne retourne qu'un élément si index manquant → ce site utilise systématiquement `entries(snap.val())`.
3. Règles Firebase en mode test : expirent au bout de 30 jours → à restreindre/renouveler dans la console Firebase.
4. Le "Solde sale/propre" (Dashboard, Stock) est un **cumul depuis le début**, pas un solde "de la semaine" — ne pas confondre avec "Gains totaux — semaine" qui ne compte que les actions de la semaine active.
5. Emojis complexes dans la nav → préférer des emojis simples (déjà respecté).

## POUR REPRENDRE LA CONVERSATION
Colle ce fichier au début d'un nouveau chat avec Claude, en précisant ce que tu veux faire ensuite.
Si tu as toujours le fichier `klan-nostrad-site.zip` ou le repo GitHub à jour, tu peux aussi le repartager directement.
