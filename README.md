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

# Afficher la liste des items

Expliquer le but de l'exercice

1. Mettre en place des items dans le Javascript
```js
// src/app.js
// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];
```

2. Mettre en place un conteneur dans le HTML
```html
<h2>La liste des tâches</h2>
<!-- Le ul sera le conteneur pour nos tâches -->
<ul></ul>
```

3. Afficher les items du tableau dans le HTML
```js
// src/app.js
// Nous créons une fonction qui servira à ajouter un élément dans le UL à partir d'un objet tâche
const addTodo = (item) => {
    // On sélectionne le <ul>
  const container = document.querySelector("ul");

  // On ajoute du HTML à la fin du <ul>
  container.insertAdjacentHTML(
    "beforeend",
    `
        <li>
            <label>
                <input type="checkbox" id="${item.id}" ${
      item.done ? "checked" : ""
    } /> 
                ${item.text}
            </label>
        </li>
    `
  );
};

// Pour chaque élément du tableau TODO_ITEMS, on appelle la fonction addTodo en fournissant
// l'item
TODO_ITEMS.forEach((item) => addTodo(item));
```