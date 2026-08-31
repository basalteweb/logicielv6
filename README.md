# Analysis Power — Basalte-Web

**Version 5.1.0 MERCHANT** — pilotage commercial local pour les extractions TGM / Rezomatic.

Analysis Power a été conçu autour d'une règle simple : **le commerçant ne doit pas avoir besoin de comprendre les calculs pour comprendre son magasin**.

Le moteur peut effectuer des contrôles complexes en arrière-plan ; l'interface restitue toujours :

> **Ce qui se passe → Pourquoi ? → Que faire ? → Voir les faits**

## Ce que Power fait

Après l'import de Clients, Ventes et Catalogue, Power :

- vérifie que les fichiers peuvent être reliés sans ambiguïté bloquante ;
- compare chiffre d'affaires, tickets, panier, marge, clients, produits, rayons et stock ;
- détecte les clients qui tardent réellement par rapport à leur propre rythme ;
- repère les produits qui progressent, reculent, risquent de manquer ou se vendent très peu ;
- analyse les zones d'origine des clients ;
- confronte les problèmes commerciaux au contexte local disponible ;
- propose des actions classées par priorité ;
- conserve les preuves détaillées dans l'audit.

## Interface commerçant

Les vues principales n'affichent pas les scores causaux, coefficients, seuils ou formules internes.

Chaque carte doit être compréhensible immédiatement :

- **Ce qui se passe** : constat en français courant ;
- **Pourquoi ?** : cause retenue si elle est suffisamment solide ;
- **Que faire ?** : prochaine action concrète ;
- **Voir les faits** : données précises utilisées pour conclure.

Si aucune cause n'est assez solide, Power affiche explicitement **« Cause non identifiée »**.

Les calculs et hypothèses rejetées restent dans **Détails & audit**.

Voir `docs/MERCHANT_UX.md` pour les règles complètes.

## Causalité : règle de sécurité

Une information extérieure peut être collectée sans être présentée comme une cause.

Un chantier, un événement, la météo, le stationnement ou T2C ne remontent dans une explication que lorsqu'ils sont cohérents avec le type de problème, le lieu, la période et les données commerciales observées.

### Travaux

Pour afficher un chantier comme explication, Power demande notamment :

- une source officielle ou suffisamment fiable ;
- une période exploitable ;
- une concordance avec la zone touchée ;
- une preuve de fréquentation autour du chantier **ou** une restriction d'accès forte et proche ;
- une cohérence avec le mouvement commercial observé.

Le descriptif officiel est conservé séparément de l'interprétation Power.

### Produits

Une baisse d'un produit isolé ne peut pas être directement attribuée aux travaux, à la météo, au parking, à T2C ou au simple nombre d'événements locaux. Power recherche d'abord les explications métier : clients acheteurs, quantité, prix, stock, substitution vers une autre référence, famille et rayon.

## Local Context Engine

Le profil du point de vente contient son nom et son adresse. L'adresse du **commerce** peut être géocodée via Géoplateforme / Base Adresse Nationale afin de mesurer la proximité des informations publiques.

Le collecteur fourni avec cette édition utilise notamment, selon disponibilité :

- Clermont Auvergne Métropole Open Data ;
- pages officielles des travaux ;
- agenda public ;
- T2C GTFS-RT ;
- stationnement ;
- C.vélo / comptages ;
- Open-Meteo ;
- calendrier Zone A / jours fériés.

Les adresses clients ne sont jamais envoyées aux services publics de géocodage.

## Import TGM

Fichiers acceptés :

- `Clients.xlsx`, `Clients(3).xlsx`, etc. ;
- un ou plusieurs `Ventes.csv`, `Ventes(1).csv`, etc. ;
- `Catalogue.xlsx`, `Catalogue(14).xlsx`, etc.

Les fichiers TGM sont traités localement dans le navigateur. Ils ne sont pas envoyés à GitHub ou Basalte-Web.

## Confidentialité

- données Clients / Ventes / Catalogue : traitement local navigateur ;
- stockage local possible via IndexedDB ;
- `.gitignore` bloque CSV/XLS/XLSX/ODS/ZIP pour limiter les commits accidentels ;
- l'adresse du commerce peut être envoyée au géocodeur public ;
- aucune adresse client n'est transmise aux API publiques.

## Installation GitHub Pages

1. Mettre tout le contenu du dossier à la racine du dépôt.
2. Vérifier `config/store.json` si le collecteur planifié doit utiliser une adresse précise.
3. Activer GitHub Pages sur `main` / `/ (root)`.
4. Dans Actions, lancer une première fois **Analysis Power · Local Context Sentinel**.
5. Si le workflow doit mettre à jour les JSON : `Settings → Actions → General → Workflow permissions → Read and write permissions`.
6. Vérifier que l'application affiche **v5.1.0 MERCHANT**.

Aucune compilation front-end n'est nécessaire.

## Tests fournis

- `node tests/test_causal_context.js`
- `node tests/test_power_causality.js`
- `node tests/test_merchant_rules.js`
- `python tests/test_public_context.py`
- `python scripts/validate_public_context.py`
- `node --check js/*.js sw.js`
- `python -m py_compile scripts/*.py`

`test_merchant_rules.js` empêche notamment le retour de deux faux raisonnements :

1. un simple écart du nombre d'événements locaux ne peut pas devenir une cause ;
2. un chantier ne peut pas être retenu uniquement parce qu'il se trouve dans le bon secteur alors que la baisse commerciale avait déjà commencé avant lui.
