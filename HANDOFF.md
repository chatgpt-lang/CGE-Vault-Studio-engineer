# CGE Vault Studio — passation technique

## Contenu livré

- `index.html` : application complète (interface, CSS et JavaScript regroupés).
- `assets/` : logos, fonds, serpents découpés, socle, cartes, produits et pictogrammes.
- `render-service/` : service Node.js / Playwright / FFmpeg pour les MP4.
- `render-service/Dockerfile`, `cloudbuild.yaml` et `DEPLOY_CLOUD_RUN.md` : conteneur et déploiement.
- `render-service/.env.example` : variables attendues, sans secret réel.

Le ZIP exclut `.git`, `.DS_Store`, `.env`, `node_modules`, caches, logs et rendus temporaires.

## Lancement local

```bash
cd cge-vault-studio
python3 -m http.server 8766 --bind 127.0.0.1
```

Ouvrir `http://127.0.0.1:8766/index.html`. Le mode local contourne volontairement l’écran d’accès.

## Objectif et ateliers

Studio client pour composer, détourer et exporter des visuels CGE Vault System :

1. `live` — Annonce de live V1 : Story, Post, Miniature.
2. `collection` — Annonce de live V2 : Story, Post, Miniature.
3. `recap` — Retour sur le live : Story, Post.
4. `duel` — Meilleur artefact récupéré : Story, Post.

Une ancienne clé interne `tree` subsiste dans le code, mais l’atelier n’est plus visible dans la démonstration.

## Fonctionnalités

- Textes, couleurs, fonds, calques, taille, rotation et déplacement.
- Compositions séparées par atelier et format.
- Import de produits, cartes et éléments.
- Détourage PhotoRoom optionnel via Make, avec cache SHA-256 dans IndexedDB.
- Bibliothèques locales de produits et fonds.
- Export PNG HD, choix des formats et ZIP.
- Export MP4 serveur avec Playwright/FFmpeg ; repli navigateur en local.
- Annulation des jobs vidéo et fichiers temporaires.
- Accès par code, mémorisation 30 jours et blocage après cinq erreurs.
- Chat Sarah via `/api/chat-sarah` quand le serveur est configuré.

## État visuel à préserver

- Univers noir/gris métal, accent orange par défaut et variantes colorées.
- Titres métalliques utilisant `assets/cge/metal-texture.png` et le filtre `cge-metal-bevel`.
- Atelier 1 : serpent arrière + anneaux avant, produits placés entre les deux.
- Atelier 3 : serpent supérieur, langue, œil lumineux, anneaux avant et 1 ou 3 cartes. Serpent et cartes réduits d’environ 50 px en Story/Post ; « POUR CE LIVE » passe derrière le serpent.
- Atelier 4 : carte en lévitation, socle, titres séparés `ARTEFACT` / `RÉCUPÉRÉ`, sous-titre centrable et option deux lignes. En Post, logo, titres, carte et socle sont déplaçables.

## Persistance navigateur

- Compositions : `localStorage`, clé `cge-compositions-v1`.
- Produits enregistrés : clé historique `f2k-saved-products`.
- Fonds et détourages : IndexedDB `cge-studio-assets`, stores `backgrounds` et `cutouts`.
- Les imports directs en URL `blob:` ne survivent pas toujours au rechargement : à migrer vers IndexedDB ou stockage serveur.

## Accès et sécurité

- Empreinte du code dans `ACCESS_CODE_HASH` ; code de démo : `F2K-STUDIO-26!` (nom historique).
- Session locale 30 jours ou session navigateur ; blocage 15 minutes après cinq erreurs.
- Le serveur peut créer un cookie `HttpOnly` via `/api/session`.
- Pour la production : comptes individuels, révocation, sessions serveur, quotas et journalisation.

## PhotoRoom / Make

- L’URL Make est actuellement dans `CUTOUT_WEBHOOK_URL` dans le HTML.
- La clé PhotoRoom doit rester dans Make ou dans le backend, jamais dans le HTML.
- Avant publication large, proxifier cette route côté serveur avec authentification, validation des fichiers, taille maximale, quotas et rate limiting.

## Export PNG

- `html-to-image@1.11.13` et `JSZip@3.10.1` sont chargés depuis jsDelivr.
- Les dimensions sont définies par `exportDimensions()`.
- Le studio attend images, polices et animations d’entrée avant la capture.

## Export vidéo

Flux : HTML autonome avec assets en data URLs → `POST /api/video-jobs` → capture Playwright → encodage FFmpeg H.264 `yuv420p` → téléchargement temporaire.

Le studio demande actuellement 5 secondes à 20 FPS pour réduire le temps de rendu. La rétention par défaut est de 24 heures.

Routes :

- `GET /api/health`
- `POST /api/session`
- `POST /api/video-jobs`
- `GET /api/video-jobs/:id`
- `DELETE /api/video-jobs/:id`
- `GET /api/video-jobs/:id/download`
- `POST /api/chat-sarah`

Variables serveur : `F2K_RENDER_API_KEY`, `F2K_ACCESS_CODE_HASH`, `F2K_SESSION_SECRET`, `F2K_RETENTION_MS`, `F2K_ALLOWED_ORIGIN`, `CHAT_SARAH_WEBHOOK_URL`, `PORT`.

Les noms `F2K_*` sont hérités du premier client mais encore utilisés par le service CGE. Les renommer uniquement avec une migration coordonnée du client, du serveur et de Cloud Run. Ne jamais committer les valeurs réelles.

## Limites techniques

- File et état des jobs vidéo en mémoire ; un worker et une instance recommandés.
- Jobs perdus au redémarrage. Pour la production : Cloud Tasks/SQS, stockage objet et base de statuts.
- `index.html` est volumineux et contient beaucoup de règles spécifiques : le scinder en modules CSS/JS.
- Dépendances CDN à héberger localement ou encadrer par une CSP.
- Webhook PhotoRoom visible et authentification partagée à remplacer avant une diffusion publique.

## Recette recommandée

Pour chaque atelier/format : modifier textes/couleurs, importer avec et sans détourage, déplacer et redimensionner, changer de format puis revenir, exporter PNG/ZIP/MP4, annuler un MP4, vérifier l’absence de contrôles dans l’export et tester Chrome/Safari.

## Priorités ingénieur

1. Sortir PhotoRoom/Make du HTML public.
2. Mettre une authentification révocable et des quotas.
3. Persister tous les imports.
4. Refactorer `index.html`.
5. Ajouter tests de formats et non-régression visuelle.
6. Externaliser la file vidéo avant plusieurs instances.
7. Ajouter métriques, alertes de coût et suppression vérifiable.

## Déploiement

`render-service/DEPLOY_CLOUD_RUN.md` vient du premier déploiement F2K. Vérifier projet Google Cloud, service, domaine, secrets et compte de service avant toute commande. Ne pas réutiliser aveuglément une commande de production.
