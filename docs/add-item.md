# Ajout d'une nouvelle tâche

Maintenant qu'on sait afficher une liste dynamique dont **les données sont projetées du javascript vers le html**, il faut permettre à l'utilisateur d'ajouter une tâche depuis l'interface.

## Sommaire :
* [But de l'exercice](#but-de-l-exercice)
* [Afficher un formulaire dans l'interface (HTML)](#afficher-un-formulaire-dans-l-interface--html-)
* [Ecouter un événement du HTML dans le Javascript](#ecouter-un-événement-du-html-dans-le-javascript)
* [Ce que vous avez appris](#ce-que-vous-avez-appris--)

## But de l'exercice 
Retenez toujours le flux de données dans une application front (expliqué dans le chapitre précédent). Le Javascript gère les données et les projète dans le HTML, le HTML fait appel à des traitements disponibles dans le Javascript. 

1. Le HTML devra désormais afficher un formulaire permettant à l'utilisateur d'entrer le texte d'une nouvelle tâche
2. Le Javascript devra observer le formulaire pour réagir à sa soumission et projeter une nouvelle tâche dans le visuel

**Ici, on s'attachera à comprendre comment réagir à un événement qui a lieu sur l'interface depuis le code Javascript**

## Afficher un formulaire dans l'interface (HTML)

Modifions le fichier *index.html* afin de disposer un formulaire :

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

Nous avons déjà une fonction `addTodo(item)` dans le Javascript qui est responsable de projeter une tâche dans la liste, ce qu'il nous faut maintenant, c'est intercepter la soumission du formulaire afin de gérer l'ajout d'une nouvelle tâche :

## Ecouter un événement du HTML dans le Javascript

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

Faites un essai dans votre navigateur, en théorie, tout devrait fonctionner :-)

# Ce que vous avez appris :
* Ecouter un événement qui a lieu dans l'interface afin de l'intercepter au sein du Javascript et de lancer un traitement précis ;
* Annuler le comportement par défaut d'un événement donné grâce à la méthode `preventDefault()` de l'événement ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Appels HTTP et API REST](http.md)
