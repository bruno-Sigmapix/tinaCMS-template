# Template Astro + TinaCMS + GitHub Pages

Template prêt à l'emploi pour créer un site statique avec [Astro](https://astro.build), [TinaCMS](https://tina.io) comme CMS headless et un déploiement automatique sur GitHub Pages via Docker.

**Contenu du template :** une collection "Pages" (MDX) avec titre, image, meta description et contenu riche. Une page d'accueil d'exemple. Un layout avec navigation (Accueil + Admin). Tailwind CSS v4. Workflow GitHub Actions (déploiement manuel + cron quotidien).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- Un compte [GitHub](https://github.com) avec un repo (public ou privé)
- Un compte [TinaCloud](https://app.tina.io) (gratuit)

---

## Mise en route

### 1. Créer le repo (GitHub)

Sur [bruno-Sigmapix/tinaCMS-template](https://github.com/bruno-Sigmapix/tinaCMS-template), cliquer **"Use this template"** > **"Create a new repository"**.

Pas besoin de le cloner tout de suite : l'étape suivante n'a besoin que du repo GitHub.

### 2. Créer le projet TinaCloud (TinaCloud)

1. Aller sur [app.tina.io](https://app.tina.io) et se connecter
2. **Add Project** > **Existing Project**
3. Sélectionner **Only select repositories** et choisir votre repo
4. Configurer **Set up Git Authoring** : choisir *Act as bot* (TinaCloud commite en son nom) ou *Act as self* (commite au nom de l'utilisateur)
5. Récupérer le **Client ID** sur la page Overview et le token **Content (Readonly)** sur la page Tokens
6. Dans les paramètres du projet TinaCloud, ajouter les **Site URLs** :
   - `http://localhost:4321` (dev local)
   - L'URL de production (domaine custom ou `https://<username>.github.io/<repo-name>`)

### 3. Cloner, configurer .env et installer (local)

```bash
git clone <url-de-votre-nouveau-repo> mon-site
cd mon-site
cp .env.example .env
```

Remplir `.env` avec les valeurs récupérées à l'étape précédente :

```env
NEXT_PUBLIC_TINA_CLIENT_ID=<client-id sur la page overview>
TINA_TOKEN=<token Content (Readonly) sur la page tokens>
```

Puis installer les dépendances :

```bash
docker compose run --rm dev npm install
```

> **Important :** ne pas faire `npm install` en local. Les dépendances doivent être installées dans le container Docker (Alpine/musl). Un `npm install` sur la machine hôte produira des binaires natifs incompatibles avec le container.

> **Pourquoi `--rm` :** sans cette option, chaque `docker compose run` laisse un conteneur arrêté derrière lui. Comme ces commandes sont répétées régulièrement, `--rm` évite d'accumuler des conteneurs morts au fil du temps — c'est pourquoi il est utilisé dans toutes les commandes de ce README.

### 4. Vérifier l'indexation TinaCloud (TinaCloud)

Le fichier `tina/tina-lock.json` est indispensable pour que TinaCloud puisse indexer le contenu et les branches. Il est déjà commité dans le template : **"Use this template" le copie automatiquement dans votre nouveau repo**, vous n'avez rien à générer.

Dans TinaCloud, cliquer **Refresh Branches** -- la branche `main` doit apparaître.

> **Ne pas ajouter `tina-lock.json` au `.gitignore`.** TinaCloud en a besoin pour l'indexation.
>
> Si la branche n'apparaît pas ou que TinaCloud renvoie "No Tina config found", voir [Troubleshooting](#no-tina-config-found) -- vous y trouverez la procédure de génération manuelle du fichier (utile aussi si vous reconstruisez le projet sans le template, voir [RECREATE_TEMPLATE.md](RECREATE_TEMPLATE.md)).

### 5. Configurer les secrets GitHub (GitHub)

Dans **Settings > Secrets and variables > Actions**, ajouter :

| Secret | Valeur | Où la trouver |
|---|---|---|
| `TINA_CLIENT_ID` | Le Client ID de TinaCloud (même valeur que `NEXT_PUBLIC_TINA_CLIENT_ID` dans le `.env`) | [app.tina.io](https://app.tina.io) > Overview |
| `TINA_TOKEN` | Le Read-Only Token de TinaCloud | [app.tina.io](https://app.tina.io) > Tokens |

### 6. Activer GitHub Pages (GitHub)

Dans **Settings > Pages** :

- **Source** : GitHub Actions

> Le menu déroulant "Source" propose deux options : **"GitHub Actions"** (celle à choisir ici) et **"Deploy from a branch"** (l'ancienne méthode). Si GitHub vous demande de préciser une branche, c'est que "Deploy from a branch" est encore sélectionné -- le workflow de ce template (`actions/deploy-pages`) ne pousse pas vers une branche, aucune branche n'est à choisir.

### 7. Configurer le domaine (local + GitHub)

Décommenter et remplir `site` dans `astro.config.mjs` :

```js
site: "https://mondomaine.fr",
```

Pour un domaine personnalisé :

1. Dans **Settings > Pages > Custom domain**, entrer votre domaine et cliquer **Save**
2. Cocher **Enforce HTTPS** (disponible après propagation DNS)
3. Ajouter un enregistrement DNS CNAME chez votre registrar :

```
Type :   CNAME
Nom :    sous-domaine (ex: www)
Valeur : <username>.github.io.
TTL :    3600 (ou auto)
```

4. Créer le fichier `public/CNAME` contenant votre domaine :

```bash
echo "mondomaine.fr" > public/CNAME
```

> **Note :** si vous déployez sans domaine custom (sur `https://<username>.github.io/<repo-name>/`), ajoutez `base: "/<repo-name>"` dans `astro.config.mjs` et `basePath: "<repo-name>"` dans la propriété `build` de `tina/config.ts` pour que les assets et l'admin TinaCMS se chargent correctement.

### 8. Premier déploiement (GitHub)

Aller dans **Actions** > workflow **Deploy to GitHub Pages** > **Run workflow**.

### 9. Lancer le dev local (local)

```bash
docker compose run --rm --service-ports dev
```

Le site est disponible sur `http://localhost:4321` et l'admin TinaCMS sur `http://localhost:4321/admin/`.

> En mode local sans `.env` (ou avec des valeurs vides), TinaCMS utilise le filesystem directement. Les modifications sont écrites dans `content/`.

---

## Utilisation

### Flux de publication

1. L'éditeur modifie le contenu via l'interface TinaCMS (`/admin/`)
2. TinaCloud sauvegarde les modifications dans le repo Git (commit automatique sur main)
3. Le déploiement se déclenche :
   - **Manuellement** via Actions > Run workflow
   - **Automatiquement** chaque jour à 8h (cron `0 6 * * *` UTC)

> Il n'y a pas de déploiement automatique au push. C'est volontaire pour laisser le contrôle à l'éditeur.

### Modifier la config Tina

Après modification de `tina/config.ts` :

```bash
docker compose run --rm dev npx tinacms build --skip-cloud-checks
```

Commiter les fichiers générés, en particulier `config.prebuild.jsx` et `_schema.json`. Ne pas commiter `client.ts` (déjà dans `.gitignore`, contient un token en clair).

---

## Limites du plan gratuit

### GitHub Pages / Actions

- 2 000 minutes d'Actions par mois (un build prend ~1 minute)
- 1 Go de stockage pour le site
- 100 Go de bande passante par mois

### TinaCloud

- 2 utilisateurs
- 2 projets
- Modifications illimitées (chaque sauvegarde = un commit Git)

---

## Recréer le template de zéro (optionnel)

Voir [RECREATE_TEMPLATE.md](RECREATE_TEMPLATE.md) si vous voulez reconstruire le socle technique depuis un projet vide, sans utiliser le template.

---

## Troubleshooting

### "local Tina schema doesn't match remote"

TinaCloud n'a pas encore indexé le dernier schéma :
- Attendre quelques minutes et relancer
- Utiliser `--skip-cloud-checks` pour contourner la vérification
- Vérifier que `config.prebuild.jsx` est bien commité

### `client.ts` contient un token en clair

C'est normal. Ce fichier est gitignoré et ne doit jamais être commité. Il est regénéré à chaque `tinacms build` ou `tinacms dev`. Si `client.ts` a été commité par erreur, révoquer le token dans [TinaCloud > Tokens](https://app.tina.io) et en générer un nouveau.

### TinaCloud : "No Tina config found" / aucune branche indexée

TinaCloud a besoin du fichier `tina/tina-lock.json` pour détecter le schéma. Ce cas se produit typiquement si le fichier a été supprimé, ou si vous reconstruisez le projet sans le template (voir [RECREATE_TEMPLATE.md](RECREATE_TEMPLATE.md)) -- dans le flux normal (`Use this template`), le fichier est déjà présent et cette procédure n'est pas nécessaire.

Le fichier est généré uniquement par `tinacms dev`, **pas** par `tinacms build`. Pour le régénérer :

1. Lancer `docker compose run --rm --service-ports dev`
2. Attendre que le serveur démarre (il génère `tina/tina-lock.json` au démarrage), puis l'arrêter (Ctrl+C)
3. Commiter et pousser le fichier généré :

```bash
git add tina/tina-lock.json
git commit -m "Add tina-lock.json for TinaCloud indexing"
git push
```

4. Cliquer **Refresh Branches** dans TinaCloud -- la branche `main` doit apparaître

Si ça ne fonctionne toujours pas, supprimer le projet dans TinaCloud et le recréer.
