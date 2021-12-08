# Tester son code pour éviter les régressions
Notre application est désormais terminée : elle affiche une liste des tâches sauvegardées à distance et permet aussi de les modifier et d'en ajouter de nouvelles. Elle permet enfin de voir le détail d'une tâche donnée.

On veut désormais s'assurer que de futures modifications de notre code n'aille pas à l'encontre de ce fonctionnement et pour ce faire nous allons écrire des .. Tests !

Comme tout langage de programmation, Javascript possède des utilitaires et frameworks de tests qui vont permettre d'écrire des tests unitaires afin de valider le bon fonctionnement du code.

## Sommaire :

* [But de l'exercice](#but-de-l-exercice)
* [Installation de Jest](#installation-de-jest)
* [Premier test avec Jest](#premier-test-avec-jest)
* [Le problème du HTML dans nos tests](#le-problème-du-html-dans-nos-tests)
* [Aller plus loin et tester notre application](#aller-plus-loin-et-tester-notre-application)
* [Ce que vous avez appris](#ce-que-vous-avez-appris--)

## But de l'exercice 

Nous allons découvrir ici les tests unitaires en Javascript avec le framework de test **Jest**.

**Ici, les notions importantes sur lesquelles se concentrer sont l'écriture de tests, la simulation des intéractions avec le DOM ainsi que l'utilisation de fonctions simulées (mocks)**

## Installation de Jest
Encore une fois, NPM va être notre allié le plus précieux ! Il va nous permettre d'installer un nouvel outil ainsi que des plugins nécessaires pour écrire nos tests unitaires !

```bash
npm install --save-dev jest babel-jest @babel/core @babel/preset-env @types/jest
```

Ce qu'on installe ici c'est :
1. Le framework de tests Jest ;
2. Un plugin babel-jest qui permettra à Jest de demander à Babel de compiler le code JS avant de lancer les tests. En effet, Jest ne supporte pas la gestion des modules telle que nous les avons utilisé ;
3. L'outil Babel lui-même du coup ;
4. Un plugin pour Babel qui lui permet de savoir dans quel format nous souhaitons traduire notre code JS ;
5. Enfin, une librairie de types spécifique à Jest qui permettra simplement à notre éditeur de code de connaître les fonctions et méthodes disponibles dans Jest afin d'obtenir une belle autocomplétion :-) ;

Comme nous devons utiliser Babel (du fait que Jest ne gère pas les modules tels que nous les utilisons), il nous faut configurer babel :

```js
// babel.config.js
module.exports = {
  presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
};
```

Et bien sur, il nous faut ajouter une tâche qu'on pourra appeler depuis NPM :

```json
//package.json

"scripts": {
    "test": "jest",
    "dev": "webpack --mode development",
    "serve": "live-server --entry-file=./index.html",
    "watch": "webpack --mode development --watch"
  },
```

En théorie, vous devriez initialiser Jest afin d'obtenir un fichier de configuration. Ce fichier *jest.config.js* vous a déjà été fourni dans le dépôt avec quelques modifications.

## Premier test avec Jest

Ecrivons un premier test juste pour s'assurer que le framework fonctionne bien et que notre configuration est valide :

```js
// tests/app.test.js

// test("it should work fine", () => {})
it("should work fine", () => {
    expect(true).toBe(true);
})
```

Lançons notre tâche de test pour voir la sortie de Jest : `npm test`

Si tout va bien, on peut continuer et écrire un véritable test !

## Le problème du HTML dans nos tests 
Comme vous le constatez, Jest ne se lance pas dans notre navigateur mais dans notre terminal. Dans ce contexte, comment faire pour tester des intéractions ou des projections dans le DOM ?

Pour répondre à ce challenge, Jest intègre une librairie tierce appelée **JSDOM** qui permet de simuler le DOM d'un navigateur en Javascript et donc utilisable au sein de nos tests !

## Aller plus loin et tester notre application
Lorsque l'on test du HTML et de l'Asynchrone, il faut être capable d'attendre que les tâches asynchrones soient faites, surtout si le résultat de ces tâches a un impact sur le HTML.

Pour cela, on peut simuler une attente via une fonction toute simple (mais sommes toute assez dégoutante :p), créez un fichier *tests/utils.js* :

```js
// tests/utils.js

/**
 * Permet de simuler une attente dans nos tests
 * @returns Promise<null>
 */
export const tick = () => {
    return new Promise(resolve => {
        setTimeout(() => resolve());
    });
}
```

Maintenant qu'on a cette fonction, passons au véritable test d'interface : nous voulons tester le fait que la fonction `displayTodoList()` affichera bien des tâches dans l'interface.

Comme il n'est pas question d'appeler Supabase à chaque fois qu'on joue notre test, nous allons tout simplement **mocker la fonction `loadTodoItemsFromApi()`** de telle sorte qu'elle ne soit pas réellement appelée.

Le fait de mocker une fonction permet non seulement de ne pas l'appeler pour de bon, mais en plus de piloter sa valeur de retour, ce qui est fort pratique quand on écrit des tests :

```js
// tests/app.test.js

import { displayTodoList } from "../src/ui";
import { tick } from "./utils";

// On explique à Jest que toutes les fonctions du fichier src/api.js seront mockées :
// Jest va donc créer de fausses fonctions qui vont remplacer les vraies.
// Il nous donnera en plus la possibilité de contrôler les valeurs de retour, et nous permettra aussi de contrôler le fait que telle ou telle fonction a bien été appelée
jest.mock("../src/api");

// Testons que l'application affichera bien les tâches en provenance de l'API
it("displays todo items from API", async () => {

    // Simulons un document HTML qui aurait un élément <main>
    document.body.innerHTML = `
        <main></main>
      `;

    // On explique ici à la fausse fonction loadTodoItemsFromApi que quand elle sera appelée
    // Elle devra retourner un tableau contenant 2 tâches
    loadTodoItemsFromApi.mockResolvedValue([
        { id: 1, text: "MOCK_TODO", done: false },
        { id: 2, text: "MOCK_TODO_2", done: true },
    ]);

    // Appelons displayTodoList() afin de constater ensuite du résultat
    displayTodoList();

    // Comme displayTodoList() fait un travail asynchrone (l'appel à l'API, même si en réalité l'API ne sera pas appelée, ça reste une Promesse et donc un travail asynchrone)
    // On attend la fonction tick() qui permet de simuler une petite attente
    await tick();

    // Après l'attente, le HTML doit avoir été modifié par la fonction displayTodoList() :
    // On s'attend à ce que <main> contienne désormais un <ul>
    expect(document.querySelector("main ul")).not.toBeNull();
    // On s'attend à ce que le <ul> contienne 2 <li>
    expect(document.querySelectorAll("ul li").length).toBe(2);
    // On s'attend à ce que le premier <li> contienne le texte MOCK_TODO
    expect(document.querySelector("ul li").textContent).toContain("MOCK_TODO");
    // On s'attend à ce que le deuxième <li> contienne le texte MOCK_TODO_2
    expect(document.querySelector("ul li:nth-child(2)").textContent).toContain("MOCK_TODO_2");
});
```

Et voilà ! Vous avez testé la fonction `displayTodoList()`, ce qui vous permet désormais d'en modifier des détails d'implémentation tout en pouvant systématiquement et automatiquement vérifier que vous n'avez pas changer le comportement général (et donc que votre client restera heureux de sa fonctionnalité :-)).

Vous pouvez bien sur aller encore plus loin avec un deuxième test :

```js
// tests/app.test.js

import { loadTodoItemsFromApi, saveTodoItemToApi } from "../src/api";
import { displayTodoList } from "../src/ui";
import { tick } from "./utils";

jest.mock("../src/api");

it("displays todo items from API", async () => {
  // ...
});

// Testons qu'on peut ajouter une tâche avec le formulaire
it("should add a todo item", async () => {
    // Imaginons que l'API ne renvoie aucune tâche déjà enregistrée
    loadTodoItemsFromApi.mockResolvedValue([]);
    // Et partons du principe que lorsqu'on va sauvegarder une tâche dans Supabase, celle-ci va nous retourner cette tâche tel que :
    saveTodoItemToApi.mockResolvedValue([
        { id: 1, text: "MOCK_TASK", done: false },
    ]);

    // On simule un document HTML qui contient un élément <main>
    document.body.innerHTML = `
        <main></main>
      `;

    // On appelle displayTodoList() dont le but est d'afficher la liste ET le formulaire
    displayTodoList();

    // On peut désormais manipuler le formulaire
    // Donnons la valeur MOCK_TASK a notre <input />
    document.querySelector('input[type=text]').value = "MOCK_TASK";
    // Puis nous soumettons le formulaire
    document.querySelector('form').submit();

    // Normalement, cela devrait déclencher la fonction onSubmitForm() et donc l'appel HTTP à Supabase
    // Puis l'ajout d'une nouvelle tâche dans le <ul>
    // Comme on a de l'asynchrone, on simule une petite attente :
    await tick();

    // Et on s'attend à trouver un <li> dans le <ul> (la tâche qu'on vient d'ajouter)
    expect(document.querySelectorAll('ul li').length).toBe(1);
    // Vous pourriez ajouter d'autres vérifications pour vous assurer que ça fonctionne correctement :)
});
```

Vous pourriez même avoir envie de tester votre routeur ! N'hésitez pas, cela pourrait donner quelque chose comme :

```js
// tests/routing.test.js

import { applyRouting } from "../src/routing";
import { displayTodoDetails, displayTodoList } from "../src/ui";

// Nous ne voulons pas que les fonctions qui se trouvent dans le module src/ui.js soient
// réellement appelées, nous souhaitons les mocker et les contrôler nous même.
jest.mock("../src/ui");

// Testons que l'URL "/" appellera bien le comportement adéquat
it("should display todo list", async () => {
    // On appelle applyRouting avec l'URL "/"
    applyRouting("/");

    // On s'attend à ce que la fonction displayTodoList ait été appelée
    expect(displayTodoList).toBeCalled();
});

// Testons que ces différentes URL afficheront bien le contenu adéquat
it("should display details", () => {
    // Si on appelle 3 fois notre routeur avec 3 URLs différentes
    // mais correspondantes à la page de détails d'une tâche
    applyRouting("/1/details");
    applyRouting("/12/details");
    applyRouting("/440/details");

    // On s'attend à ce que chaque appel ait déclenché la fonction displayTodoDetails !
    expect(displayTodoDetails).toBeCalledTimes(3);
});
```

Voilà ! On a testé deux fonctionnalités phares de notre application ainsi que le routeur !

A vous d'explorer le monde des tests unitaires avec Jest pour en apprendre plus !

N'hésitez pas à aller plus loin pour découvrir les tests E2E (End To End) avec Cypress ou Protractor ;-)

# Ce que vous avez appris :
* Installer Jest et le configurer afin qu'il tienne compte du système de modules ;
* Créer un test unitaire (y compris impliquant des intéractions avec le DOM) ;
* Créer des mocks de fonctions afin de les piloter et d'empêcher de réels appels ;

