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

# Routing et affichage dynamique
Introduction au concept de routage


## Mise en place et premiers tests
```js
// src/routing.js

/**
 * Appelle la fonction correspondante à une URL donnée
 * @param {string} url 
 */
export const applyRouting = (url) => {
    let params;

    // Si l'URL ressemble à /12/details
    if (params = url.match(/^\/(\d+)\/details$/)) {
        // On cherche à afficher le détail d'une tâche
        const id = +params[1];

        console.log("J'affiche la tâche n°" + id);
        return;
    }
    // Dans tous les autres cas, on présente la liste des tâches
    console.log("J'affiche la liste des tâches");
}
```

```js
// src/app.js
import { applyRouting } from "./routing.js";
```

```js
// src/app.js
document.addEventListener("DOMContentLoaded", () => {
    // On applique le routage par rapport au pathname (tout ce qui vient après le nom de domaine)
    // Exemple : /12/details
    applyRouting(window.location.pathname);

    loadTodoItemsFromApi().then((items) => {
        items.forEach((item) => addTodo(item));
    });
});
```

## Refactoring pour aller plus loin

Pour aller plus loin, il faut absolument refactoriser notre code qui touche à l'UI afin qu'il puisse être appelé par le routeur :

```js
// src/ui.js

/**
 * Permet d'ajouter visuellement une tâche dans l'interface
 * @param {{id: number, text: string, done: boolean}} item
 */
const addTodo = (item) => {
  // Code d'ajout d'une tâche dans l'interface
};

/**
 * Permet d'ajouter visuellement la liste des tâches dans l'interface
 */
export const displayTodoList = () => {
    // Nous injectons nous même le code HTML de base de la liste des tâches désormais
    // Ainsi que le formulaire, de telle sorte qu'on puisse afficher ces éléments via un simple appel de fonction
    document.querySelector("main").innerHTML = `
          <h2>La liste des tâches</h2>
          <ul></ul>
          <form>
            <input type="text" name="todo-text" placeholder="Ajouter une tâche" />
            <button>Ajouter</button>
          </form>
        `;

    // Une fois le code HTML introduit dans l'interface, on peut afficher les tâches dans le <ul>
    loadTodoItemsFromApi().then((items) => {
        items.forEach((item) => addTodo(item));
    });

    // Il faudra alors ajouter la gestion du submit du <form>
    document.querySelector("form").addEventListener("submit", onSubmitForm);
};

/**
 * Gestion du formulaire d'ajout d'une tâche
 * @param {Event} e
 */
const onSubmitForm = (e) => {
  // Code de gestion du formulaire ...
};

/**
 * Gestion du click sur une Checkbox
 * @param {MouseEvent} e
 */
const onClickCheckbox = (e) => {
  // Code de gestion des checkbox ...
};
```

Désormais, c'est le routeur qui devient responsable des appels de fonctions :

```js
// src/routing.js
import { displayTodoList } from "./ui";

/**
 * Appelle la fonction correspondante à une URL donnée
 * @param {string} url 
 */
export const applyRouting = (url) => {
    let params;

    // Si l'URL ressemble à /12/details
    if (params = url.match(/^\/(\d+)\/details$/)) {
        // On cherche à afficher le détail d'une tâche
        const id = +params[1];

        console.log("J'affiche la tâche n°" + id);
        return;
    }
    // Dans tous les autres cas, on présente la liste des tâches
    displayTodoList();
}
```

Et notre point d'entrée app.js devient effectivement uniquement un point d'entrée dont le seul but est d'initialiser l'application et le routeur :

```js
import { applyRouting } from "./routing.js";

document.addEventListener("DOMContentLoaded", () => {
    applyRouting(window.location.pathname);
});
```

A ce stade, le routeur devrait fonctionner :
* Affichage de la liste lorsqu'on va sur **/**
* Aucun affichage lorsqu'on va sur **/12/details**

## Gestion dynamique des URLs

```diff
// src/ui.js
const addTodo = (item) => {
    const container = document.querySelector("ul");
    container.insertAdjacentHTML(
        "beforeend",
        `
          <li>
              <label>
                  <input type="checkbox" id="todo-${item.id}" ${item.done ? "checked" : ""} /> 
                  ${item.text}
              </label>
+             <a id="goto-${item.id}" href="/${item.id}/details">Détails</a>
          </li>
      `
    );
    document
        .querySelector("input#todo-" + item.id)
        .addEventListener("click", onClickCheckbox);
};
```

On constate que cela fonctionne (bien que rien ne s'affiche, la console nous confirme qu'on se trouve dans le cadre du détails d'une tâche), néanmoins, on cherche à ne pas avoir de rechargement de page afin d'améliorer l'intéractivité et la réactivité de l'application.

```js
// src/routing.js

/**
 * Gestion du click sur un lien
 * @param {MouseEvent} e 
 */
export const onClickLink = (e) => {
    // On empêche le comportement par défaut de l'événement
    // qui reviendrait à réellement naviguer vers l'URL
    e.preventDefault();

    // On récupère l'URL du lien
    const href = e.target.href;

    // On ajoute à l'historique du navigateur ce lien (et par là même, on modifie l'URL dans la barre d'adresse)
    window.history.pushState({}, '', href);
}
```

```js
// src/ui.js
import { onClickLink } from "./routing";

const addTodo = (item) => {
  // ...

  document
      .querySelector('a#goto-' + item.id)
      .addEventListener('click', onClickLink);
}
```
A ce stade on constate bien que l'adresse du navigateur réagit. On perd par contre le comportement attendu, la page ne change plus, seule l'URL est affectée, mais pas l'application.

Pour cela, il faut conditionner l'application à réagir à un événement précis : `popstate` :

```js
// src/routing.js

// On ajoute la gestion de l'événement popstate qui a lieu à chaque fois
// que l'historique du navigateur change
window.addEventListener('popstate', (e) => {
    console.log("Changement d'URL");
    applyRouting(window.location.pathname);
});
```
Néanmoins, cela ne sera pas suffisant, car l'appel à `window.history.pushState()` n'émet pas automatiquement ce fameux événement `popstate` que l'on attend ! On peut néanmoins le faire nous-même :

```js
// src/ui.js

/**
 * Gestion du click sur un lien
 * @param {MouseEvent} e 
 */
const onClickLink = (e) => {
    // On empêche le comportement par défaut de l'événement
    // qui reviendrait à réellement naviguer vers l'URL
    e.preventDefault();

    // On récupère l'URL du lien
    const href = e.target.href;

    // On ajoute à l'historique du navigateur ce lien (et par là même, on modifie l'URL dans la barre d'adresse)
    window.history.pushState({}, '', href);

    // On déclenche manuellement un événement popstate afin que le routeur soit conscient qu'il doit retravailler
    window.dispatchEvent(new PopStateEvent('popstate'));
}
```

## Page de détails d'une tâche
A ce stade, on constate bien dans la console qu'en clickant sur des liens de détails de tâche, le routeur s'en apperçoit mais nous n'avons rien à afficher, il faut donc créer un comportement qui soit capable de **remplacer la liste par le détail** :

```js
// src/api.js

/**
 * Récupère une tâche sur Supabase grâce à son identifiant
 * @param {number} id 
 * @returns Promise<{id: number, text: string, done: boolean}>
 */
export const loadTodoItemFromApi = (id) => {
    return fetch(`${SUPABASE_URL}?id=eq.${id}`, {
        headers: {
            "Content-Type": "application/json",
            apiKey: SUPABASE_API_KEY,
            Prefer: "return=representation",
        },
    })
        .then((response) => response.json())
        .then((items) => items[0]);
};
```

```js
// src/ui.js


/**
 * Affiche dans l'interface le détails d'une tâche
 * @param {number} id 
 */
export const displayTodoDetails = (id) => {
    // On appelle l'API afin de récupérer une tâche
    loadTodoItemFromApi(id).then((item) => {
        // On injecte du HTML dans le <main> 
        // (supprimant donc ce qu'il contient à ce stade)
        document.querySelector("main").innerHTML = `
                <h2>Détails de la tâche ${item.id}</h2>
                <p><strong>Texte :</strong> ${item.text}</p>
                <p><strong>Status : </strong> ${item.done ? "Complété" : "A faire"}</p>
                <a id="back" href="/">Retour à la liste</a>
            `;
        
        // On n'oublie pas que le lien doit être géré par le routeur
        document.querySelector('a#back')
            .addEventListener('click', onClickLink);
    });
};
```