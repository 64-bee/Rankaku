# Recherche : API gratuites de données et d'actualités anime

> Recherche technique préparatoire au lot 0 (voir [`docs/cadrage-produit.md`](../cadrage-produit.md)).
> Date de la recherche et des appels réels : **2026-11-13**. User-Agent des tests : `Rankaku-research/0.1`, quelques requêtes espacées par fournisseur.
> Règle de sources : uniquement des sources primaires (docs officielles, CGU, dépôts officiels, réponses réelles). Les mentions « **observé** » désignent un appel fait le 2026-11-13. Quand une information n'a pas été trouvée en source primaire, elle est marquée « **non documenté** » ou « **non vérifié** ».
> Ce document ne recommande pas de choix : il expose les faits.

## Résumé

- **AniList** (GraphQL, sans clé) renvoie en une seule requête les titres romaji, anglais et natif, les synonymes, `trending`, `popularity` et `nextAiringEpisode`. Mais l'API est en **mode dégradé à 30 req/min** (observé), et ses CGU, modifiées le 2026-08-18, **interdisent l'usage dans un service concurrent de type « anime list ou tracker »**, ainsi que le stockage et la collecte de masse.
- **MyAnimeList API v2** exige un Client ID. Son contrat **interdit de traduire ou modifier** le contenu MAL et ne publie aucun rate limit chiffré.
- **Jikan** (scraper MAL non officiel) documente 3 req/s et 60 req/min avec un cache serveur de 24 h. L'API publique était **injoignable depuis le poste de test** le 2026-11-13, alors que le dépôt est actif.
- **Kitsu** (JSON:API, sans clé) répond sur `kitsu.io` comme sur `kitsu.app`. Ses rate limits ne sont pas documentés et ses CGU n'ont pas pu être lues.
- **Shikimori** (REST + GraphQL, 5 req/s et 90 req/min) répond bien, mais il a migré vers `shikimori.io` et son dépôt est archivé. Le synopsis est en russe.
- **TMDB** est le seul candidat qui documente un paramètre de langue (`fr-FR`). En revanche : clé obligatoire, usage **non commercial** uniquement, **cache limité à 6 mois**, logo obligatoire.
- **anime-offline-database** (manami-project) est **archivé depuis juillet 2026** : les données sont figées à la release 2026-27.
- **Simkl** exige des « tracker apps » qu'elles proposent la synchronisation Simkl pour utiliser son catalogue.
- Pour les **actualités** : RSS ANN (anglais, actif ; l'édition française n'a **plus rien publié depuis le 2025-02-28**), RSS MAL News, RSS Crunchyroll (anglais et **français**, actifs mais **non documentés publiquement**) et RSS LiveChart (titres et épisodes diffusés).

### Tableau comparatif

| API | Gratuité | Auth | Format | Rate limit documenté | Anime | Manga | News | Planning de diffusion | Licence / CGU (points saillants) | Maintenance (2026-11-13) |
|---|---|---|---|---|---|---|---|---|---|---|
| AniList | Oui, non commercial (et < 150 $/mois de revenus) | Aucune en lecture (OAuth pour l'utilisateur) | GraphQL | 90/min nominal, **30/min en mode dégradé** (observé : 30) | Oui | Oui | Non | Oui (`nextAiringEpisode`, `AiringSchedule`) | Interdit : stockage, collecte de masse, **service concurrent de type tracker** | Actif ; mode dégradé en cours |
| MAL API v2 | Oui, non commercial | Client ID (`X-MAL-CLIENT-ID`) ou OAuth2 | REST/JSON | **Non documenté** (seulement « 403 DoS detected ») | Oui | Oui | Non (forum seulement) | Saison oui ; diffusion via champ `broadcast` | Interdit de traduire ou modifier le contenu ; attribution « MyAnimeList » | Non vérifié (pas de clé de test) |
| Jikan v4 | Oui | Aucune | REST/JSON | 3/s, 60/min, illimité par jour | Oui | Oui | News **par titre** | Oui (`/schedules`, `/seasons`) | Non affilié à MAL ; page CGU en 404 | Dépôt actif ; **API injoignable** pendant le test |
| Kitsu | Oui | Aucune en lecture (OAuth2 pour l'utilisateur) | JSON:API | **Non documenté** | Oui | Oui | Non | Partiel (`nextRelease`) | CGU non lisibles (SPA, 403) | Serveur actif (push 2026-06) ; doc peu maintenue (2024-08) |
| Shikimori | Oui | Aucune en lecture ; User-Agent obligatoire | REST v1/v2 + GraphQL | 5/s, 90/min | Oui | Oui | Non | Partiel (`next_episode_at`) | CGU API **non documentées** | Domaine migré `.one` → `.io` ; dépôt public archivé |
| AniDB | Oui | Client enregistré (HTTP) + compte (UDP) | XML (HTTP), UDP texte | **Non vérifié** (wiki derrière Cloudflare) | Oui | Non | Non | Non vérifié | Non vérifié | API HTTP répond (observé) |
| ANN Encyclopedia | Oui | Aucune | XML | 1 req/s par IP (retardé) ; `nodelay` : 503 au-delà | Oui | Oui | **RSS séparé** | Dates de diffusion (« Vintage ») | Attribution + lien vers la fiche ANN obligatoires | Actif |
| TMDB | Oui, non commercial | Clé API ou Bearer (compte) | REST/JSON | ~40 req/s (indicatif) | Via mot-clé « anime » | Non | Non | Oui (`next_episode_to_air`) | Logo + mention obligatoires ; **cache ≤ 6 mois** | Actif |
| AnimeSchedule v3 | Oui ; commercial sur permission | Bearer token d'application (compte) | REST/JSON | 120/min | Oui | Non | Non | **Oui (cœur du service)** | Crédit visible ; cache « au besoin » ; pas d'IA | Actif |
| Annict | Oui, commercial autorisé | Token obligatoire (un token partagé est permis sous conditions) | REST v1 + GraphQL | 600/min avec token, 60/min sans | Oui (marché japonais) | Non | Non | Oui (programmes de diffusion) | Pas de redistribution en masse ; nouvelles CGU au 2026-11-25 | Actif |
| AnimeThemes | Oui | Aucune (non mentionnée) | JSON:API + GraphQL | 90/min | Oui (OP/ED) | Non | Non | Non | Code MIT ; CGU des données non trouvées | Dépôt actif ; **503** pendant le test |
| Simkl | Oui, < 150 $/mois | `client_id` ; OAuth pour l'utilisateur | REST/JSON | 10 GET/s, quotas journaliers | Oui | Non | Non | Calendrier (CDN) | Attribution par fiche ; **un tracker doit proposer la sync Simkl** | Actif |
| anime-offline-database | Oui (jeu de données) | Aucune | Fichier JSON/JSONL | N/A (téléchargement) | Oui (métadonnées + liens croisés) | Non | Non | Non | ODbL 1.0 | **Archivé** (juillet 2026) |

---

## 1. AniList (GraphQL)

**1. Gratuité et accès**
- Gratuit pour un usage non commercial. Un usage commercial sous **150 $ de revenus par mois** est gratuit sans permission ; au-delà, il faut une licence commerciale ([terms-of-use.md](https://github.com/AniList/docs/blob/master/docs/guide/terms-of-use.md), publié sur [docs.anilist.co/guide/terms-of-use](https://docs.anilist.co/guide/terms-of-use)).
- Aucune clé pour les requêtes publiques en lecture (**observé** : requêtes `Media` et `Page` sans en-tête d'authentification, HTTP 200). OAuth pour les données utilisateur ([dossier auth](https://github.com/AniList/docs/tree/master/docs/guide/auth)).

**2. Protocole**
- GraphQL, un seul endpoint `POST https://graphql.anilist.co` (**observé**). Les docs sont dans le dépôt [AniList/docs](https://github.com/AniList/docs). La page `docs.anilist.co` renvoie un 403 aux clients automatisés, d'où la lecture des sources Markdown sur GitHub.

**3. Rate limits**
- Nominal : **90 req/min**. Un avertissement en tête de page indique : « The API is currently in a degraded state and is limited to **30 requests per minute** » ([rate-limiting.md](https://github.com/AniList/docs/blob/master/docs/guide/rate-limiting.md), dernière modification le 2025-08-16).
- **Observé** : `X-RateLimit-Limit: 30`, `X-RateLimit-Remaining: 29`. Le mode dégradé est donc toujours actif.
- En cas de dépassement : HTTP 429, **une minute de blocage**, en-têtes `Retry-After` (secondes) et `X-RateLimit-Reset` (timestamp Unix), corps GraphQL `"Too Many Requests."`. Il existe aussi un *burst limiter* dont les chiffres ne sont pas publiés. Les demandes d'augmentation de limite sont actuellement refusées ([rate-limiting.md](https://github.com/AniList/docs/blob/master/docs/guide/rate-limiting.md)).
- En cas de panne sévère : les limites peuvent baisser ou l'API être suspendue, avec un **HTTP 403** et un message GraphQL. Le blocage manuel d'IP est possible ([considerations.md](https://github.com/AniList/docs/blob/master/docs/guide/considerations.md)).
- Cache : aucune durée recommandée ; l'en-tête observé est `Cache-Control: no-cache, private`.

**4. Structure des réponses** (**observé**, `Media(id: 5114)`, tronqué)
```json
{
  "data": { "Media": {
    "id": 5114, "idMal": 5114,
    "title": { "romaji": "Hagane no Renkinjutsushi: FULLMETAL ALCHEMIST",
               "english": "Fullmetal Alchemist: Brotherhood",
               "native": "鋼の錬金術師 FULLMETAL ALCHEMIST" },
    "synonyms": ["FMA", "FMAB", "Hagaren", "Full Metal Alchemist: Brotherhood", "Стальной алхимик: Братство", "..."],
    "episodes": 64, "status": "FINISHED",
    "startDate": { "year": 2009, "month": 4, "day": 5 },
    "season": "SPRING", "seasonYear": 2009,
    "genres": ["Action", "Adventure", "Drama", "Fantasy"],
    "studios": { "nodes": [ { "name": "bones" } ] },
    "coverImage": { "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx5114-....jpg" },
    "averageScore": 90, "popularity": 744208, "trending": 33,
    "nextAiringEpisode": null
  } }
}
```
- Pagination : on enveloppe la requête dans `Page`, qui contient `pageInfo` (`currentPage`, `hasNextPage`, `perPage`). Un seul champ de données par `Page`. **`total` et `lastPage` ne sont pas fiables** : seul `hasNextPage` doit être utilisé ([pagination.md](https://github.com/AniList/docs/blob/master/docs/guide/graphql/pagination.md)).
- **Observé** : `Page(perPage: 2) { media(sort: TRENDING_DESC) { nextAiringEpisode { episode airingAt timeUntilAiring } } }` renvoie par exemple `"episode": 8, "airingAt": 1795183200`.
- Sur l'échantillon testé (FMA:B), les synonymes ne contiennent **aucun titre français**.

**5. Couverture**
- Anime, manga, personnages, staff, studios, saisons, tendances, planning (`nextAiringEpisode`, `AiringSchedule`), recommandations, listes utilisateur (`MediaListCollection`). Les enums de la référence en témoignent ([docs/reference](https://github.com/AniList/docs/tree/master/docs/reference/enum) : `airingsort`, `mediatrendsort`, `recommendationsort`, `medialiststatus`…). Pas de news.

**6. Licence et CGU** ([terms-of-use.md](https://github.com/AniList/docs/blob/master/docs/guide/terms-of-use.md), modifié le **2026-08-18**, commit « Update wording of competing services terms ») :
- Clause 2 : « Using the AniList API as a **backup or data storage service** is strictly prohibited. »
- Clause 3 : « **Hoarding** or mass collection of data […] is strictly prohibited. » Tolérance pour les projets « purely educational ».
- Clause 5 : « Use of the AniList API within **competing, non-complementary services of the same nature is prohibited**. This includes, but is not limited to, **anime and manga list or tracker services**. The restriction applies to all data provided through the API, including both user data and media data. »
- Règles de nommage : pas de « AniList » seul dans le nom de l'application.
- Contenu adulte : le genre « Ecchi » n'est pas considéré comme adulte ; le filtrage n'est pas garanti ([considerations.md](https://github.com/AniList/docs/blob/master/docs/guide/considerations.md)).

**7. Fiabilité**
- Dépôt docs mis à jour le 2026-08-18. Mode dégradé annoncé depuis au moins 2024-09 (historique des commits de `rate-limiting.md`) et toujours actif (observé). Les pannes sont annoncées sur Discord ([considerations.md](https://github.com/AniList/docs/blob/master/docs/guide/considerations.md)).

---

## 2. MyAnimeList API officielle v2

**1. Gratuité et accès**
- Il faut un Client ID obtenu par inscription. Il est révocable et ne doit pas être partagé ([API License and Developer Agreement](https://myanimelist.net/static/apiagreement.html), § 2(a), dernière modification le 8 août 2019).
- Applications non commerciales autorisées, y compris avec de la publicité non intrusive. Pour un usage commercial, il faut une autorisation écrite (§ 1(h), 1(i), 3(a)(xiv)).
- Authentification : l'en-tête `X-MAL-CLIENT-ID` suffit pour les données publiques. L'OAuth2 (scope `write:users`) sert aux listes utilisateur ([référence API v2](https://myanimelist.net/apiconfig/references/api/v2), section *Authentication*).
- **Observé** : un appel sans Client ID renvoie `403 {"message":"","error":"forbidden"}`.

**2. Protocole**
- REST/JSON, URL de base `https://api.myanimelist.net/v2`. La version n'est incrémentée qu'en cas de changement incompatible ([référence](https://myanimelist.net/apiconfig/references/api/v2), section *Versioning*). La doc s'intitule toujours « MyAnimeList API (beta ver.) ».

**3. Rate limits**
- **Non documenté** en chiffres. La table des codes indique seulement « 403 Forbidden – DoS detected etc. » ([référence](https://myanimelist.net/apiconfig/references/api/v2)).
- Le contrat réserve le droit de limiter l'usage à tout moment et interdit de contourner les limites. Il prévoit aussi la possibilité de rendre l'API payante après préavis ([accord](https://myanimelist.net/static/apiagreement.html), § 6).

**4. Structure des réponses** (d'après la doc ; pas d'exemple de réponse publié, la doc affiche « No sample »)
- Les champs sont choisis via `fields` : `id, title, main_picture, alternative_titles, start_date, end_date, synopsis, mean, rank, popularity, num_list_users, num_scoring_users, nsfw, media_type, status, genres, num_episodes, start_season, broadcast, source, average_episode_duration, rating, pictures, background, related_anime, related_manga, recommendations, studios, statistics` ([référence, *Get anime details*](https://myanimelist.net/apiconfig/references/api/v2)).
- Pagination : `{"data": [...], "paging": {"previous": "...", "next": "..."}}` avec `limit` et `offset`. Limite max : 100 pour la recherche, 500 pour le classement et la saison, 1000 pour la liste utilisateur ([référence](https://myanimelist.net/apiconfig/references/api/v2)).

**5. Couverture**
- Anime et manga (recherche, détails, classement `ranking_type` dont `airing`, `upcoming` et `bypopularity`), saison (`/anime/season/{year}/{season}`), suggestions (utilisateur authentifié), listes utilisateur en lecture et écriture, forum. Pas de personnages ni de staff en endpoints dédiés, pas de news ([référence](https://myanimelist.net/apiconfig/references/api/v2)).

**6. Licence et CGU** ([accord](https://myanimelist.net/static/apiagreement.html))
- § 3(a)(xii) : interdiction de « delete, **translate**, edit, modify or otherwise alter in any way any MyAnimeList Content » sans autorisation écrite.
- § 3(a)(ix) : interdiction d'« imitate or attempt to impersonate the user experience of the Company Offering ».
- § 3(a)(xiii) : la marque ne peut être utilisée que pour attribuer la source (exemple donné : « authorized by MyAnimeList »).
- § 3 (sécurité) : interdiction de stocker côté serveur les **données personnelles des utilisateurs MAL et le contenu qu'ils génèrent** (posts de forum…). Cette restriction ne vise pas les métadonnées d'anime.

**7. Fiabilité**
- Non vérifié par un appel authentifié (pas de Client ID pour la recherche).

---

## 3. Jikan v4 (API non officielle de MyAnimeList)

**1. Gratuité et accès**
- Gratuit, sans clé, lecture seule (GET uniquement). Jikan scrape MyAnimeList et n'est pas affilié à MAL ([spécification OpenAPI, champ `info.description`](https://github.com/jikan-me/jikan-rest/blob/master/storage/api-docs/api-docs.json) ; doc rendue sur [docs.api.jikan.moe](https://docs.api.jikan.moe/)).
- La doc renvoie vers des « terms of use » à `https://jikan.moe/terms`, mais cette URL renvoie une **page 404** (**observé**).

**2. Protocole**
- REST/JSON, URL de base `https://api.jikan.moe/v4` (champ `servers` de l'OpenAPI). Le serveur est open source (MIT) et auto-hébergeable ([jikan-me/jikan-rest](https://github.com/jikan-me/jikan-rest)).

**3. Rate limits** ([OpenAPI](https://github.com/jikan-me/jikan-rest/blob/master/storage/api-docs/api-docs.json))
- **3 req/s, 60 req/min, illimité par jour.** « It's still possible to get rate limited from MyAnimeList.net instead. »
- HTTP 429 `RateLimitException` (que la limite vienne de Jikan ou de MAL) ; 503 en maintenance.
- Cache serveur : **toutes les réponses sont cachées 24 h**, avec les en-têtes `Expires`, `Last-Modified` et `X-Request-Fingerprint`. Un `ETag` est renvoyé et `If-None-Match` donne un 304.

**4. Structure des réponses**
- **Non observée** : l'API était injoignable depuis le poste de test (voir point 7). Champs du schéma `anime` tirés de l'OpenAPI : `mal_id, url, images, trailer, approved, titles, title, title_english, title_japanese, title_synonyms, type, source, episodes, status, airing, aired, duration, rating, score, scored_by, rank, popularity, members, favorites, synopsis, background, season, year, broadcast, producers, licensors, studios, genres, explicit_genres, themes, demographics`. Les réponses sont enveloppées dans `{"data": ...}`.
- Pagination : `{"pagination": {"last_visible_page": int, "has_next_page": bool}}`, avec une variante `pagination_plus` (schémas `pagination` et `pagination_plus` de l'OpenAPI).

**5. Couverture** (chemins de l'OpenAPI)
- `anime`, `manga`, `characters`, `people`, `producers`, `magazines`, `genres`, `clubs`, `users` (profils et listes publics), `seasons` (`/seasons/now`, `/seasons/upcoming`, `/seasons/{year}/{season}`), `schedules`, `top`, `recommendations`, `reviews`, `random`, `watch`.
- News : **uniquement par titre** (`/anime/{id}/news`, `/manga/{id}/news`). Pas de flux global.

**6. Licence et CGU**
- Code serveur sous licence MIT ([dépôt](https://github.com/jikan-me/jikan-rest)). Les données viennent de MAL ; Jikan déclare n'être pas affilié. Page de CGU introuvable (404 observé).

**7. Fiabilité**
- Dépôt `jikan-me/jikan-rest` : dernier push le 2026-11-12, dernière release taguée v4.2.4 le 2024-11-09. La migration de la bibliothèque sous-jacente vers v5 est en cours ([issue #619](https://github.com/jikan-me/jikan-rest/issues/619)).
- **Observé** : `api.jikan.moe` est **injoignable** (timeout TCP vers 135.181.39.91 et l'IPv6, environ cinq essais entre 20:11 et 20:20 UTC, WebFetch en échec également), alors que `jikan.moe` répond 200. La cause n'est pas déterminée (panne ponctuelle ou filtrage réseau local).

---

## 4. Kitsu (JSON:API)

**1. Gratuité et accès**
- Gratuit. « Authentication is not required for most public-facing `GET` endpoints » ; OAuth2 pour les données utilisateur. Le contenu NSFW/R18 est masqué sans authentification ([kitsu.yml](https://github.com/hummingbird-me/api-docs/blob/openapi3/api/kitsu.yml)).

**2. Protocole**
- JSON:API. Version `edge`. La doc indique `https://kitsu.io/api/edge`. **Observé** : la même requête répond en 200 sur `kitsu.io` et sur `kitsu.app`. Les liens de pagination renvoyés via `kitsu.app` pointent vers `kitsu.app`, et les images sont servies par `media.kitsu.app`.
- La recherche avancée passe par Algolia (clés obtenues via l'API) ([kitsu.yml](https://github.com/hummingbird-me/api-docs/blob/openapi3/api/kitsu.yml)).

**3. Rate limits**
- **Non documenté.** **Observé** : aucun en-tête de rate limit ; `Cache-Control: max-age=0, private, must-revalidate`.

**4. Structure des réponses** (**observé**, `GET /api/edge/anime?filter[text]=fullmetal&page[limit]=1`, tronqué)
```json
{
  "data": [ { "id": "3936", "type": "anime",
    "attributes": {
      "canonicalTitle": "Fullmetal Alchemist: Brotherhood",
      "titles": { "en": "Fullmetal Alchemist: Brotherhood", "en_jp": "Hagane no Renkinjutsushi: Fullmetal Alchemist", "ja_jp": "鋼の錬金術師 FULLMETAL ALCHEMIST" },
      "synopsis": "...", "episodeCount": 64, "status": "finished", "subtype": "TV",
      "startDate": "2009-04-05", "endDate": "2010-07-04", "nextRelease": null,
      "averageRating": "83.06", "popularityRank": 7, "ageRating": "R",
      "posterImage": { "tiny": "...", "small": "...", "medium": "...", "large": "https://media.kitsu.app/anime/poster_images/3936/large.jpg", "original": "..." }
    },
    "relationships": { "genres": {}, "categories": {}, "mediaRelationships": {}, "characters": {}, "staff": {}, "productions": {}, "episodes": {}, "streamingLinks": {}, "mappings": {} }
  } ],
  "meta": { "count": 14 },
  "links": { "first": "...page%5Boffset%5D=0", "next": "...page%5Boffset%5D=1", "last": "...page%5Boffset%5D=13" }
}
```
- Pagination : `page[limit]` (10 par défaut, **20 maximum**) et `page[offset]`, avec des liens `first`, `next` et `last` ([kitsu.yml](https://github.com/hummingbird-me/api-docs/blob/openapi3/api/kitsu.yml), [page.yml](https://github.com/hummingbird-me/api-docs/blob/openapi3/api/parameters/page.yml)).
- Sur l'échantillon testé, `titles` ne contient **pas de titre français**. Les studios et les genres demandent `include=`.

**5. Couverture**
- Anime, manga, épisodes, personnages, staff, productions (studios), relations entre médias, mappings vers d'autres bases, liens de streaming, données utilisateur (OAuth). `nextRelease` est le seul indice de planning. `GET /api/edge/trending/anime` répond 200 (**observé**), mais la route est **commentée dans la doc** (`#- trending`, [kitsu.yml](https://github.com/hummingbird-me/api-docs/blob/openapi3/api/kitsu.yml)) : elle est donc non documentée. Pas de news.

**6. Licence et CGU**
- La doc pointe vers `https://kitsu.io/terms`. `kitsu.app/terms` est une application monopage dont le texte n'est pas extractible, et WebFetch reçoit un 403. **CGU non vérifiées.** La doc elle-même est sous Apache 2.0.

**7. Fiabilité**
- `hummingbird-me/kitsu-server` : dernier push le 2026-06-20 (licence Apache-2.0). `hummingbird-me/api-docs` : dernier push le 2024-08-17, conversion OpenAPI 3 « en cours » ([README](https://github.com/hummingbird-me/api-docs)).

---

## 5. Shikimori

**1. Gratuité et accès**
- Gratuit, lecture publique sans token (**observé**). OAuth2 pour l'authentification. Il est obligatoire de mettre le nom de son application OAuth2 dans le `User-Agent` : « Don't mimic a browser. Your IP address may be banned if you use API without properly set `User-Agent` header » ([doc API](https://shikimori.io/api/doc)).

**2. Protocole**
- Trois versions : **GraphQL** (recommandée), v2 et v1 (« outdated »). « New anime/manga/character/person posters available only in `graphql` API » ([doc API](https://shikimori.io/api/doc)). HTTPS uniquement.
- **Observé** : `https://shikimori.one/api/...` redirige en **301** vers `https://shikimori.io/api/...`. Endpoint GraphQL : `POST https://shikimori.io/api/graphql`.

**3. Rate limits**
- « API access is limited by **5rps** and **90rpm** » ([doc API](https://shikimori.io/api/doc)). Le comportement en cas de dépassement n'est **pas documenté**. **Observé** : aucun en-tête de rate limit.
- « Please do not parse the main site » ([doc API](https://shikimori.io/api/doc)).

**4. Structure des réponses** (**observé**)
- REST `GET /api/animes/5114` (tronqué) :
```json
{ "id": 5114, "name": "Fullmetal Alchemist: Brotherhood", "russian": "Стальной алхимик: Братство",
  "english": ["Fullmetal Alchemist: Brotherhood"], "japanese": ["鋼の錬金術師 FULLMETAL ALCHEMIST"],
  "kind": "tv", "score": "9.11", "status": "released", "episodes": 64, "episodes_aired": 0,
  "aired_on": "2009-04-05", "released_on": "2010-07-04", "next_episode_at": null,
  "myanimelist_id": 5114, "franchise": "fullmetal_alchemist",
  "image": { "original": "/system/animes/original/5114.jpg?1711949773", "preview": "..." },
  "genres": [ { "id": 27, "name": "Shounen", "russian": "Сёнен", "kind": "genre" } ],
  "studios": [ { "id": 4, "name": "Bones" } ],
  "description": "Ремейк одноимённого аниме-сериала «[anime=121]Стальной алхимик[/anime]» 2003 года…" }
```
- GraphQL `animes(search: "Frieren", limit: 1)` renvoie `id`, `malId`, `name`, `english`, `japanese`, `kind`, `status` (`anons`), `episodes`, `episodesAired`, `airedOn.date` (`2027-10-01`), `nextEpisodeAt`, `score`, `poster.originalUrl`, `genres`, `studios`.
- Sur les 2 échantillons testés, l'identifiant Shikimori **est égal à l'identifiant MAL** (`id` = `myanimelist_id` = 5114 ; `id` = `malId` = 63816). Le synopsis est **en russe**, avec un balisage BBCode maison (`[anime=121]…[/anime]`).
- Pagination : le paramètre `limit` a été utilisé dans la requête GraphQL ; le reste du mécanisme (page, curseur) n'a pas été vérifié dans la doc.

**5. Couverture**
- Anime, manga, ranobe, personnages, personnes, studios, genres, franchise, `next_episode_at`, notes et statistiques de listes, listes utilisateur (OAuth), vidéos et captures d'écran. Pas de news exploitable pour un public francophone.

**6. Licence et CGU**
- **Non documentées** dans la doc API (seulement les restrictions de débit et de User-Agent ci-dessus).

**7. Fiabilité**
- Le dépôt public [`shikimori/shikimori`](https://github.com/shikimori/shikimori) est **archivé** (dernier push le 2025-02-02, licence OSL-3.0). Le service en production répond néanmoins (**observé**), sur un nouveau domaine.

---

## 6. AniDB (HTTP API + UDP API)

**1. Gratuité et accès**
- **Observé** : `GET http://api.anidb.net:9001/httpapi?request=hotanime&client=…&clientver=1&protover=1` avec un client non enregistré renvoie `<error code="302">client version missing or invalid</error>`. Un **client enregistré** est donc obligatoire.
- La doc officielle ([HTTP API Definition](https://wiki.anidb.net/HTTP_API_Definition), [UDP API Definition](https://wiki.anidb.net/UDP_API_Definition)) est derrière un **challenge Cloudflare** : curl et WebFetch ont reçu un 403 (`Cf-Mitigated: challenge`). **Elle n'a pas pu être lue en source primaire.**

**2 à 7.** Format XML (HTTP, observé) ; rate limits, règles de cache, CGU, structure complète et couverture **non vérifiés en source primaire**. Voir « Points ouverts ». Le fichier de titres `anime-titles.xml.gz` est lui aussi derrière le challenge (403 observé).

---

## 7. Anime News Network : Encyclopedia API

**1. Gratuité et accès**
- « We provide this API free of charge but you must list Anime News Network as the source of the data [and] include a link to the relevant Encyclopedia entry (like "full details at Anime News Network") on any page that displays anime/manga/person details » ([api.php](https://www.animenewsnetwork.com/encyclopedia/api.php)). Pas de clé.

**2. Protocole**
- XML. Deux API ([api.php](https://www.animenewsnetwork.com/encyclopedia/api.php)) :
  - `reports.xml?id=155&type=anime&nlist=all` : liste de base des titres (import initial, puis import quotidien avec `nlist=50`) ;
  - `https://cdn.animenewsnetwork.com/encyclopedia/api.xml?anime={id}` (ou `manga=`, `title=`, `title=~nom` pour une recherche) : jusqu'à 50 titres par lot.

**3. Rate limits**
- « rate-limited to **1 request per second per IP** address; requests over this threshold will be **delayed**. » La variante `nodelay.api.xml` accepte 5 requêtes groupées toutes les 5 s, mais renvoie un **503** au-delà de 1/s ([api.php](https://www.animenewsnetwork.com/encyclopedia/api.php)).
- La doc demande de mettre en cache : « please try to spare our poor servers by fetching (and then caching) details from this API only as needed ». **Observé** : `Cache-Control: public`, aucun en-tête de rate limit.

**4. Structure des réponses** (**observé**, `api.xml?anime=10216`, tronqué)
```xml
<ann><anime id="10216" type="TV" name="Fullmetal Alchemist: Brotherhood" generated-on="2026-11-13T20:11:49Z">
  <related-prev rel="alternate retelling of" id="2960"/>
  <info type="Picture" src="https://cdn.animenewsnetwork.com/thumbnails/fit200x200/encyc/A10216-220.jpg">…</info>
  <info type="Main title" lang="EN">Fullmetal Alchemist: Brotherhood</info>
  <info type="Alternative title" lang="JA">鋼の錬金術師 FULLMETAL ALCHEMIST</info>
  <info type="Genres">adventure</info> …
  <info type="Plot Summary">Two brothers lose their mother to an incurable disease. …</info>
  <info type="Number of episodes">64</info>
  <info type="Vintage">2009-04-05 to 2010-07-04</info>
  <info type="Vintage">2009-10-19 (France, Virgin 17)</info> …
  <ratings nb_votes="6178" weighted_score="9.0752" bayesian_score="9.0721"/>
  <!-- présents aussi dans ce fichier : info Opening/Ending Theme, Themes, Running time, Official website ;
       64 éléments <episode>, 98 <staff>, 198 <cast>, 6 <credit> -->
</anime></ann>
```
- Les identifiants ANN sont propres à ANN (10216 ≠ 5114 côté MAL). Les dates de diffusion sont en **texte libre** dans `Vintage`, y compris par pays (on y trouve des diffusions françaises). Pas de pagination sur `api.xml` (lots de 50) ; `nlist` sur `reports.xml`.

**5. Couverture**
- Anime et manga, avec épisodes, staff, cast, crédits (`<credit>`, 6 sur l'échantillon) et génériques (**observé** sur `anime=10216`). Pas de popularité ni de tendances via cette API ; un score de votes est présent. Les news passent par des flux RSS séparés (voir « Sources de news »).

**6. Licence et CGU**
- Attribution et lien vers la fiche obligatoires (citation ci-dessus). Aucune autre restriction de stockage n'est mentionnée sur la page.

**7. Fiabilité**
- Réponse générée à la volée (`generated-on` du jour, **observé**). Pas de changelog public trouvé.

---

## 8. TMDB (catégorie animation)

**1. Gratuité et accès**
- « Our API is free to use for non-commercial purposes as long as you attribute TMDB as the source of the data and/or images » ([FAQ](https://developer.themoviedb.org/docs/faq)). L'usage commercial passe par un accord écrit séparé ([API Terms of Use](https://www.themoviedb.org/api-terms-of-use), § 1.A et 2.A, mis à jour le 20 octobre 2023).
- Il faut un compte TMDB pour obtenir une clé (`api_key` en query) ou un Bearer token (« API Read Access Token »). Aucune étape d'approbation n'est décrite ([authentication-application](https://developer.themoviedb.org/docs/authentication-application)). **Non testé** (pas de clé).

**2. Protocole**
- REST/JSON, v3 (`/3/...`). `append_to_response` permet de combiner jusqu'à 20 sous-ressources ([tv-series-details](https://developer.themoviedb.org/reference/tv-series-details)).
- Langue : paramètre `language` au format ISO 639-1 + ISO 3166-1 (par exemple `fr-FR`). « Most metadata endpoints return translated data », sauf les noms de personnes et de personnages ([languages](https://developer.themoviedb.org/docs/languages)). Le comportement de repli quand la traduction manque n'est **pas documenté** sur cette page.

**3. Rate limits**
- La limite historique (40 req / 10 s) a été désactivée le 16 décembre 2019. Les limites actuelles se situent « somewhere in the **40 requests per second** range » et peuvent changer ; il faut respecter les **429** ([rate-limiting](https://developer.themoviedb.org/docs/rate-limiting)).

**4. Structure des réponses** (d'après le schéma de la doc, `GET /3/tv/{id}`)
- `id, name, original_name, overview, first_air_date, last_air_date, status, type, in_production, number_of_episodes, number_of_seasons, seasons[], episode_run_time[], genres[{id,name}], networks[], production_companies[], origin_country[], original_language, poster_path, backdrop_path, vote_average, vote_count, popularity, last_episode_to_air, next_episode_to_air` ([tv-series-details](https://developer.themoviedb.org/reference/tv-series-details)).
- Pagination (discover et search) : `{"page", "results", "total_pages", "total_results"}` ([discover-tv](https://developer.themoviedb.org/reference/discover-tv)).
- Modèle différent des bases anime : une série TMDB regroupe ses **saisons** (`seasons[]`), alors que AniList, MAL et Kitsu traitent chaque saison comme une entrée distincte. Simkl le confirme : « all six *Attack on Titan* cour titles share TVDB `267440` and TMDB `1429` » ([Simkl, guide anime](https://api.simkl.org/guides/anime)).

**5. Couverture**
- Films et séries TV. Pas de catégorie « anime » native : on filtre via `/3/discover/tv` (`with_keywords`, `with_genres`, `with_origin_country`, `with_original_language`, `sort_by=popularity.desc`) ([discover-tv](https://developer.themoviedb.org/reference/discover-tv)). Le mot-clé « **anime** » a l'ID **210024** et **4 476 séries** l'utilisent ([page mot-clé](https://www.themoviedb.org/keyword/210024-anime/tv), observé via WebFetch). L'ID du genre « Animation » n'a **pas été vérifié** en source primaire.
- Planning : `next_episode_to_air`. Popularité : `popularity`. Pas de manga ni de news.

**6. Licence et CGU** ([API Terms of Use](https://www.themoviedb.org/api-terms-of-use))
- § 3 : logo TMDB obligatoire, plus la mention « This [website, program, service, application, product] uses TMDB and the TMDB APIs but is not endorsed, certified, or otherwise approved by TMDB ».
- § 1.C : interdiction de « Cache, **for longer than 6 months**, any information obtained through or from TMDB », de créer des œuvres dérivées et d'utiliser TMDB comme hébergeur d'images.
- § 1.D : purge de tout le contenu en cache à la résiliation.

**7. Fiabilité**
- Non mesurée (pas de clé). Doc officielle à jour sur `developer.themoviedb.org`.

---

## 9. AnimeSchedule.net API v3

**1. Gratuité et accès**
- Il faut un compte AnimeSchedule et créer une « Application » pour obtenir un **Bearer token**, à envoyer à chaque requête. Les endpoints publics sans token sont « for internal use only » et ont un débit « far less forgiving » ([documentation](https://animeschedule.net/api/v3/documentation)). **Observé** : `GET /api/v3/timetables/sub` sans token → `401 Unauthorized`.
- Usage commercial sur permission ([documentation](https://animeschedule.net/api/v3/documentation), résumé des CGU).

**2. Protocole**
- REST/JSON, base `https://animeschedule.net/api/v3`. Champs en camelCase ; dates ISO 8601 UTC ; la valeur nulle des dates est `0001-01-01T00:00:00Z` ([documentation](https://animeschedule.net/api/v3/documentation)).

**3. Rate limits**
- « The global rate limit is set to **120 requests per minute** », modifiable à tout moment. En-têtes `X-RateLimit-Limit`, `X-RateLimit-Remaining` et `X-RateLimit-Reset` (timestamp Unix). Le code HTTP en cas de dépassement n'est **pas documenté** ([ratelimits](https://animeschedule.net/api/v3/documentation/ratelimits)).

**4. Structure des réponses** (d'après la doc, [anime](https://animeschedule.net/api/v3/documentation/anime))
- Objet Anime : `id, title, route` (slug), `names {romaji, english, native, abbreviation, synonyms}`, `description` (HTML), `episodes, lengthMin, status, premier, subPremier, dubPremier, jpnTime, subTime, dubTime, season, year, genres[], studios[], sources[], mediaTypes[], imageVersionRoute, stats {averageScore, ratingCount, trackedCount, trackedRating…}, websites {official, mal, aniList, kitsu, animePlanet, anidb, streams}, relations`.
- Timetable (`GET /timetables/{raw|sub|dub|all}?year=&week=&tz=`) : `episodeDate, episodeNumber, airType, airingStatus (airing, aired, unaired, delayed-air), title, route, status, episodes, streams…`. Fuseau par défaut : `Europe/London`.
- Pagination de `GET /anime` : `page`, 18 résultats par page.

**5. Couverture**
- Anime et planning de diffusion (raw, sub et dub) à la semaine, saisons, catégories (genres, studios). Listes utilisateur (OAuth). Pas de manga ni de news.

**6. Licence et CGU** (résumé figurant dans la [documentation](https://animeschedule.net/api/v3/documentation))
- « Cache API data only as needed and do not use it to train AI » ; « Ask us for permission before using the API for commercial purposes » ; « Include a visible link or credit to AnimeSchedule.net in your app ». Le texte complet des CGU n'a pas été lu.

**7. Fiabilité**
- La doc expose un onglet *Changelog* (contenu non extrait). Le service répond (401 observé).

---

## 10. Annict

**1. Gratuité et accès**
- « Annict APIを使うには、認証を行ってアクセストークンを用意する必要があります » : il faut un token (token personnel ou OAuth) ([Wikino, page 356](https://wikino.app/s/annict/pages/356)). **Observé** : `GET https://api.annict.com/v1/works` sans token → `401` avec `ratelimit-limit: 60`.
- Usage commercial autorisé sans permission individuelle, sauf pour l'IA commerciale et pour fournir les données comme dataset ([API Terms, art. 3](https://annict.com/en/terms/api)).

**2. Protocole**
- REST v1 et GraphQL. La doc a migré de `developers.annict.com` (qui renvoie désormais un 404 de redirection, [dépôt annict-developers](https://github.com/annict/annict-developers)) vers [Wikino, sujet « Developer Help »](https://wikino.app/s/annict/topics/5).

**3. Rate limits** ([Wikino, page 419](https://wikino.app/s/annict/pages/419))
- **600 req/min** avec token (par couple token et IP), **60 req/min** sans token (par IP). Fenêtre fixe à la minute. Une requête GraphQL compte pour une.
- Dépassement : **429** + `Retry-After` ; en-têtes `RateLimit-Limit`, `RateLimit-Remaining` et `RateLimit-Reset` (compte à rebours en secondes) ; corps `rate_limit_exceeded`.

**4. Structure des réponses**
- **Non observée** (token requis). Les pages de référence REST (`works`, `episodes`, `programs`, `casts`, `staffs`, `characters`, `series`…) sont listées sur [Wikino](https://wikino.app/s/annict/topics/5) mais n'ont pas été extraites.

**5. Couverture**
- Œuvres, épisodes, séries, personnages, personnes, organisations, staff, cast, programmes de diffusion, avis, statuts et activités utilisateur ; identifiants Shobo Calendar ([Wikino](https://wikino.app/s/annict/topics/5)). Service japonais : la langue des données n'a pas été vérifiée (pas de token).

**6. Licence et CGU** ([API Terms, version japonaise qui fait foi](https://annict.com/terms/api) ; [traduction anglaise](https://annict.com/en/terms/api) ; établies le 2026-10-25, **en vigueur au 2026-11-25**)
- Art. 5 (texte japonais lu directement) : une application qui ne demande pas de compte Annict à ses utilisateurs peut embarquer **un seul** token personnel du développeur et l'utiliser pour toutes les requêtes, à trois conditions : ne récupérer que des informations sur les œuvres (pas sur les utilisateurs), utiliser un token en lecture seule (`read`), et prévoir de pouvoir remplacer le token, que l'opérateur peut révoquer sans préavis en cas d'abus.
- Art. 6 : interdiction de fournir les données telles quelles ou en masse sous une forme extractible (URL publique, dépôt public).
- Art. 7 : stockage et cache autorisés « to the extent necessary to provide the application », côté serveur ou client.
- Art. 13 : attribution facultative ; interdiction de laisser croire à une application officielle.

**7. Fiabilité**
- [`annict/annict`](https://github.com/annict/annict) : dernier push le 2026-11-13. Le service est actif.

---

## 11. AnimeThemes

**1. Gratuité et accès**
- Aucune authentification mentionnée ([api-docs.animethemes.moe](https://api-docs.animethemes.moe/)). Code sous licence MIT ([dépôt](https://github.com/AnimeThemes/animethemes-server)).

**2. Protocole**
- JSON:API (`https://api.animethemes.moe`) et GraphQL (`graphql.animethemes.moe`) ([api-docs](https://api-docs.animethemes.moe/)).

**3. Rate limits**
- « The AnimeThemes API limits requests to **90 per minute** ». En-têtes `X-RateLimit-Limit` et `X-RateLimit-Remaining` ; en cas de dépassement, `Retry-After` et `X-RateLimit-Reset`, corps « Too Many Attempts. » ([ratelimiting](https://api-docs.animethemes.moe/intro/ratelimiting/)).

**4. Structure des réponses**
- **Non observée** : deux appels ont reçu **503** derrière Cloudflare le 2026-11-13. C'est une observation ponctuelle, pas la preuve d'un arrêt (le dépôt a reçu un push le 2026-11-08).

**5. Couverture**
- Openings et endings (thèmes, vidéos, artistes, chansons), mappings vers MAL et AniList ([api-docs](https://api-docs.animethemes.moe/)). Utile pour les futures tier lists d'openings (cadrage, § 7) ; pas un catalogue généraliste.

**6. Licence et CGU**
- Pas de CGU des données trouvées sur la doc.

**7. Fiabilité**
- [`AnimeThemes/animethemes-server`](https://github.com/AnimeThemes/animethemes-server) : dernier push le 2026-11-08.

---

## 12. Simkl

**1. Gratuité et accès**
- Gratuit pour les services de moins de **150 $ de revenus par mois** ; au-delà, licence commerciale ([API rules, § 5](https://api.simkl.org/api-rules)).
- `client_id` obligatoire (création d'une app), paramètres `app-name` et `app-version`, OAuth 2.0 ou PIN pour les utilisateurs ([api.simkl.org](https://api.simkl.org/)). **Observé** : un appel sans `client_id` → `412 client_id_failed`.

**2. Protocole**
- REST/JSON, base `https://api.simkl.com` ([api.simkl.org](https://api.simkl.org/)).

**3. Rate limits** ([rate-limits](https://api.simkl.org/resources/rate-limits))
- **10 GET/s, 1 POST/s**. Quotas journaliers par utilisateur (AUTH V2) : 500 (Free), 1 000 (PRO), 10 000 (VIP), remis à zéro à minuit heure de l'Est américaine.
- 429 (`rate_limit`, `user_limit_exceeded`, `app_limit_exceeded`). En cas de dépassement prolongé : « client_id suspended… No warning, no appeal ».
- Les détails, les listes d'épisodes, les fichiers *trending* et *calendar* sont **exemptés** du quota.
- Cache conseillé : « minutes for user data, hours for catalogs, forever for images » ([API rules, § 7](https://api.simkl.org/api-rules)).

**4. Structure des réponses**
- **Non observée** (pas de `client_id`). Champs documentés pour l'anime : `anime_type`, `title_romaji`, `title_en`, `en_title`, `alt_titles[]`, `season_name_year`, `studios`, `relations[]` (`relation_type`, `is_direct`), `ids {simkl, mal, anilist, anidb, kitsu, tmdb, tvdb}` ([guide anime](https://api.simkl.org/guides/anime)).

**5. Couverture**
- Films, séries et anime ; recherche (y compris par ID MAL ou AniDB), *trending* et *calendar* via CDN sans authentification, synchronisation des listes utilisateur ([api.simkl.org](https://api.simkl.org/)).

**6. Licence et CGU** ([API rules](https://api.simkl.org/api-rules))
- Attribution : lien vers la fiche Simkl de chaque élément ; une rubrique « trending » doit nommer Simkl.
- Règle 2 « Tracker apps must offer Simkl sync » (vérifiée directement) : « If your app or service is a Movie / TV / Anime / Manga tracker that already syncs with another tracker, you may use the Simkl API only if you also offer Simkl login and sync alongside. In other words: the catalog and discovery endpoints are not a free metadata source for a competing tracker that doesn't integrate Simkl. […] If you're using Simkl's catalog without offering Simkl sync, contact us first. »
- Règle 3 : « Simkl is built for tracking and discovery, not as a CDN for the world's metadata. »

**7. Fiabilité**
- [`SIMKL/API`](https://github.com/SIMKL/API) : dernier push le 2026-10-14. La doc `api.simkl.org` est à jour.

---

## 13. anime-offline-database (manami-project)

**1. Gratuité et accès**
- Jeu de données téléchargeable (JSON minifié, JSONL, `.zst`) dans les releases GitHub, sans authentification ([README](https://github.com/manami-project/anime-offline-database)).

**2. Protocole**
- Fichiers statiques. Schéma JSON fourni (`anime-offline-database-minified.schema.json`).

**3. Rate limits**
- Sans objet (téléchargement GitHub).

**4. Structure**
- Racine : `$schema`, `license`, `repository`, `scoreRange`, `lastUpdate`, `data[]`. Chaque anime contient des métadonnées agrégées et des `sources` vers MAL, AniList, Kitsu, AniDB, ANN, LiveChart, Simkl… ([README](https://github.com/manami-project/anime-offline-database)). Dernière release : 41 537 entrées (65 % revues).

**5. Couverture**
- Anime uniquement, métadonnées de base et **correspondances d'identifiants entre fournisseurs**. Pas de synopsis riche, pas de planning, pas de news.

**6. Licence**
- **ODbL 1.0** (badge et fichier [LICENSE](https://github.com/manami-project/anime-offline-database/blob/2026-27/LICENSE)) : attribution et partage à l'identique pour les bases dérivées.

**7. Fiabilité**
- **Dépôt archivé.** Dernière release `2026-27` le 2026-07-04. Tous les dépôts liés de `manami-project` (`modb-app`, `manami`, `kommand`) sont archivés entre fin juin et début juillet 2026 (`gh api users/manami-project/repos`). Aucune annonce de fin n'a été trouvée dans le README. **Les données sont figées.**

---

## Sources de news

| Source | URL | Format | Langue | Fraîcheur observée (2026-11-13) | Statut documentaire | Conditions |
|---|---|---|---|---|---|---|
| ANN, toutes rubriques | `https://www.animenewsnetwork.com/all/rss.xml?ann-edition=w` | RSS 2.0 | en-US | Dernier item le 2026-11-13 ; 159 items | Flux exposé par le site (pas de page de doc trouvée) | `<copyright>© Anime News Network LLC</copyright>` ; `Cache-Control: public, max-age=14400` (4 h). CGU de réutilisation **non trouvées** |
| ANN, édition française | `https://www.animenewsnetwork.com/news/rss.xml?ann-edition=fr` | RSS 2.0 | fr-FR | **Item le plus récent : 2025-02-28** (40 items, du 27 au 28/02/2025) | Idem | Idem. **Édition FR sans publication depuis plus de 20 mois** |
| MAL News | `https://myanimelist.net/rss/news.xml` | RSS 2.0 (+ `media:`) | en | Dernier item le 2026-11-13 ; 20 items | Flux exposé par le site | `Cache-Control: no-cache`. CGU de réutilisation du flux **non vérifiées** |
| MAL News par titre (Jikan) | `GET /v4/anime/{id}/news` | JSON | en | Non testé (API injoignable) | [OpenAPI Jikan](https://github.com/jikan-me/jikan-rest/blob/master/storage/api-docs/api-docs.json) | Mêmes conditions que Jikan ; pas de flux global |
| Crunchyroll News | `https://cr-news-api-service.prd.crunchyrollsvc.com/v1/en-US/rss` | RSS 2.0 | en-US | Dernier item le 2026-11-13 ; 50 items | **Endpoint non documenté publiquement, observé fonctionnel.** Non lié depuis `crunchyroll.com/news` (HTML vérifié) | `<copyright>© Crunchyroll, LLC</copyright>`. CGU **non vérifiées** |
| Crunchyroll News FR | `https://cr-news-api-service.prd.crunchyrollsvc.com/v1/fr-FR/rss` | RSS 2.0 | fr | Dernier item le 2026-11-13 ; 50 items | Idem (non documenté) | Idem |
| LiveChart, titres d'actualité | `https://www.livechart.me/feeds/headlines` | RSS | en | Non testé | [Doc RSS](https://www.livechart.me/pages/rss) : « up to 50 of our most recent headlines » | « provided as-is with no warranty or guarantee of uptime » ; `If-Modified-Since` conseillé |
| LiveChart, épisodes diffusés | `https://www.livechart.me/feeds/episodes` | RSS | en | Non testé | [Doc RSS](https://www.livechart.me/pages/rss) : épisodes diffusés « in the past 24 hours », sans les épisodes à venir | Idem |

Exemple d'item ANN (**observé**) :
```xml
<item>
  <title>J-Novel Club Licenses Putting a Price on World Peace Novels, …</title>
  <link>https://www.animenewsnetwork.com/news/2026-11-13/…/.243781</link>
  <guid isPermaLink="true">https://www.animenewsnetwork.com/cms/.243781</guid>
  <description>Company to also release &lt;cite&gt;Rebuild World&lt;/cite&gt; novels, …</description>
  <pubDate>Fri, 13 Nov 2026 15:00:00 -0500</pubDate>
  <category>Novels</category><category>Manga</category>
</item>
```

Aucune des API de catalogue ci-dessus (AniList, MAL v2, Kitsu, Shikimori, TMDB, AnimeSchedule) ne fournit de flux d'actualités global.

---

## Écartées

| Candidat | Raison | Source |
|---|---|---|
| **TheTVDB** | Pas d'offre gratuite affichée : l'API est « available for both commercial projects and to individual developers », sans prix publiés (contact commercial), avec de simples « significant discounts » pour l'open source. Attribution avec lien obligatoire. | [thetvdb.com/api-information](https://thetvdb.com/api-information) |
| **LiveChart.me (API)** | « We do not offer a public API at this time. » Seuls les flux RSS existent (retenus dans les news). | [FAQ LiveChart](https://www.livechart.me/pages/faq) |
| **anime-offline-database** (comme source vivante) | Projet archivé en juillet 2026, données figées. Il reste utilisable comme **instantané** de correspondances d'identifiants (ODbL). | [dépôt](https://github.com/manami-project/anime-offline-database) |

---

## Points ouverts / non vérifiés

1. **AniDB** : rate limits, règles de cache, risque de ban, CGU et structure complète non lus en source primaire, car la documentation est derrière un challenge Cloudflare (403 observé). Seul fait établi : un client enregistré est obligatoire (erreur 302 observée). Il faut ouvrir [HTTP_API_Definition](https://wiki.anidb.net/HTTP_API_Definition) et [UDP_API_Definition](https://wiki.anidb.net/UDP_API_Definition) dans un navigateur.
2. **Jikan** : API injoignable depuis le poste de test le 2026-11-13. À retester depuis un autre réseau ; vérifier un payload réel et les en-têtes de cache (`Expires`, `ETag`).
3. **MAL API v2** : aucun rate limit chiffré publié ; payload réel non observé (Client ID nécessaire). Le flux d'autorisation documenté est « OAuth2 implicit » dans la référence ; le flux réellement exigé (PKCE ou autre) n'a pas été vérifié.
4. **AniList, clause 5 des CGU** : la portée exacte de « competing, non-complementary services… anime and manga list or tracker services » pour une watchlist doublée de tier lists n'est pas tranchée ici. La doc renvoie à `contact@anilist.co` pour les questions commerciales ; la tolérance « educational » ne vise explicitement que la clause 3.
5. **Kitsu** : CGU (`kitsu.app/terms`) non lisibles ; rate limits non documentés ; route `/trending/anime` fonctionnelle mais non documentée ; domaine canonique (`kitsu.io` ou `kitsu.app`) non précisé par la doc, les deux répondant.
6. **Shikimori** : CGU de l'API non trouvées ; comportement en cas de dépassement de 5 req/s ou 90 req/min non documenté.
7. **TMDB** : ID du genre « Animation » non vérifié ; repli de langue quand `fr-FR` manque non documenté ; aucun payload réel (clé requise).
8. **Crunchyroll RSS** : endpoint non documenté publiquement ; CGU de réutilisation non vérifiées ; pérennité inconnue.
9. **ANN RSS et MAL RSS** : aucune condition de réutilisation spécifique aux flux trouvée (seulement les mentions de copyright).
10. **AnimeThemes** : 503 au moment du test ; structure réelle et CGU des données non vérifiées.
11. **Annict** : langue des données (titres japonais seulement ?) non vérifiée faute de token. Les nouvelles CGU entrent en vigueur le 2026-11-25.
12. **Simkl** et **AnimeSchedule** : payloads réels non observés (identifiants requis). Texte intégral des CGU AnimeSchedule non lu (seulement le résumé dans la doc).
13. **Titres français** : aucun titre ni synopsis en français n'a été trouvé sur l'échantillon testé (FMA:B) chez AniList, Kitsu, Shikimori ou ANN (ce dernier a des dates de diffusion françaises). TMDB documente des traductions (`language=fr-FR`) mais n'a pas été testé. Il faut vérifier sur le jeu d'animés du lot 0.
