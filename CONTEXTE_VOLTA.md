# ⚔️ VOLTA — Contexte complet du projet (à jour)

## INFORMATIONS GÉNÉRALES
- **Nom du groupe** : Volta
- **Devise** : Honneur · Loyauté · Silence
- **Type** : Faction FiveM / GTA RP, Los Santos
- **Fondateur** : Tony Diaz (membre `tony_diaz`, grade "Fondateur", role admin, protégé — non supprimable)
- **Site web** : https://antho6262.github.io/klan-nostrad-site/
- **Repo GitHub** : https://github.com/Antho6262/klan-nostrad-site
- **Dossier local** : `C:\Users\amalh\Desktop\Klan Nostrad\klan-nostrad-site`

## STACK TECHNIQUE
- Frontend : HTML/CSS/JS vanilla — GitHub Pages (statique)
- Base de données : Firebase Realtime Database (région europe-west1)

## FIREBASE CONFIG (js/firebase-config.js)
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

## PALETTE
Rouge (`#c41e22`) + gris/noir + blanc. Danger = rouge foncé (`#8b1518`). Plus de doré.

## STRUCTURE DU SITEµ
klan-nostrad-site/
├── index.html          ← Connexion — "VOLTA", "Honneur · Loyauté · Silence", trim() mdp, "Mot de passe oublié ?"
├── css/style.css        ← Thème rouge
├── img/background.png + logo.png  ← Logo Volta
├── js/app.js            ← Sidebar "VOLTA", NAV_ITEMS
├── js/firebase-config.js
└── pages/
├── dashboard.html, tracker.html, labo.html, stock.html
├── quotas.html, stats.html, blanchiment.html, paye.html
├── transactions.html, taxes.html, admin.html, profil.html

Pages supprimées : objectifs, sanctions, logs, tv, consommation.

## AUTH
- Firebase Anonymous Authentication activé
- Login (index.html) : après vérif mot_de_passe + actif, `signInAnonymously()` puis écrit `sessions/{uid} = membreId`
- Rules Firebase conditionnent write sur `actif === true` via `root.child('sessions').child(auth.uid)`

## FIREBASE — NŒUDS CLÉS
- `sessions/{uid}` : membreId lié à la session Firebase Auth anonyme
- `membres/{id}` : prenom, nom, grade, mot_de_passe, actif, role, quota
- `grades/{id}` : nom, ordre
- `visibilite_grades/{gradeNom}/{page}` : true/false — page ∈ {quotas, stats, paye, tracker, labo}
- `actions/{semaineId}/{id}` : produit_drogue_id (présent si cat_variable), participants_ids/noms (Armurerie/Fleeca)
- `stock/{catId}/produits/{id}` : nom, prix, stock, seuil, recipe (optionnel Labo)
- `labo_stock/{membreId}/{produitId}` : stock PERSONNEL produits finis
- `labo_stock_commun/{produitId}` : stock COMMUN ingrédients
- `events_drogue/{id}` : debut, fin (timestamps), taux (% drogue), taux_actions (% actions), nom
- `reset_requests/{membreId}` : demandes mot de passe oublié
- `transactions/{id}` : inclut membre_id, prenom_membre
- `config` : blanchiment_taux (35), taux_paye_drogue (20), taux_paye_autres (45)

## QUOTAS
- Global : filtre `!a.produit_drogue_id` (exclut drogue ET labo)
- Par catégorie variable : généré automatiquement
- Logique dupliquée dans `quotas.html` ET `admin.html` onglet Quotas

## TRACKER
- Labo exclu de la liste d'actions
- Armurerie/Fleeca : équipe sans minimum, coéquipiers sans action comptée
- Historique : sélecteur 10/30/50/Tout
- Bannière event animée si event actif selon l'action sélectionnée

## LABO
- Bootstrap auto (crée stock/labo_cat + action "Labo" si absents)
- Ingrédients → stock COMMUN ; Produits finis → stock PERSO
- Catalogue gérable dans Admin → Stock catégorie Labo
- Page Stock connectée à labo_stock_commun

## PAYE
- Sale par défaut. Propre → -blanchiment_taux% auto
- Montant calculé (lecture seule), paiement à 0 autorisé
- Events : taux drogue et/ou actions appliqués par action selon createdAt
- Armurerie/Fleeca : gains divisés entre participants
- Gains arrondis (Math.round)

## BLANCHIMENT
- Bouton Annuler (admin) : supprime + rollback argent

## EVENTS
- Admin → Config → "🎯 Events — Taux spéciaux"
- Bannière animée sur Dashboard et Tracker
- Paye applique le bon taux selon createdAt de chaque action

## MOT DE PASSE OUBLIÉ
- Login → reset_requests / Admin → Membres → panneau 🔔

## ADMIN
- Onglets : Semaines / Membres / Stock / Actions / Quotas / Grades / Visibilité / Permissions / Config
- Visibilité : matrice grade × page (distinct de Permissions)

## POINTS D'ATTENTION
1. Toujours trier en JS, pas via Firebase orderByChild
2. Règles Firebase mode test expirent à 30 jours
3. Solde = cumul depuis le début, pas par semaine
4. Après git push : 60s + Ctrl+F5
5. Quota → modifier dans quotas.html ET admin.html
6. Fichiers modifiés ici ne sont pas auto dans le repo local — télécharger et remplacer manuellement