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
                <input type="checkbox" id="todo-${item.id}" ${item.done ? "checked" : ""} /> 
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
  fetch(`${SUPABASE_URL}?order=created_at`, {
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

-    // On appelle la fonction créée plus tôt qui ajoutera la tâche dans le <ul>
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

# Passer les éléments à "fait" ou "pas fait"

```js
// src/app.js
// Nous souhaitons intervenir lors d'un click sur une checkbox
const onClickCheckbox = (e) => {
    // Nous récupérons l'identifiant de la checkbox (ressemble à "todo-1" ou "todo-23" ...)
  const inputId = e.target.id; 
  // Nous en déduisons l'identifiant de la tâche dans Supabase (ne récupérant que le nombre)
  const id = +inputId.split("-").pop();  
  // On découvre si la checkbox était déjà cochée ou pas
  const isDone = e.target.checked;

  // Nous empêchons le comportement par défaut de l'événement (cocher ou décocher)
  e.preventDefault();

  fetch(`${SUPABASE_URL}?id=eq.${id}`, {
    method: "PATCH",
    headers: {
      "Content-Type": "application/json",
      apiKey: SUPABASE_API_KEY,
      Prefer: "return=representation",
    },
    body: JSON.stringify({ done: isDone }),
  }).then(() => {
      // Lorsque le serveur a pris en compte la demande et nous a répond
      // Nous cochons (ou décochons) la case
    e.target.checked = isDone;
  });
};
```

```js
 // src/app.js à la fin de la fonction addTodo()
 document
    // Nous sélectionnons la checkbox fraichement ajoutée au DOM
    .querySelector("input#todo-" + item.id)
    // Et nous lions la fonction onClickCheckbox au click 
    .addEventListener("click", onClickCheckbox);
```

# Refactoring des appels HTTP
Expliquer la problématique du couplage, et même du bordel dans notre code
Et expliciter la notion de modules en JS

```js
// src/api.js
const SUPABASE_URL = "https://IDENTIFIANT_SUPABASE.supabase.co/rest/v1/todos";
const SUPABASE_API_KEY = "CLE_API_SUPABASE";

/**
 * Récupère les items sur Supabase
 * @returns Promise<array>
 */
export const loadTodoItemsFromApi = () => {
  return fetch(`${SUPABASE_URL}?order=created_at`, {
    headers: {
      apiKey: SUPABASE_API_KEY,
    },
  }).then((response) => response.json());
};

/**
 * Modifie le statut d'une tâche sur Supabase
 * @returns Promise<array>
 */
export const toggleComplete = (id, done) => {
  return fetch(`${SUPABASE_URL}?id=eq.${id}`, {
    method: "PATCH",
    headers: {
      "Content-Type": "application/json",
      apiKey: SUPABASE_API_KEY,
      Prefer: "return=representation",
    },
    body: JSON.stringify({ done: done }),
  });
};

/**
 * Créé une nouvelle tâche dans Supabase
 * @returns Promise<{id: number, text: string, done: boolean}>
 */
export const saveTodoItemToApi = (item) => {
  return fetch(SUPABASE_URL, {
    method: "POST",
    body: JSON.stringify(item),
    headers: {
      "Content-Type": "application/json",
      apiKey: SUPABASE_API_KEY,
      Prefer: "return=representation",
    },
  }).then((response) => response.json());
};
```

Expliquer la notion d'export et d'import

```js
// src/app.js
import {
  loadTodoItemsFromApi,
  toggleComplete,
  saveTodoItemToApi,
} from "./api.js";
```

```js
// src/app.js

// Dès le chargement des élements du DOM
document.addEventListener("DOMContentLoaded", () => {
  // On appelle l'API pour récupérer les tâches
  loadTodoItemsFromApi().then((items) => {
    // Pour chaque tâche, on l'affiche dans l'interface
    items.forEach((item) => addTodo(item));
  });
});

/**
 * Gestion du click sur une Checkbox
 * @param {MouseEvent} e
 */
const onClickCheckbox = (e) => {
  // On récupère l'identifiant de la tâche cliquée (todo-1 ou todo-12 par exemple)
  const inputId = e.target.id;
  // On ne garde que la partie numérique (1 ou 12 par exemple)
  const id = +inputId.split("-").pop();
  // On récupère le fait que la checkbox soit cochée ou pas lors du click
  const isDone = e.target.checked;

  // On annule le comportement par défaut de l'événement (cocher ou décocher la case)
  // Car on ne souhaite cocher / décocher que si le traitement va au bout
  e.preventDefault();

  // On appelle l'API afin de changer le statut de la tâche
  toggleComplete(id, isDone).then(() => {
    // Lorsque c'est terminé, on coche ou décoche la case
    e.target.checked = isDone;
  });
};

/**
 * Permet d'ajouter visuellement une tâche dans l'interface
 * @param {{id: number, text: string, done: boolean}} item
 */
const addTodo = (item) => {
  // On récupère le <ul>
  const container = document.querySelector("ul");

  // On intègre le HTML de la tâche à la fin du <ul>
  container.insertAdjacentHTML(
    "beforeend",
    `
        <li>
            <label>
                <input type="checkbox" id="todo-${item.id}" ${item.done ? "checked" : ""} /> 
                ${item.text}
            </label>
        </li>
    `
  );

  // Alors que la tâche a été ajoutée, on attache au click sur la checkbox la fonction onClickCheckbox
  document
    .querySelector("input#todo-" + item.id)
    .addEventListener("click", onClickCheckbox);
};

// Gestion de la soumission du formulaire
document.querySelector("form").addEventListener("submit", (e) => {
  // On annule le comportement par défaut de la soumission 
  // (qui aurait pour effet de recharger la page, ce qu'on ne souhaite pas vu qu'on souhaite gérer nous-même le comportement)
  e.preventDefault();

  // On récupère l' <input /> du formulaire
  const input = document.querySelector('input[name="todo-text"]');

  // On créé une tâche avec la valeur de l'<input />
  const item = {
    text: input.value,
    done: false,
  };
  
  // On appelle l'API afin de sauver la nouvelle tâche
  saveTodoItemToApi(item).then((items) => {
    // La réponse de l'API est un tableau avec les tâches
    // touchées par le traitement, on prend la première (la seule en fait)
    // Et on l'affiche dans l'interface
    addTodo(items[0]);

    // On vide l'<input /> et on remet le curseur dessus
    input.value = "";
    input.focus();
  });
});
```