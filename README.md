# Créer une application Vanilla JS (Todo List)

Dans ce TP, vous allez créer une Single Page Application (SPA) en Vanilla JS dans le but de découvrir différentes notions telles que les affichages dynamiques, la gestion des modules et du bundling, les appels HTTP et les promesses, le routage et bien sur les tests unitaires qui assureront la qualité et la non régression de vos codes.

## Mettre en place le projet en local
Clonez ce dépôt où bon vous semble :
```bash
git clone https://github.com/liorchamla/amu-tp-vanilla-todo.git
```
Ouvrez le dossier dans votre éditeur de code favoris !

## Travailler directement sur le navigateur
Plutôt que de cloner le dépôt, vous pouvez aussi travailler directement en ligne via Gitpod.

Pour cela il vous faudra *forker* le dépôt sur votre propre compte (bouton *Fork* en haut à droite de GitHub). Une fois sur la page du fork (la copie de ce dépôt sur votre propre compte), vous pourrez simplement ajouter `gitpod.io/#` juste devant l'URL du dépôt.

Par exemple, le lien suivant ouvrira **ce dépôt** dans Gitpod : https://gitpod.io/#https://github.com/liorchamla/amu-tp-vanilla-todo

**Attention : le travail ne sera pas sauvegardé d'une session de travail à l'autre, vous devrez systématiquement commit et push vos modifications si vous espérez les retrouver ensuite !**



## Attention au départ !

Une fois que le projet est mis en place, vous pouvez tout simplement suivre le TP étape par étape :

* [**Installation des outils de développement**](docs/setup.md)
  * [Initialiser la gestion de dépendances dans le projet](docs/setup.md#initialiser-la-gestion-de-dépendances-dans-le-projet)
  * [Installer le module bundler Webpack](docs/setup.md#installer-le-module-bundler-webpack)
  * [Lancer l'application dans le navigateur](docs/setup.md#lancer-l-application-dans-le-navigateur)
  * [Vérification de l'outillage et des liens](docs/setup.md#vérification-de-l-outillage-et-des-liens)
  * [Ce que vous avez appris ](docs/setup.md#ce-que-vous-avez-appris--)
* [**Afficher une liste dynamique**](docs/display-list.md)
  * [But de l'exercice](docs/display-list.md#but-de-l-exercice)
  * [Modéliser une liste de tâches dans le JS](docs/display-list.md#modéliser-une-liste-de-tâches-dans-le-js)
  * [Mettre en place un conteneur dans le HTML](docs/display-list.md#mettre-en-place-un-conteneur-dans-le-html)
  * [Afficher les items du tableau dans le HTML](docs/display-list.md#afficher-les-items-du-tableau-dans-le-html)
  * [Vérifier la bonne marche de notre application](docs/display-list.md#vérifier-la-bonne-marche-de-notre-application)
  * [L'avantage du watch avec Wepback](docs/display-list.md#l-avantage-du-watch-avec-wepback)
  * [Ce que vous avez appris](docs/display-list.md#ce-que-vous-avez-appris--)
* [**Ajout d'une nouvelle tâche**](docs/add-item.md)
    * [But de l'exercice](docs/add-item.md#but-de-l-exercice)
    * [Afficher un formulaire dans l'interface (HTML)](docs/add-item.md#afficher-un-formulaire-dans-l-interface--html-)
    * [Ecouter un événement du HTML dans le Javascript](docs/add-item.md#ecouter-un-événement-du-html-dans-le-javascript)
    * [Ce que vous avez appris](docs/add-item.md#ce-que-vous-avez-appris--)
* [**Appels HTTP et API REST**](docs/http.md)
    * [But de l'exercice :](docs/http.md#but-de-l-exercice--)
    * [Créer un projet sur Supabase :](docs/http.md#créer-un-projet-sur-supabase--)
    * [Comprendre comment fonctionne l'API de Supabase](docs/http.md#comprendre-comment-fonctionne-l-api-de-supabase)
    * [Afficher les tâches (Requêtes HTTP et Promesses)](docs/http.md#afficher-les-tâches--requêtes-http-et-promesses-)
        + [Système de Promesses](docs/http.md#système-de-promesses)
        + [Enchaînement de comportements](docs/http.md#enchaînement-de-comportements)
        + [La fonction fetch() de Javascript](docs/http.md#la-fonction-fetch---de-javascript)
    * [Ajouter une tâche dans la base de données](docs/http.md#ajouter-une-tâche-dans-la-base-de-données)
    * [Passer les éléments à "fait" ou "pas fait"](docs/http.md#passer-les-éléments-à--fait--ou--pas-fait-)
    * [Ce que vous avez appris](docs/http.md#ce-que-vous-avez-appris--)
* [**Routing et affichage dynamique**](docs/routing.md)
    * [But de l'exercice :](docs/routing.md#but-de-l-exercice--)
    * [Mise en place et premiers tests](docs/routing.md#mise-en-place-et-premiers-tests)
        + [Premier test : la page d'accueil](docs/routing.md#premier-test---la-page-d-accueil)
        + [Deuxième test : la future page de détails d'une tâche](docs/routing.md#deuxième-test---la-future-page-de-détails-d-une-tâche)
    * [Refactoring pour aller plus loin](docs/routing.md#refactoring-pour-aller-plus-loin)
    * [Gestion dynamique des URLs](docs/routing.md#gestion-dynamique-des-urls)
    * [Page de détails d'une tâche](docs/routing.md#page-de-détails-d-une-tâche)
    * [Ce que vous avez appris](docs/routing.md#ce-que-vous-avez-appris--)
* [**Tester son code avec Jest**](docs/tests.md)
    * [But de l'exercice](docs/tests.md#but-de-l-exercice)
    * [Installation de Jest](docs/tests.md#installation-de-jest)
    * [Premier test avec Jest](docs/tests.md#premier-test-avec-jest)
    * [Le problème du HTML dans nos tests](docs/tests.md#le-problème-du-html-dans-nos-tests)
    * [Aller plus loin et tester notre application](docs/tests.md#aller-plus-loin-et-tester-notre-application)
    * [Ce que vous avez appris](docs/tests.md#ce-que-vous-avez-appris--)

