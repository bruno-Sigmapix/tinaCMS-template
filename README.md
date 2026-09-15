# Template Astro + TinaCMS + GitHub Pages

Template prêt à l'emploi pour créer un site statique avec [Astro](https://astro.build), [TinaCMS](https://tina.io) comme CMS headless et un déploiement automatique sur GitHub Pages via Docker.

## Mise en route

### 1. Cloner et installer

```bash
git clone <repo-url> mon-site
cd mon-site
docker compose run --rm dev npm install
```

### 2. Créer le projet TinaCloud

1. Aller sur [app.tina.io](https://app.tina.io)
2. **Add Project** > **Existing Project**
3. Sélectionner **Only select repositories** et choisir votre repo
4. Récupérer le **Client ID** et générer un **Token** (Read-Only suffit pour le build)
5. Dans les paramètres du projet TinaCloud, ajouter les **Site URLs** :
   - `http://localhost:4321` (dev local)
   - `https://<username>.github.io/<repo-name>` (production)

### 3. Configurer .env

```bash
cp .env.example .env
```

Remplir les valeurs :

```
NEXT_PUBLIC_TINA_CLIENT_ID=<votre-client-id>
TINA_TOKEN=<votre-token>
```

### 4. Configurer les secrets GitHub

Dans **Settings > Secrets and variables > Actions**, ajouter :

- `TINA_CLIENT_ID` : votre Client ID
- `TINA_TOKEN` : votre Token

### 5. Activer GitHub Pages

Dans **Settings > Pages** :

- **Source** : GitHub Actions

### 6. Configurer un domaine personnalisé (optionnel)

1. Créer un enregistrement DNS `CNAME` pointant vers `<username>.github.io`
2. Dans **Settings > Pages > Custom domain**, entrer votre domaine
3. Créer le fichier `public/CNAME` contenant votre domaine :

```bash
echo "mondomaine.fr" > public/CNAME
```

4. Décommenter et modifier `site` dans `astro.config.mjs` :

```js
site: "https://mondomaine.fr",
```

### 7. Premier déploiement

Aller dans **Actions** > workflow **Deploy to GitHub Pages** > **Run workflow**.

### 8. Lancer le dev local

```bash
docker compose run --rm --service-ports dev
```

Le site est disponible sur `http://localhost:4321` et l'admin TinaCMS sur `http://localhost:4321/admin/`.

## Utilisation

### Flux de publication

1. L'éditeur modifie le contenu via l'interface TinaCMS (`/admin/`)
2. TinaCloud sauvegarde les modifications dans le repo Git
3. Le déploiement se déclenche :
   - **Manuellement** via Actions > Run workflow
   - **Automatiquement** chaque jour à 8h (cron `0 6 * * *` UTC)

### Modifier la config Tina

Après modification de `tina/config.ts` :

```bash
docker compose run --rm dev npx tinacms build --skip-cloud-checks
```

Commiter les fichiers générés **sauf** `tina/__generated__/client.ts` (déjà dans `.gitignore`).

## Troubleshooting

### "local Tina schema doesn't match remote"

Utiliser `--skip-cloud-checks` lors du build, ou attendre que TinaCloud ait indexé le nouveau schéma.

### `client.ts` contient un token en clair

C'est normal. Ce fichier est gitignoré et ne doit jamais être commité. Il est regénéré à chaque `tinacms build` ou `tinacms dev`.
