# TP de Javascript (Vanilla)

TODO: Introduire le TP et parler de l'outillage

# Installation de dépendances de développement

`npm init -y`

Expliquer le concept de package.json
Expliquer l'apparition du dossier node_modules ainsi que package.lock

## Webpack 

`npm install --save-dev webpack webpack-cli`

```js
// webpack.config.js
// Permet de résoudre des chemins de fichiers
const path = require("path");

module.exports = {
  // Le fichier qui sert de point d'entrée et qui importe les différentes dépendances de l'application
  entry: "./src/app.js",
  // Le dossier et nom du fichier qui sera généré par Webpack après le build
  // Webpack va donc intégrer l'ensemble des modules (dépendances) nécessaires dans un seul fichier dist/app.js
  output: {
    filename: "app.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development"
  },
```

`npm run dev` et webpack va générer le fichier dist/app.js

# Lancer l'application dans le navigateur

`npm install --save-dev live-server`

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development",
    "serve": "live-server"
  },
```
