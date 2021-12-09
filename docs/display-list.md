# Afficher la liste des tâches

Dans une application front, les différentes responsabilités suivent un schéma relativement simple : 
1. Les données ainsi que les traitements sont gérées par le Javascript
2. L'interface est gérée via le HTML

Le Javascript va donc projeter des données dans l'interface HTML, alors que le HTML pourra appeler le Javascript lors des diverses intéractions de l'utilisateur avec la page.

## Sommaire
  * [But de l'exercice](#but-de-l-exercice)
  * [Modéliser une liste de tâches dans le JS](#modéliser-une-liste-de-tâches-dans-le-js)
  * [Mettre en place un conteneur dans le HTML](#mettre-en-place-un-conteneur-dans-le-html)
  * [Afficher les items du tableau dans le HTML](#afficher-les-items-du-tableau-dans-le-html)
  * [Vérifier la bonne marche de notre application](#vérifier-la-bonne-marche-de-notre-application)
  * [L'avantage du watch avec Wepback](#l-avantage-du-watch-avec-wepback)
  * [Ce que vous avez appris](#ce-que-vous-avez-appris--)

## But de l'exercice 
Nous souhaitons afficher de façon dynamique une liste de tâches au sein de la page. En suivant le schéma explicité ci-dessus, nous comprenons :

1. Le Javascript devra mettre en place les données (une liste de tâches sous la forme d'un tableau d'objets)
2. Le Javascript devra aussi posséder un comportement qui permettra de projeter ces données dans le HTML
3. Le HTML devra être capable d'accueillir cette projection dans un *conteneur* (un élément HTML)

**Ici, on tentera vraiment de comprendre que l'on peut et comment on peut projeter du contenu dans l'interface HTML depuis le code Javascript**

## Modéliser une liste de tâches dans le JS

Créons tout d'abord la liste des tâches (en dur pour l'instant) au sein de notre fichier src/app.js

```js
// src/app.js

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];
```

## Mettre en place un conteneur dans le HTML

Modifions maintenant le fichier index.html afin qu'il contienne un élément qui accueillera la liste des tâches :

```html
<main>
    <h2>La liste des tâches</h2>
    <!-- Le ul sera le conteneur pour nos tâches -->
    <ul></ul>
</main>
```

## Afficher les items du tableau dans le HTML

Il nous faut maintenant matérialiser les comportements qui permettront d'afficher la liste des tâches :


```js
// src/app.js

// On créé ici un tableau TODO_ITEMS qui contient deux objets 
const TODO_ITEMS = [
  { id: 1, text: "Faire les courses", done: false },
  { id: 2, text: "Aller chercher les enfants", done: true },
];

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

## Vérifier la bonne marche de notre application
Encore une fois, on constate en actualisant notre navigateur que rien n'a changé : nous n'avons tout simplement pas "bundlé" notre code, et le fichier *dist/app.js* n'a donc pas pris en compte nos derniers changements. N'oublions donc pas de lancer la commande `npm run dev` avant d'actualiser à nouveau le navigateur pour vérifier que tout fonctionne.

## L'avantage du watch avec Wepback

Vous pourriez éventuellement trouver lourdingue de devoir lancer à chaque modification de votre code la commande `npm run dev` afin que Webpack fasse le bundling de l'application. 

C'est pourquoi Webpack intègre une possibilité de "watching" (surveillance) de votre code source afin de se déclencher automatiquement à chaque changement.

```diff
// package.json

"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development",
+    "watch": "webpack --mode development --watch",
    "serve": "live-server --entry-point=./index.html"
  },
```

En lançant la commande `npm run watch` vous verrez que Webpack va se lancer et ne plus vous rendre la main : il attend que des changements soient fait dans votre dossier *src/* pour se relancer automatiquement.

# Ce que vous avez appris :
* Le flux des données dans une application front : les données et comportements sont gérés dans le Javascript. Les données sont projetées par Javascript dans l'interface (HTML), alors que l'interface peut déclencher des traitements du Javascript ;
* Utiliser l'option `--watch` de Webpack afin de ne pas avoir à relancer sans cesse le bundling de nos fichiers sources

[Revenir au sommaire](../README.md) ou [Passer à la suite : Ajouter une tâche](add-item.md)
