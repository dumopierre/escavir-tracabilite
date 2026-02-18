# ESCAVIR srl — Documentation Application Traçabilité

> **Version actuelle :** v2.2  
> **URL :** https://dumopierre.github.io/escavir-tracabilite  
> **Dernière mise à jour :** 18 février 2026

---

## 📋 TABLE DES MATIÈRES

1. [Présentation de l'application](#1-présentation)
2. [Cahier des charges](#2-cahier-des-charges)
3. [Architecture technique](#3-architecture-technique)
4. [Modules & fonctionnalités](#4-modules--fonctionnalités)
5. [Journal des modifications](#5-journal-des-modifications)

---

## 1. Présentation

**ESCAVIR srl** est une entreprise artisanale spécialisée dans les produits de la mer (poisson fumé, plats cuisinés, rillettes, soupes…).

L'application **ESCAVIR Traçabilité** est un outil web de traçabilité AFSCA développé sur mesure pour gérer :
- Les fournisseurs et achats de matières premières
- Les productions de produits finis
- Les stocks
- La traçabilité bidirectionnelle (MP → Produit fini et inversement)
- Les rapports et exports CSV conformes AFSCA

L'application fonctionne entièrement **dans le navigateur** (pas de serveur), avec les données stockées en **localStorage** sur l'appareil. Elle est accessible via GitHub Pages et installable comme application mobile (PWA).

---

## 2. Cahier des charges

### 2.1 Contexte réglementaire

- Conformité **AFSCA** (Agence Fédérale pour la Sécurité de la Chaîne Alimentaire)
- Règlement **CE 178/2002** — traçabilité alimentaire obligatoire
- Traçabilité **un pas en avant / un pas en arrière** (fournisseur → production → client)
- Numéros de lot, dates de réception, DLC/DDM obligatoires

### 2.2 Utilisateurs

- Utilisateur unique (responsable traçabilité ESCAVIR)
- Accès multi-appareils (PC, tablette, smartphone)
- Pas de gestion multi-utilisateurs requise

### 2.3 Contraintes techniques

| Critère | Choix |
|---|---|
| Hébergement | GitHub Pages (gratuit) |
| Technologie | HTML/CSS/JS pur (1 fichier) |
| Stockage | localStorage (navigateur) |
| Connexion | Non requise (hors-ligne possible) |
| Installation | PWA — icône sur écran d'accueil |

### 2.4 Exigences fonctionnelles

| Priorité | Module | Description |
|---|---|---|
| ✅ P1 | Design & charte | Logo ESCAVIR, couleurs marine/cyan |
| ✅ P2 | Fournisseurs | Gestion des fournisseurs agréés AFSCA |
| ✅ P2 | Achats MP | Réception matières premières multi-catégories |
| ✅ P2 | Productions | Fabrication produits finis avec MP liées |
| ✅ P2 | Recettes | Association MP ↔ produits finis |
| ✅ P2 | Stocks | Suivi des stocks MP et produits finis |
| ✅ P2 | Traçabilité | Recherche bidirectionnelle par lot |
| ✅ P2 | Rapports | Exports CSV conformes AFSCA |
| 🔄 P3 | Sync données | Synchronisation entre appareils |
| 🔄 P4 | Mobile | Optimisation expérience smartphone |

---

## 3. Architecture technique

### 3.1 Structure du fichier

```
index.html (≈ 513 KB)
├── <head>          — PWA manifest, icônes base64, polices Google Fonts
├── <style>         — CSS complet (variables, composants, responsive)
├── <body>
│   ├── #login-screen    — Écran d'accueil (sessionStorage)
│   ├── .app-bg          — Background animé (grille + poisson + logo)
│   ├── .sidebar         — Navigation latérale
│   ├── .main
│   │   ├── .topbar      — Barre supérieure
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

### 3.2 Stockage localStorage

| Clé | Contenu |
|---|---|
| `esc_four` | Tableau des fournisseurs |
| `esc_achats` | Tableau des achats MP |
| `esc_prods` | Tableau des productions |
| `esc_produits` | Liste des produits finis personnalisés |
| `esc_recettes` | Objet recettes {produit: [MP...]} |
| `esc_cfg` | Configuration société |

### 3.3 Charte graphique

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

---

## 4. Modules & Fonctionnalités

### 4.1 Fournisseurs
- Ajout / suppression de fournisseurs
- Champs : Société, Contact, Téléphone, Email, Type (fournisseur/transporteur/prestataire/multi-produits), N° agrément AFSCA, Notes
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

**Champs communs :** Désignation (avec autocomplétion), Fournisseur, N° lot, N° bon de livraison, Date de réception, DLC/DDM, **Pays d'origine du produit**, Quantité, Prix, Notes

### 4.3 Productions
- Produit fini (liste personnalisable + ajout rapide **"+ Nouveau"**)
- Matières premières utilisées (sélection multiple)
- **Pré-sélection automatique des MP** selon la recette définie
- Date de production, DLC produit fini
- Quantité en pièces + poids approximatif par pièce
- Notes

### 4.4 Recettes *(nouveau module)*
- Interface deux panneaux : liste produits / détail recette
- Association libre MP ↔ produit fini
- Pré-sélection automatique lors des productions
- Badge indiquant le nombre de MP par recette

### 4.5 Stocks
- Vue consolidée MP restantes et produits finis
- Alertes DLC (expiré / urgent / OK)

### 4.6 Traçabilité
- Recherche par N° lot, désignation, fournisseur
- Résultat : arbre MP → productions utilisant cette MP
- Traçabilité bidirectionnelle conforme AFSCA

### 4.7 Rapports
- Export CSV : achats, productions, fournisseurs
- Rapport complet traçabilité

### 4.8 Paramètres
- Gestion liste produits finis (ajout/suppression)
- Informations société (nom, agrément AFSCA, adresse, responsable)
- Réinitialisation des données

---

## 5. Journal des modifications

### Session du 18 février 2026

#### 🎨 Design & charte graphique
- **Intégration logo ESCAVIR** (base64 inline) dans sidebar, topbar et écran de connexion
- **Palette marine professionnelle** : fonds `#05080f → #0d1526`, accent cyan `#00aadd`
- **Écran de connexion** : carte animée avec fond lignes + poisson flottant, badge AFSCA
- **Background app** : grille cyan subtile + poisson filigrane + logo watermark (visible après connexion)
- **sessionStorage** : écran de connexion affiché une seule fois par session
- **PWA** : icônes 192×192 et 512×512 pour installation smartphone

#### 🏷️ Fournisseurs
- ❌ Suppression du champ **"Pays d'origine"** (non pertinent sur la fiche fournisseur)

#### 🛒 Achats MP
- ➕ Ajout du champ **"Pays d'origine du produit"** (après DLC/DDM) — logiquement c'est le pays du produit, pas du fournisseur
- 🔀 **Fusion Désignation + Variété** : suppression du champ "Variété" (redondant), la Désignation devient un champ avec **autocomplétion intelligente** (datalist alimenté par les désignations déjà utilisées)
- 📋 Nouvelle colonne "Pays origine" dans le tableau des achats

#### ⚙️ Productions
- ➕ Bouton **"+ Nouveau"** à côté du select Produit fini pour ajouter un produit à la volée sans quitter le formulaire
- 📋 Liste des produits triée alphabétiquement
- 🔗 **Pré-sélection automatique des MP** selon la recette définie pour le produit choisi

#### 📋 Recettes *(nouveau module)*
- ➕ **Nouvel onglet "Recettes"** dans la sidebar (section Production)
- Interface deux panneaux : liste produits à gauche, détail recette à droite
- Ajout/suppression d'ingrédients par recette
- Compteur MP affiché par produit
- Synchronisé avec le formulaire de production (pré-sélection automatique)

#### 🔧 Corrections techniques
- Fix syntaxe JS : `refreshDesigList()` mal placé dans le bloc `if/else`
- Fix apostrophes non échappées dans template literals JS
- Fix guillemets dans alert (`d'abord`)

---

## 🔄 Prochaines étapes planifiées

| Priorité | Tâche |
|---|---|
| 🔄 P3 | Synchronisation des données entre appareils |
| 🔄 P4 | Optimisation expérience mobile |
| 💡 Idée | Modifier un achat/production existant |
| 💡 Idée | Notifications DLC proches |

---

*Document généré automatiquement — ESCAVIR srl © 2026*
