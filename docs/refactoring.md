# Refactoring des appels HTTP et système de modules

Notre application est en bonne voie ! Elle fonctionne plutôt bien mais son code, par contre, est assez peu clair et relativement entremêlé.

Comme tout langage de programmation, Javascript (même ça lui a manqué pendant des décennies) s'est doté de la possibilité de répartir du code dans différents fichiers grâce à son système de **modules**.

## Sommaire :
- [But de l'exercice :](#but-de-l-exercice--)
- [Créer un fichier api.js qui contiendra les appels HTTP](#créer-un-fichier-apijs-qui-contiendra-les-appels-http)
- [Utiliser les fonctions refactorisées dans notre code principal](#utiliser-les-fonctions-refactorisées-dans-notre-code-principal)
- [Ce que vous avez appris](#ce-que-vous-avez-appris--)

## But de l'exercice :
Nous allons commencer à refactoriser notre code de telle sorte qu'il gagne en clarté. Et pour ce faire nous utiliserons le système de modules de Javascript afin de répartir différents codes dans différents fichiers.

Notre objectif est de centraliser toutes les lignes de code qui ont un rapport avec l'API dans un seul fichier, ainsi :
1. Ces traitements pourront être rappelés facilement si on a de nouveaux besoins ;
2. Notre code sera agnostique de ces détails d'appels HTTP et ne fera qu'appeler les fonctions telles que `loadTodoItemsFromApi()` sans se préoccuper de quel serveur est appelé ou de quels traitements sont réalisés lors de la réponse ;

**Ici, on se concentrera sur le fonctionnement des modules et notament de la façon dont on peut rendre une fonction utilisable dans un autre fichier !**

## Créer un fichier api.js qui contiendra les appels HTTP
On veut donc centraliser dans un nouveau fichier *src/api.js* tout ce qui a un rapport avec notre service Supabase. 

On peut donc extraire l'ensemble des appels `fetch()` qui sont dans notre fichier *src/app.js* et les confier à des fonctions dans le fichier *src/api.js*.

**Attention : remplacez bien dans les 2 constantes votre identifiant Supabase ainsi que la clé d'API que vous trouverez tout deux sur votre projet Supabase**

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

Vous noterez la présence d'un mot clé devant nos fonctions qui pour l'instant n'avait pas été utilisé : `export`

Ce mot clé explique à Javascript que ces fonctions, bien qu'écrites dans le fichier *src/api.js* pourront être utilisées dans d'autres fichiers. Dans le monde Javascript, on appelle ces fichiers **des modules** !

## Utiliser les fonctions refactorisées dans notre code principal

Lorsque l'on souhaite utiliser dans un module une fonction qui se trouve dans un autre module, il suffit de l'importer en précisant de quel module provient cette fonction.

**Important : le fonctionnement de l'export et de l'import marche aussi avec des classes ou des variables** !

```js
// src/app.js

// On importe dans notre module principal des fonctions en provenance
// du module api.js
import {
  loadTodoItemsFromApi,
  toggleComplete,
  saveTodoItemToApi,
} from "./api.js";
```

On va remplacer tous les appels à `fetch()` dans notre code principal par l'appel de nos fonctions refactorisées :

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
Et voilà, normalement, tout devrait fonctionner correctement !

# Ce que vous avez appris :
* Créer différents modules (fichiers JS) afin de répartir correctement vos fonctions et vos responsabilités ;
* Exporter des fonctions d'un module afin qu'elles soient utilisables dans un autre module ;
* Importer des fonctions dans un module à partir d'un autre module ;
* Centraliser certaines fonctions afin de pouvoir les réutiliser et les faire évoluer plus facilement ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Routing et affichage dynamique](routing.md)