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

Lancer `tinacms dev` une première fois pour générer `tina-lock.json`, le commiter et le pousser (voir [étape 4 de la mise en route](README.md#tina-lock-json) du README).
