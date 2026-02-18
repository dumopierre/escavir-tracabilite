# ESCAVIR srl — Documentation Application Traçabilité

> **Version actuelle :** v3.0  
> **URL :** https://dumopierre.github.io/escavir-tracabilite  
> **Dépôt GitHub :** https://github.com/dumopierre/escavir-tracabilite  
> **Base de données :** Firebase Realtime Database (europe-west1 — Belgique)  
> **Dernière mise à jour :** 18 février 2026

---

## 📋 TABLE DES MATIÈRES

1. [Présentation de l'application](#1-présentation)
2. [Cahier des charges](#2-cahier-des-charges)
3. [Architecture technique](#3-architecture-technique)
4. [Modules & fonctionnalités](#4-modules--fonctionnalités)
5. [Journal des modifications](#5-journal-des-modifications)
6. [Guide d'utilisation](#6-guide-dutilisation)

---

## 1. Présentation

**ESCAVIR srl** est une entreprise artisanale spécialisée dans les produits de la mer (poisson fumé, plats cuisinés, rillettes, soupes…).

L'application **ESCAVIR Traçabilité** est un outil web de traçabilité AFSCA développé sur mesure pour gérer :
- Les fournisseurs et achats de matières premières
- Les productions de produits finis
- Les recettes avec calcul de coûts et marges
- Les stocks / tableau de bord MP
- La traçabilité bidirectionnelle (MP → Produit fini et inversement)
- Les rapports et exports CSV conformes AFSCA

L'application fonctionne entièrement **dans le navigateur** (pas de serveur), avec les données synchronisées en temps réel via **Firebase Realtime Database**. Elle est accessible via GitHub Pages et installable comme application mobile (PWA).

---

## 2. Cahier des charges

### 2.1 Contexte réglementaire

- Conformité **AFSCA** (Agence Fédérale pour la Sécurité de la Chaîne Alimentaire)
- Règlement **CE 178/2002** — traçabilité alimentaire obligatoire
- Traçabilité **un pas en avant / un pas en arrière** (fournisseur → production → client)
- Numéros de lot, dates de réception, DLC/DDM obligatoires

### 2.2 Utilisateurs

- Utilisateur principal ESCAVIR (responsable traçabilité)
- Accès multi-appareils simultané (PC, tablette, smartphone)
- Synchronisation temps réel entre appareils

### 2.3 Contraintes techniques

| Critère | Choix |
|---|---|
| Hébergement | GitHub Pages (gratuit) |
| Technologie | HTML/CSS/JS pur (1 fichier) |
| Stockage cloud | Firebase Realtime Database (gratuit) |
| Stockage local | localStorage (fallback hors-ligne) |
| Connexion | Optionnelle — fonctionne hors-ligne |
| Installation | PWA — icône sur écran d'accueil |

### 2.4 Exigences fonctionnelles

| Statut | Module | Description |
|---|---|---|
| ✅ | Design & charte | Logo ESCAVIR, couleurs marine/cyan |
| ✅ | Fournisseurs | Gestion des fournisseurs agréés AFSCA |
| ✅ | Achats MP | Réception matières premières multi-catégories |
| ✅ | Productions | Fabrication produits finis avec MP liées |
| ✅ | Recettes | Association MP ↔ produits finis + calcul coûts/marges |
| ✅ | Stocks / TB MP | Tableau de bord MP avec alertes DLC |
| ✅ | Traçabilité | Recherche bidirectionnelle multi-résultats |
| ✅ | Rapports | 6 exports CSV + impression PDF |
| ✅ | Multi-support | Synchronisation Firebase temps réel |
| 🔄 | Authentification | Sécurisation accès (optionnel) |

---

## 3. Architecture technique

### 3.1 Structure du fichier

```
index.html (≈ 537 KB)
├── <head>          — PWA manifest, icônes base64, polices Google Fonts
├── Firebase SDK    — firebase-app-compat + firebase-database-compat
├── <style>         — CSS complet (variables, composants, responsive)
├── <body>
│   ├── #login-screen    — Écran d'accueil (sessionStorage)
│   ├── .app-bg          — Background animé (grille + poisson + logo)
│   ├── .sidebar         — Navigation latérale
│   ├── .main
│   │   ├── .topbar      — Barre supérieure + indicateur sync ☁
│   │   └── .content
│   │       ├── #view-dashboard
│   │       ├── #view-fournisseurs
│   │       ├── #view-achats
│   │       ├── #view-productions
│   │       ├── #view-recettes
│   │       ├── #view-stocks
│   │       ├── #view-tracabilite
│   │       ├── #view-rapports
│   │       └── #view-parametres
│   └── Modales (achat, production)
└── <script>        — Logique JS complète
```

### 3.2 Stockage & synchronisation

```
┌──────────────┐    DB.set()     ┌──────────────────────────┐
│  Navigateur  │ ──────────────▶ │  Firebase Realtime DB     │
│ localStorage │ ◀ ────────────  │  (europe-west1 / BE) 🇧🇪  │
└──────────────┘  onValue() sync └──────────────────────────┘
       ▲                                      ▲
       │ fallback hors-ligne                  │ temps réel
  PC / Mac                           Smartphone / Tablette
```

**Clés Firebase / localStorage :**

| Clé | Type | Contenu |
|---|---|---|
| `four` | Array | Fournisseurs |
| `achats` | Array | Achats MP |
| `prods` | Array | Productions |
| `produits` | Array | Liste produits finis personnalisés |
| `recettes` | Object | Ingrédients par recette `{produit: [MP...]}` |
| `recettes_cfg` | Object | Config coûts/prix par recette |
| `cfg` | Object | Configuration société |

### 3.3 Couche DB (DB Layer)

```javascript
DB.get(k)     // Lit depuis cache → localStorage
DB.getObj(k)  // Idem pour objets
DB.set(k, v)  // Écrit cache + localStorage + Firebase
```

Comportement offline : si Firebase est indisponible, l'app fonctionne normalement en localStorage. La synchronisation reprend automatiquement à la reconnexion.

### 3.4 Charte graphique

| Élément | Valeur |
|---|---|
| Fond principal | `#05080f` |
| Fond secondaire | `#080d1a` |
| Surface | `#0d1526` |
| Accent cyan ESCAVIR | `#00aadd` |
| Texte principal | `#dce8f8` |
| Police titres | Syne (Google Fonts) |
| Police corps | Epilogue (Google Fonts) |
| Police monospace | JetBrains Mono (Google Fonts) |

### 3.5 Firebase

| Paramètre | Valeur |
|---|---|
| Projet | `escavir-tracabilite` |
| Service | Realtime Database |
| Région | europe-west1 (Belgique) |
| URL | `https://escavir-tracabilite-default-rtdb.europe-west1.firebasedatabase.app` |
| Nœud racine | `/escavir` |
| Plan | Gratuit (Spark) |

---

## 4. Modules & Fonctionnalités

### 4.1 Fournisseurs
- Ajout / suppression de fournisseurs
- Champs : Société, Contact, Téléphone, Email, Type, N° agrément AFSCA, Notes
- Tableau récapitulatif avec badges colorés par type

### 4.2 Achats MP — 8 catégories

| Catégorie | Champs spécifiques |
|---|---|
| 🐟 Poisson | Espèce, origine, température, conditionnement |
| 🥬 Légume frais | Origine pays, température, conditionnement |
| ❄️ Légume surgelé | Température, conditionnement |
| 🌿 Épices & aromates | Type, conditionnement, allergènes |
| 🥛 Produits laitiers | Type, traitement thermique, température, conditionnement |
| 🌾 Pâtes & farines | Type, conditionnement, allergènes |
| 📦 Emballages | Type, référence, matériau |
| 🔹 Autre | Type libre, origine |

Champs communs : Désignation (autocomplétion intelligente), Fournisseur, N° lot, N° BL, Date de réception, DLC/DDM, Pays d'origine du produit, Quantité, Prix, Notes

### 4.3 Productions
- Produit fini (liste personnalisable + bouton "+ Nouveau" pour ajout rapide)
- Matières premières utilisées (sélection multiple)
- Pré-sélection automatique des MP selon la recette définie
- Date de production, DLC produit fini, quantité en pièces, poids par pièce, notes

### 4.4 Recettes & Coûts

Interface deux panneaux (liste produits / détail recette) avec :

**Ingrédients** : tableau des MP avec quantités, unités et prix auto depuis le dernier achat

**Coûts additionnels** :
- Emballage (€ / batch)
- Temps de production × taux horaire = coût main d'œuvre automatique
- Frais fixes (énergie, eau, amortissement)
- Rendement : nombre de pièces ou kg par batch

**Résumé financier** :

| Indicateur | Calcul |
|---|---|
| Coût total batch | MP + emballage + MO + frais fixes |
| Coût / pièce ou / kg | Coût total ÷ rendement |
| Prix de vente | Saisie manuelle |
| Marge € | Prix de vente − coût unitaire |
| Marge % | (PV − coût) / PV × 100 |

Mode de vente configurable par produit : à la pièce (quiches, rillettes…) ou au poids/kg (saumon fumé…)

### 4.5 Tableau de bord MP (Stocks)

- 4 KPI : lots expirés ❌ / urgents ⚠️ (≤3j) / proches ⏳ (≤7j) / OK ✓
- Filtre par catégorie ou "Alertes uniquement"
- Tableau : stock initial, utilisations en production, restant disponible, pays d'origine
- Tri automatique : expirés en premier

### 4.6 Traçabilité bidirectionnelle

Recherche par : désignation, N° lot, N° BL, fournisseur, pays d'origine, nom de production

- Aval (MP → Productions) : tous les lots + toutes les productions où la MP a été utilisée
- Amont (Production → MP) : toutes les MP utilisées + leurs fournisseurs
- Tous les résultats correspondants affichés (pas de limitation)
- Badge compteur de résultats

### 4.7 Rapports — 6 exports

| Rapport | Format | Contenu |
|---|---|---|
| Registre Achats MP | CSV | Toutes catégories avec détails complets |
| Registre Productions | CSV | Lots, produits, MP liées, DLC |
| Traçabilité complète | CSV | Chaîne amont/aval par production |
| Registre Fournisseurs | CSV | Liste avec agréments AFSCA |
| Alertes DLC | CSV | Lots expirés ou ≤ 7 jours, triés par urgence |
| Rapport PDF complet | Impression | Document pour inspecteurs AFSCA |

### 4.8 Paramètres
- Gestion liste produits finis (ajout/suppression)
- Informations société (nom, agrément AFSCA, adresse, responsable)
- Réinitialisation des données par module

---

## 5. Journal des modifications

### Session du 18 février 2026

#### v1.0 — Design & Infrastructure
- Intégration logo ESCAVIR (base64 inline)
- Palette marine professionnelle, accent cyan `#00aadd`
- Écran de connexion animé (grille + poisson flottant)
- Background app : grille + poisson + logo watermark
- PWA : icônes 192×192 et 512×512
- Hébergement GitHub Pages

#### v1.1 — Achats MP
- Champ "Pays d'origine" déplacé du fournisseur vers les achats
- Fusion Désignation + Variété avec autocomplétion intelligente
- Nouvelle colonne "Pays origine" dans le tableau

#### v1.2 — Productions
- Bouton "+ Nouveau" pour ajouter un produit à la volée
- Liste produits triée alphabétiquement
- Pré-sélection automatique des MP selon recette

#### v1.3 — Recettes (module initial)
- Nouvel onglet "Recettes" dans la sidebar
- Interface deux panneaux
- Association ingrédients par produit

#### v2.0 — Recettes & Coûts (module complet)
- Tableau ingrédients avec quantités, unités, prix auto depuis achats
- Coûts : emballage, main d'œuvre, frais fixes
- Mode vente : à la pièce ou au poids selon produit
- Calcul automatique coût total, coût unitaire, marge €/%

#### v2.1 — Tableau de bord MP
- Refonte onglet Stocks → tableau de bord MP
- 4 KPI alertes, filtre par catégorie, tri par urgence DLC

#### v2.2 — Traçabilité & Rapports
- Traçabilité multi-résultats (tous les lots)
- Recherche étendue : désignation, lot, BL, fournisseur, pays
- Ajout Registre Fournisseurs (CSV)
- Ajout Alertes DLC (CSV)

#### v3.0 — Multi-support & Synchronisation
- Intégration Firebase Realtime Database (europe-west1)
- Synchronisation temps réel entre appareils
- Indicateur "☁ Sync OK" / "⚠ Hors-ligne" dans la topbar
- Fallback localStorage si hors-ligne

#### Corrections techniques
- Fix `refreshDesigList()` dans `if/else`
- Fix apostrophes dans template literals
- Fix chaînes JS adjacentes sans opérateur `+`
- Fix `DB` déclaré en double après migration Firebase
- Fix scripts Firebase chargés après le code JS

---

## 6. Guide d'utilisation

### 6.1 Accès

| Appareil | Action |
|---|---|
| PC / Mac | https://dumopierre.github.io/escavir-tracabilite |
| Smartphone Android | Chrome → ⋮ → "Ajouter à l'écran d'accueil" |
| iPhone / iPad | Safari → Partager → "Sur l'écran d'accueil" |

### 6.2 Workflow quotidien recommandé

```
1. RÉCEPTION MP
   └─ Achats MP → Nouvelle entrée → Lot, désignation, DLC, pays

2. PRODUCTION
   └─ Productions → Nouvelle production → Produit (MP pré-sélectionnées)

3. CONTRÔLE DLC (chaque matin)
   └─ Stocks → Alertes rouges/orange en haut de page

4. TRAÇABILITÉ (contrôle AFSCA)
   └─ Traçabilité → Taper désignation ou n° lot

5. EXPORT MENSUEL
   └─ Rapports → Achats + Productions + Traçabilité en CSV
```

### 6.3 Configuration initiale

1. **Paramètres** → Remplir les informations société
2. **Paramètres** → Vérifier la liste des produits finis
3. **Fournisseurs** → Encoder tous vos fournisseurs habituels
4. **Recettes** → Définir ingrédients et coûts pour chaque produit

### 6.4 Firebase — Règles de sécurité

```json
{
  "rules": {
    "escavir": {
      ".read": true,
      ".write": true
    }
  }
}
```

> Pour sécuriser davantage, ajouter une authentification Firebase (email/mot de passe).

---

## 🔄 Prochaines étapes possibles

| Idée | Description |
|---|---|
| 🔐 Authentification | Login email/mot de passe Firebase |
| ✏️ Édition | Modifier un achat ou une production existante |
| 🔔 Notifications | Alertes push DLC sur smartphone |
| 📊 Dashboard financier | Coûts & marges globaux par période |
| 📄 Export recettes | Fiche recette en PDF |

---

*Document généré automatiquement — ESCAVIR srl © 2026*
