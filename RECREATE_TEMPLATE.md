# Recréer le template de zéro

Ces étapes ne sont nécessaires que si vous voulez reconstruire le socle technique depuis un projet vide, sans utiliser le template. Pour l'utilisation normale du template, voir [README.md](README.md).

```bash
mkdir mon-site && cd mon-site
git init

# Initialiser Astro
npm create astro@latest . -- --template minimal --typescript strict

# Ajouter TinaCMS
npx @tinacms/cli@latest init

# Ajouter React et Tailwind
npm install @astrojs/react react react-dom @tailwindcss/vite tailwindcss
```

Ensuite, reproduire la structure du template :

- `tina/config.ts` : définir les collections, le build (`outputFolder: "admin"`) et les médias
- `astro.config.mjs` : ajouter les intégrations `react()` et `tailwindcss()`
- `src/layouts/Layout.astro` : layout avec header, nav, footer
- `src/pages/index.astro` : page d'accueil qui fetch le contenu via le client TinaCMS
- `content/pages/home.mdx` : contenu d'exemple
- `compose.yaml` : services Docker (`dev`, `node`)
- `.github/workflows/deploy.yml` : workflow de déploiement GitHub Pages
- `.gitignore` : exclure `client.ts`, `.cache/`, `public/admin/`, `.env`

## Générer `tina-lock.json`

Contrairement au flux normal du template (où `tina/tina-lock.json` est déjà commité et copié automatiquement par "Use this template", voir [étape 4 du README](README.md#tina-lock-json)), ici le fichier n'existe pas encore : ce projet part de zéro, il faut donc le générer soi-même. C'est une étape facile à manquer car rien ne l'indique explicitement dans le flux de setup habituel -- si TinaCloud affiche "No Tina config found" ou n'indexe aucune branche, c'est généralement que cette étape a été oubliée.

Le fichier est généré uniquement par `tinacms dev` (au démarrage du serveur), **pas** par `tinacms build`. Procédure :

1. Lancer le serveur de dev : `docker compose run --rm --service-ports dev`
2. Attendre que le serveur démarre -- `tina/tina-lock.json` est créé à ce moment-là -- puis l'arrêter (Ctrl+C)
3. Commiter et pousser le fichier généré :

```bash
git add tina/tina-lock.json
git commit -m "Add tina-lock.json for TinaCloud indexing"
git push
```

4. Dans TinaCloud, cliquer **Refresh Branches** -- la branche `main` doit apparaître

> **Ne pas ajouter `tina-lock.json` au `.gitignore`.** TinaCloud en a besoin pour l'indexation, et il n'est pas régénérable côté TinaCloud lui-même.

Voir aussi [Troubleshooting](README.md#no-tina-config-found) dans le README si la branche ne s'indexe toujours pas après ça.
