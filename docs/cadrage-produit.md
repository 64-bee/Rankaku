# Rankaku : cadrage produit

Document issu de la session de cadrage. Il sert de base à la rédaction du `GLOSSARY.md`, des ADR et des issues. Rien ici n'est encore un ticket.

## 1. Vision

Rankaku est une application de **watchlist d'animés** qui débouche sur des **tier lists**. Le cas d'usage de départ : après avoir fini un animé, on le range dans une tier list, et on en débat (avec sa copine, à l'origine du projet).

À terme :

- une **tier list globale et communautaire**, alimentée par les tier lists publiques des utilisateurs ;
- un fonctionnement à la Reddit : des **topics** auxquels on s'abonne, des **votes** sur les classements (« ça ne mérite pas B, plutôt D »), des **commentaires** ;
- d'autres médias de pop culture (jeux vidéo, séries) et d'autres thèmes de tier lists (personnages, OST, openings), plus ouverts et communautaires.

Priorité actuelle : **la watchlist d'animés uniquement**.

## 2. Public et objectif du projet

- Utilisateurs de départ : l'auteur et sa copine. Ouverture au public pop culture ensuite.
- C'est avant tout un **projet d'apprentissage et de démonstration de compétences**. Cela justifie : une architecture hexagonale, une réflexion en amont sur la scalabilité (surtout côté back) et une vraie base PostgreSQL dès le début.

## 3. Décisions techniques

| Sujet | Décision |
|---|---|
| Front | Angular, en **PWA installable** sur mobile (pas de publication Play Store pour l'instant) |
| Back | Spring Boot, **Java 21**, architecture **hexagonale** (domaine, ports, adaptateurs) |
| Base de données | **PostgreSQL**, dès le lot 1 |
| Langue | **Français uniquement** pour l'instant, avec l'i18n préparée (voir 3.1) |
| Données externes | Uniquement des **API publiques et gratuites**, animés seulement au début |
| Déploiement | Docker Compose en local. Minikube, Docker et chart Helm **après le POC** |
| CI et tests | GitHub Actions, tests unitaires du domaine, tests d'intégration avec Testcontainers |

### 3.1 Préparer l'i18n sans gros refactoring

- Aucun texte en dur dans les composants Angular : tout passe par un système de traduction (Transloco ou `@angular/localize`) avec un seul fichier `fr`.
- Le back renvoie des **codes d'erreur stables** (par exemple `ANIME_NOT_FOUND`), pas des phrases.
- Dates et nombres formatés selon la locale.

### 3.2 Authentification

- **Lot 1 : aucune authentification.** Un utilisateur implicite unique (`UserId` fixe) est déjà présent dans le modèle, avec un `user_id` sur les tables concernées, pour éviter une migration lourde plus tard.
- **Plus tard : Google OAuth 2.** Pas de gestion de mots de passe.
- Un seul profil en lot 1 : la séparation des données entre l'auteur et sa copine arrive avec l'OAuth.

### 3.3 Persistance

La watchlist est stockée **dans PostgreSQL via l'API** dès le lot 1 (et non dans le navigateur). L'application est utilisée **en local sur le PC** pendant le lot 1. L'accès depuis les téléphones vient avec le déploiement Minikube, après le POC.

## 4. Lot 0 : POC des API de données

**Objectif** : appeler les API candidates, inspecter toutes les informations qu'elles renvoient, et décider si on peut s'appuyer dessus. Les limites de débit ne sont pas un critère pour l'instant. Le critère principal est **la qualité et la richesse des données**.

**Forme** : un outil jetable, séparé de `apps/api`, qui appelle chaque fournisseur avec le même jeu d'animés de test (un classique, un récent, un titre ambigu, un film, une suite) et enregistre les réponses brutes. Il produit un **rapport Markdown dans `docs/`** avec le positif et le négatif de chaque source. Le rapport sert de base à une ADR sur le choix de la source.

**Candidats** : AniList, Jikan (MyAnimeList), Kitsu, TMDB (clé gratuite), Shikimori, plus un flux RSS pour les actualités.

**À vérifier en particulier** :

- recherche par titre en tolérant les variantes (romaji, anglais, français) ;
- fiche : synopsis, image, nombre d'épisodes, statut de diffusion, studio, genres, saisons et suites liées ;
- planning de diffusion ou sorties de la saison en cours ;
- tendances et popularité (pour les recommandations et l'onboarding) ;
- actualités (peuvent venir d'une source séparée).

Hors périmètre : « où regarder » (streaming).

**Conception** : le catalogue est accessible derrière un port (par exemple `AnimeCatalogPort`), pour pouvoir changer de source.

## 5. Lot 1 : watchlist et onboarding

### 5.1 Le Kanban

Colonnes :

1. **À voir** : les envies de l'utilisateur.
2. **En cours**.
3. **À jour** : l'animé est encore diffusé et l'utilisateur a vu tous les épisodes sortis. **Posée à la main** par glisser-déposer, puisque l'appli ne propose pas la lecture et ne peut pas savoir ce qui a été vu.
4. **Terminés**.
5. **Pause / Abandon** : colonne à part, avec deux sous-statuts (pause et abandon) distingués par un badge. Un animé abandonné peut quand même être évalué (« c'est de la daube »).

Les saisons et suites sont traitées comme des animés distincts, reliés entre eux par la fiche.

Reporté : la progression par épisode détaillée et les notes texte et tags (à reconsidérer). La pré-note (note provisoire pendant le visionnage) est **reportée**.

On peut passer directement de « À voir » à « Terminés » (animé vu avant l'appli).

### 5.2 Évaluation : le palier remplace la note

- Pas de note sur 10. **Le palier de la tier list (S à D) est l'évaluation.**
- À la fin d'un animé, **aucune note n'est forcée**. L'animé rejoint un backlog **« à classer »**. Un badge dans l'onglet tier list indique par exemple « 10 animés en attente d'être classés ».
- Les animés non finis (abandonnés) sont classables, avec un marqueur « pas fini » visible dans la tier list. Leur traitement dans le classement global est à décider plus tard.

### 5.3 Recommandations

- Une **zone séparée du Kanban**, façon Netflix, avec des rangées : *Tendances du moment*, *Saison en cours*, *Populaires de tous les temps*.
- Son emplacement à l'écran sera décidé au moment du design.
- Ajout à « À voir » depuis la recherche ou depuis ces rangées.
- **Reportées** : les rangées personnalisées (« parce que tu as regardé X »), qui demandent de l'historique et un algorithme.

### 5.4 Onboarding par swipe

Pour remplir son historique rapidement, l'utilisateur swipe des cartes d'animés connus :

| Geste | Effet |
|---|---|
| Droite (vu) | L'animé passe en *Terminés* et rejoint le backlog « à classer » |
| Haut (pas fini) | Question rapide : *En cours* ou *Abandonné* |
| Gauche (pas vu) | L'animé est écarté, rien n'est ajouté |

- Source des cartes : le classement de popularité tiré de l'API, limité aux 50 premiers au départ, avec une seule carte par franchise (pas de doublons de saisons).
- À la fin, **aucune note n'est demandée** : seulement le badge « à classer ».
- L'onboarding est livré dès le lot 1 (il alimente *Terminés*). Le badge « à classer » apparaît avec les tier lists (lot 2).

## 6. Lot 2 : tier lists personnelles

- Un propriétaire unique par tier list.
- **Paliers fixes S, A, B, C, D** au début. Personnalisables (noms, nombre, couleurs) plus tard : ces tier lists personnalisées **ne compteront pas dans le classement principal**, car elles ne sont pas normées.
- Classement par **glisser-déposer**, avec un ordre possible à l'intérieur d'un palier.
- Seuls les animés **terminés** sont classables, ainsi que les animés abandonnés avec le marqueur « pas fini » (voir 5.2).
- Privées par défaut.

## 7. Lots suivants (esquisse)

Ordre indicatif, à détailler plus tard :

1. Google OAuth 2 et séparation des utilisateurs.
2. Partage par lien et option « publique » (une tier list publique compte dans le classement global).
3. Tier list globale (calcul à définir).
4. Fonctionnement à la Reddit : topics, abonnements, votes sur les classements, commentaires.
5. Tier list partagée à deux (animés en commun et animés de chacun).
6. Autres médias (jeux vidéo, séries) et autres thèmes (personnages, OST, openings).
7. Déploiement : Minikube, Docker, chart Helm.

## 8. Questions ouvertes

- **PWA** : installation et comportement hors ligne.
- **Tier list globale** : comment agréger les tier lists publiques en un classement unique (paliers convertis en score ? pondération ? seuil minimal de votants ?). Comment traiter les animés « pas finis » ?
- **Votes** : porte-t-on sur une tier list entière ou sur le placement d'un animé précis dans un palier ?
- **Topics à la Reddit** : qui peut en créer, comment sont-ils découvrables, comment ils sont liés à une tier list globale.
- **Modération** des commentaires et des contenus publics.
- **Exigences non fonctionnelles** : cache et protection face aux API externes, scalabilité visée, observabilité.
- **Source de données retenue** : décidée après le lot 0 (ADR).
- **Fiabilité des données de diffusion** : à vérifier dans le POC (utile pour afficher les épisodes sortis, même si « À jour » est manuel).
