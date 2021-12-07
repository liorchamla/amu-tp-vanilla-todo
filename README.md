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

# Vérification de l'outillage et des liens
```js
// src/app.js
console.log("Tout fonctionne");
```

```html
<script src="dist/app.js"></script>
```

`npm run dev`

S'assurer qu'on a bien "Tout fonctionne" dans la console


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
<main>
    <h2>La liste des tâches</h2>
    <!-- Le ul sera le conteneur pour nos tâches -->
    <ul></ul>
</main>
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
                <input type="checkbox" id="${item.id}" ${item.done ? "checked" : ""} /> 
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

# L'avantage du watch avec Wepback

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development",
    "serve": "live-server",
    "watch": "webpack --mode development --watch"
  },
```

# Ajout d'une nouvelle tâche
```html
<main>
    <h2>La liste des tâches</h2>
    <ul></ul>
    <form>
        <input type="text" name="todo-text" placeholder="Ajouter une tâche" />
        <button>Ajouter</button>
    </form>
</main>
```

```js
// src/app.js
// On souhaite réagir à chaque fois que le formulaire est soumis
document.querySelector("form").addEventListener("submit", (e) => {
  // On souhaite aussi empêcher le rechargement de la page
  e.preventDefault();

  // On récupère l'input
  const input = document.querySelector('input[name="todo-text"]');

  // On créé une nouvelle tâche avec pour text la valeur tapée dans l'input
  const item = {
    id: Date.now(),
    text: input.value,
    done: false,
  };
  
  // On appelle la fonction créée plus tôt qui ajoutera la tâche dans le <ul>
  addTodo(item);

  // On vide l'input et replace le curseur dedans
  input.value = "";
  input.focus();
});
```

# Appels HTTP vers une API REST

Expliquer le principe de Supabase (et outils apparentés tels que Firebase et autres)

Guider vers la création d'un compte Supabase et expliciter la création d'une table SQL qui contiendra les données

Linker vers la doc de l'API REST Supabase

Nous ne souhaitons plus utiliser le tableau JS TODO_ITEMS, nous pouvons donc supprimer toute référence à celui ci

```js
// src/app.js
const SUPABASE_URL = "https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos";
const SUPABASE_API_KEY = "CLE_API_SUPABASE";

// Lorsque les éléments du DOM sont tous connus
document.addEventListener("DOMContentLoaded", () => {
  // Appel HTTP vers Supabase
  fetch(SUPABASE_URL, {
    headers: {
      apiKey: SUPABASE_API_KEY,
    },
  })
    .then((response) => response.json())
    .then((items) => {
      items.forEach((item) => addTodo(item));
    });
});
```
Expliciter le système des promesses JS (rapidement)

```diff
// On souhaite réagir à chaque fois que le formulaire est soumis
document.querySelector("form").addEventListener("submit", (e) => {
    // On souhaite aussi empêcher le rechargement de la page
    e.preventDefault();

    // On récupère l'input
    const input = document.querySelector('input[name="todo-text"]');

    // On créé une nouvelle tâche avec pour text la valeur tapée dans l'input
    const item = {
-        id: Date.now(),
        text: input.value,
        done: false,
    };

    // On appelle la fonction créée plus tôt qui ajoutera la tâche dans le <ul>
-    addTodo(item);
+    fetch(SUPABASE_URL, {
+        method: "POST",
+        body: JSON.stringify(item),
+        headers: {
+            "Content-Type": "application/json",
+            apiKey: SUPABASE_API_KEY,
+            Prefer: "return=representation",
+        },
+    })
+    .then((response) => response.json())
+    .then((items) => {
+      addTodo(items[0]);
+      input.value = "";
+      input.focus();
+    });

-    // On vide l'input et replace le curseur dedans
-   input.value = "";
-   input.focus();
});
```