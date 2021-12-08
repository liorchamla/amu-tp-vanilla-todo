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