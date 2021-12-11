# Appels HTTP vers une API REST

Pour l'instant, les données de notre applications sont **particulièrement éphémères** ! En effet, à chaque réactualisation du navigateur, les tâches reviennent au départ. Il existe plusieurs possibilités pour pallier à ce problème :

1. Le stockage local : on peut tout à fait décider de stocker les données de notre application sur le navigateur via le *LocalStorage* (dont la durée de vie est illimitée sauf intervention de l'utilisateur) ou le *SessionStorage* (dont la durée de vie est relativement courte) ;
2. Le stockage distant : on peut décider au contraire de stocker les données à distance, de telle sorte que la durée de vie du stockage soit réellement illimitée et **surtout que les données restent accessibles y compris sur d'autres navigateurs** ;


**Notez que Javascript n'a aucun moyen de se connecter à une base de données, qu'elle soit distante ou locale !**

Par contre, ce que Javascript sait très bien faire, c'est appeler un serveur web distant via des requêtes HTTP.

Il faut donc créer une application web backend qui aura accès à la base de données, et que l'on pourra appeler depuis notre javascript via des requêtes HTTP (le concept même d'API ;-)).

## Sommaire :
* [But de l'exercice :](#but-de-lexercice-)
* [Créer un projet sur Supabase :](#créer-un-projet-sur-supabase-)
* [Comprendre comment fonctionne l'API de Supabase](#comprendre-comment-fonctionne-lapi-de-supabase)
* [Afficher les tâches (Requêtes HTTP et Promesses)](#afficher-les-tâches--requêtes-http-et-promesses-)
    + [Système de Promesses](#système-de-promesses)
    + [Enchaînement de comportements](#enchaînement-de-comportements)
    + [La fonction fetch() de Javascript](#la-fonction-fetch-)
* [Ajouter une tâche dans la base de données](#ajouter-une-tâche-dans-la-base-de-données)
* [Passer les éléments à "fait" ou "pas fait"](#passer-les-éléments-à-fait-ou-pas-fait)
* [Ce que vous avez appris](#ce-que-vous-avez-appris-)

## But de l'exercice :
Nous allons créer une base de données PostgreSQL sur un serveur distant avec une table qui permettra de stocker les données de nos tâches. 

Pour ce faire nous utiliserons un service sur mesure : Supabase (une alternative aux services Firebase de Google).

Cette solution nous permet non seulement de créer des bases de données, mais intègre aussi par défaut une API qui permet d'intéroger ces bases de données via des requêtes HTTP.

Nous allons donc :
1. Créer un compte sur le service Supabase (nécessite un compte GitHub)
2. Créer un projet et une base de données adéquate
3. Connecter notre application front à Supabase afin de stocker les données à distance

## Créer un projet sur Supabase :
[Rendez vous sur le service Supabase](https://supabase.com) et cliquez sur ***"Start your project"*** afin de vous y connecter.

Une fois authentifié via GitHub, vous pourrez créer un projet en cliquant sur ***New Project***.

La création du projet peut durer quelques minutes et vous pourrez ensuite aller sur le ***Table Editor*** de votre base de données et cliquer sur ***New Table*** afin de créer une table que vous appellerez ***todos*** et dont les champs seront :

| Nom du champ | Type | Defaut |
| --- | --- | --- |
| id | Laissez tel quel
| created_at | Laissez tel quel
| text | varchar | null |
| done | bool | false |

Vous pouvez enfin insérer des lignes selon vos choix afin d'avoir déjà des données d'exemple.

## Comprendre comment fonctionne l'API de Supabase

Avant de continuer, vous devrez tout de même essayer de comprendre l'API en vous rendant dans la documentation (icône de fichier sur la gauche) et en sélectionnant ***Bash*** sur l'onglet de droite pour véritablement voir les requêtes HTTP qui sont acceptables ainsi que les options.

Une fois dans la documentation de l'API, vous pourrez même sélectionner le menu "***todos***" dans la barre de gauche et voir les requêtes HTTP spécifiques à la table *todos*.

## Afficher les tâches (Requêtes HTTP et Promesses)

Nous ne souhaitons plus utiliser le tableau `TODO_ITEMS`, nous pouvons donc supprimer toute référence à celui ci.

Vous allez avoir besoin de votre identifiant de projet ainsi que de la clé de sécurité. Vous trouverez ces informations dans la partie *Authentication* de la documentation de l'API.

A la place, nous comptons récupérer les tâches via une requête HTTP et les afficher :

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

Comme vous pouvez vous en douter, une requête HTTP n'est pas instantannée, et vous ne pouvez donc pas compter sur le résultat immédiatement dans votre code.

**C'est pourquoi Javascript nous fourni un système de code asynchrone : les Promesses**

### Système de Promesses
Une Promesse Javascript est un objet dont le but est de lancer un travail puis de vous permettre de décider quoi faire du résultat de ce travail.

On peut rattacher à une Promesse un comportement à exécuter en fin de travail grâce à la méthode `.then()` à qui nous devons confier une fonction.

La Promesse exécutera cette fonction en lui passant en paramètre le résultat de son travail.

### Enchaînement de comportements
Parfois, on a envie non pas d'attacher un unique comportement à la fin d'un Promesse, mais bien un enchaînement de comportements qui permettront de raffiner les données.

On peut donc enchaîner les `.then()` après une Promesse. Le premier de la chaîne recevra le résultat du travail de la Promesse, le deuxième récupèrera la valeur retournée par le `.then()` d'avant, et ainsi de suite.

### La fonction fetch()

Ici nous utilisons la fonction `fetch()` qui permet d'envoyer une requête HTTP en tâche de fond. La fonction renvoie donc une promesse qui contient ce comportement et qui nous permet de décider quoi faire lorsque la requête HTTP aura été exécutée et que la réponse HTTP du seveur sera disponible.

Vous remarquez que nous n'avons pas rattaché un unique comportement pour la fin de la Promesse `fetch()`. Nous avons :
1. Un premier `.then()` qui pourra travailler sur la réponse http et qui retournera le JSON contenu dans la réponse ;
2. Un deuxième `.then()` qui recevra la valeur retournée par le premier `.then()` (à savoir le JSON retourné par le serveur) et qui fera un nouveau traitement là dessus ;

## Ajouter une tâche dans la base de données
On a vu comment récupérer les tâches de la base de données et les projeter depuis notre Javascript jusqu'à l'interface HTML, il faut maintenant réussir aussi à ajouter une tâche dans la base de données afin que ses informations persistent.

On va donc pouvoir modifier la fonction qui gère la soumission du formulaire afin de ne plus donner un identifiant à la tâche (puisque la base de données s'en chargera automatiquement) mais aussi faire la requête HTTP avant que la tâche n'apparaisse dans l'interface :

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

Vous remarquez qu'on attend le retour de la requête HTTP avant de toucher à l'interface. On s'assure par là qu'on n'ajoute pas une tâche dans l'interface alors que par ailleurs on aurait eu un soucis avec l'API et la base de données (on appelle cela *l'approche pessimiste*).

## Passer les éléments à "fait" ou "pas fait"

On veut désormais aller un peu plus loin, de telle sorte qu'on permettre à l'utilisateur de décider si une tâche a été faite ou pas.

Pour cela on souhaite avoir deux choses :
1. Une fonction qui décidera quoi faire lorsqu'on click sur une checkbox ;
2. Le rattachement de cette fonction à chaque checkbox qui sera générée automatiquement ;

Commençons par créer une nouvelle fonction `onClickCheckbox` qui sera plus tard appelée à chaque click sur une checkbox, et qui sera chargée de faire la requête HTTP vers l'API afin de prévenir la base de données que le statut de la tâche a changé :

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

Ensuite, il nous faudra modifier la fonction `addTodo(item)` qui créé le visuel d'une tâche dans la liste afin qu'à chaque nouvelle tâche ajoutée dans l'interface, on rattache bien à la checkbox la fonction `onClickCheckbox` :

```js
 // src/app.js à la fin de la fonction addTodo()
 document
    // Nous sélectionnons la checkbox fraichement ajoutée au DOM
    .querySelector("input#todo-" + item.id)
    // Et nous lions la fonction onClickCheckbox au click 
    .addEventListener("click", onClickCheckbox);
```

Et voilà, normalement, vous devriez désormais pouvoir afficher les tâches de la base de données, y ajouter une tâche et même modifier les tâches afin de faire varier leur statut !

# Ce que vous avez appris :
* La notion d'asynchronicité en Javascript avec les Promesses, qui permettent de lancer un travail tout en précisant quoi faire lorsque ce travail sera terminé ;
* L'enchaînement de `.then()` à la suite d'une promesse afin d'enchaîner les travaux en raffinant les données ;
* Découverte du service Supabase qui vous permet en quelques secondes de créer une base de données distante accessible par une API (mais pas que ;-)) ;
* Découverte de la fonction `fetch()` permettant d'envoyer des requêtes HTTP ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Refactoring et modules](routing.md)
