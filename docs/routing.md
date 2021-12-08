# Routing et affichage dynamique
Dans toute application web, le protocole HTTP étant primordial, l'accès à une ressource (que ce soit une page, des données, un fichier etc) se fait via un URL.

Le système de routage est donc lui aussi essentiel : le *routing* c'est le fait de décrire l'ensemble des *routes* disponibles dans votre application. Alors qu'est-ce qu'une *route* ? C'est **l'association entre une URL et un comportement** !

Le routing est donc l'ensemble des associations URL / Comportements de votre application.

Dans une Single Page Application, on chercher à faire croire au visiteur qu'il navigue dans différentes pages (et donc différentes URL) alors qu'en réalité il reste toujours sur la même page (index.html) et c'est Javascript qui donne l'impression que la page change lorsque l'URL change.

## Sommaire :
* [But de l'exercice :](#but-de-l-exercice--)
* [Mise en place et premiers tests](#mise-en-place-et-premiers-tests)
    + [Premier test : la page d'accueil](#premier-test---la-page-d-accueil)
    + [Deuxième test : la future page de détails d'une tâche](#deuxième-test---la-future-page-de-détails-d-une-tâche)
* [Refactoring pour aller plus loin](#refactoring-pour-aller-plus-loin)
* [Gestion dynamique des URLs](#gestion-dynamique-des-urls)
* [Page de détails d'une tâche](#page-de-détails-d-une-tâche)
* [Ce que vous avez appris](#ce-que-vous-avez-appris--)

## But de l'exercice :
Nous souhaitons avoir une deuxième page dans notre application qui sera chargée d'afficher le détail d'une tâche donnée. Nous avons donc deux affichages possibles :
1. La liste des tâches
2. Le détail d'une tâche

Notre but sera ici de mettre en place un système qui soit capable d'afficher un contenu ou un autre en fonction de l'URL du navigateur.

Plus encore, il faudra prendre soin que tous les liens de notre application ne déclenchent aucun rechargement de page, mais continuent de modifier l'URL. Ce sera alors à notre système (qu'on appellera **routeur**) de modifier l'affichage en fonction.


## Mise en place et premiers tests
Nous allons y aller tout doucement et étape par étape. La première étape est de créer notre **Routeur**.

Dans un nouveau module *src/routing.js* nous allons exporter une fonction `applyRouting(url)` qui sera capable de déclencher un comportement ou un autre en fonction d'une URL qu'on lui passe :

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
On peut maintenant faire en sorte qu'au chargement de la page dans le navigateur, on appelle notre fonction en lui passant l'URL qui est tapée par l'utilisateur :

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

### Premier test : la page d'accueil
Si vous allez sur votre navigateur sur l'URL `/` vous devriez voir en console le message correspondant à la liste des tâches.

### Deuxième test : la future page de détails d'une tâche
Si vous allez sur votre navigateur sur l'URL `/1/details` ou pourquoi pas sur `/298/details`, vous devriez voir en console le message correspondant à l'affichage d'une tâche en détails.

## Refactoring pour aller plus loin

Vous l'aurez compris, désormais, le **routeur** (notre fonction `applyRouting(url)`) devient "*le maître*" du jeu. C'est lui qui dit ce qu'on affiche ou pas.

Pour aller plus loin, il faut absolument refactoriser notre code qui touche à l'UI dans un module *src/ui.js* afin qu'il puisse être appelé par le routeur :

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

Désormais, on veut offrir une véritable navigation à l'utilisateur. Commençons par lui donner la possibilité d'aller voir le détail d'une tâche en ajoutant à chaque tâche un lien :

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

On va donc créer une fonction dont le but est d'intercépter les clicks sur un lien afin de stoper le comportement par défaut (le changement de la page par le navigateur lui-même) et de prendre en charge nous même le comportement à mettre en oeuvre.

On utilisera la fonction `window.history.pushState()` pour ajouter une URL à l'historique du navigateur (ce qui se matérialise visuellement par l'URL qui change dans le navigateur). 

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

On peut maintenant rattacher la fonction `onClickLink` au lien qui permet d'aller voir le détails d'une tâche :

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

Vous l'aurez compris, l'événement `popstate` est déclenché à chaque fois que l'historique du navigateur change.

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

Désormais, on le voit, lorsque l'on visite la route `/` et que l'on clique sur le lien de détails d'une tâche, non seulement l'URL change, mais on voit aussi que le routeur réagit bien car dans la console vous devriez voir qu'il souhaite afficher le détails d'une tâche !

## Page de détails d'une tâche
A ce stade, on constate bien dans la console qu'en clickant sur des liens de détails de tâche, le routeur s'en apperçoit mais nous n'avons rien à afficher, il faut donc créer un comportement qui soit capable de **remplacer la liste par le détail**.

Pour ça il nous faudra :
1. Un appel HTTP qui soit capable de récupérer une tâche donnée ;
2. Une fonction Javascript qui puisse projeter à l'écran les données obtenues ;

Commençons par l'appel HTTP :

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

Créons maintenant la fonction qui permettra d'afficher au sein de la balise `<main>` le détails d'une tâche :

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

A ce stade, notre routeur fonctionne comme on le souhaitait :
* Il écoute les changements d'URLs dans le navigateur
* Il analye l'URL et appelle le comportement correspondant
* Les comportements affichent les contenus au sein de la balise `<main>` qui devient ce qu'on appelle un **outlet** (un élément qui affiche le contenu correspondant à l'URL)

# Ce que vous avez appris :
* Notion de routing : l'association d'URLs et de comportements ;
* Agir dynamiquement sur l'URL dans le navigateur grâce à `window.history.pushState()` ;
* Réagir à un changement de l'historique de navigation en écoutant l'événement `popstate` ;
* Dispatcher par vous même un événément `popstate` ;
* Notion d' **outlet** : le container dans lequel le routeur affichera le contenu correspondant à une URL ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Tester son code avec Jest](tests.md)
