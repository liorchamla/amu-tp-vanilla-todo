# Installation de dépendances de développement

Dans la panoplie des outils Javascript modernes, le chef d'orchestre est le Node Package Manager (NPM). Il existe des alternatives (dont Yarn est la plus connue) mais NPM reste le gestionnaire de dépendances le plus utilisé. 

Comme pour tous les autres langages de programmation, le gestionnaire de dépendances (package manager) a pour mission de vous aider à installer, mettre à jour, supprimer et surtout lister les dépendances (librairies) dont votre projet est tributaire.

## Sommaire :
  * [Initialiser la gestion de dépendances dans le projet](#initialiser-la-gestion-de-dépendances-dans-le-projet)
  * [Installer le module bundler Webpack](#installer-le-module-bundler-webpack)
  * [Lancer l'application dans le navigateur](#lancer-l-application-dans-le-navigateur)
  * [Vérification de l'outillage et des liens](#vérification-de-l-outillage-et-des-liens)
  * [Ce que vous avez appris ](#ce-que-vous-avez-appris--)

## Initialiser la gestion de dépendances dans le projet
Pour pouvoir bénéficier des forces de NPM dans votre projet, vous devez tout d'abord créer le manifeste qui permettra de le configurer : le fichier package.json

Vous pouvez faire celà très rapidement en lançant la commande `npm init -y`

Vous verrez alors apparaitre un fichier package.json qui contiendra des informations basiques sur votre projet mais qui deviendra encore plus précis au fur et à mesure que l'on va ajouter des dépendances et mêmes des tâches automatisées (scripts).

Sachez aussi que chaque package (librairie ou outil) que vous allez installer verra ses fichiers être téléchargés directement dans un dossier node_modules dont vous n'avez pas à vous préoccuper. C'est de la prérogative de NPM que de gérer ce dossier.


## Installer le module bundler Webpack 
Dans le monde du front, la rapidité d'affichage d'une page est une clé de la réussite. Divers patterns ont été imaginés pour réussir à obtenir une meilleure rapidité dans le chargement d'une page.

L'un d'eux est le "bundling" qui consiste à :
1. Permettre aux developpeuses et développeurs de coder dans différents fichiers afin d'organiser le code au mieux ;
2. Ne livrer malgré tout qu'un seul fichier JS au navigateur lors de la visite de la page (et donc améliorer temps de chargement en réduisant le nombre de requêtes HTTP concernant la récupération du Javascript) ;

L'outil le plus connu (et d'aucuns diraient le plus puissant et configurable) pour arriver à ce résultat est Webpack. Nous allons donc l'installer grâce à la commande suivante :

`npm install --save-dev webpack webpack-cli`

Une fois l'outil installé, il faut absolument le configurer pour lui expliquer où sont les fichiers qui contiennent le code de base, et où doit-il écrire le fichier unique final qui sera ensuite livré au navigateur :

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

Il faut ensuite permettre aux développeurs de lancer Webpack lorsqu'ils le souhaitent, pour cela nous allons configurer NPM et ajouter un script. Ce que nous disons ci-dessous, c'est que lorsqu'on demandera à NPM le script **dev**, celui ci lancera la commande `webpack --mode development`

```json
// package.json

"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development"
},
```

Pour tester cette nouvelle tâche, nous lançons simplement la commande suivante : `npm run dev` et webpack devrait générer le fichier *dist/app.js*

## Lancer l'application dans le navigateur

Ayant installé WebPack, on peut désormais coder dans différents fichiers et organiser nos sources comme nous le souhaitons tout en ayant la possibilité de ne livrer qu'un seul fichier au navigateur (le fameux *dist/app.js*).

Pour continuer à travailler, on aimerait désormais afficher le site au sein du navigateur afin de vérifier notre avancée de façon concrète et visuelle. Pour cela nous avons besoin d'un serveur web léger et rapide, qu'on peut obtenir facilement grâce au package **live-server**. Son but est de lancer facilement un serveur web qui servira notre application en local.

Lançons donc la commande :

`npm install --save-dev live-server`

Et encore une fois, afin de permettre aux développeurs de lancer facilement ce serveur web, nous allons configurer une nouvelle tâche pour NPM :

```json
// package.json

"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack --mode development",
    "serve": "live-server --entry-file=./index.html"
},
```

Ce que nous décrivons ici, c'est qu'en lançant la commande `npm run serve` nous allons lancer un serveur web local qui servira toujours le fichier *index.html* quelque soit la requête HTTP qu'il recevra.

**Attention :** comprenez bien que toutes les URLs que vous taperez sur cette application seront toujours redirigées vers index.html, ce qui ne serait pas le cas sur un serveur web classique tel que Apache ou NGinx, du moins pas sans configuration particulière ! Ce sera donc à prendre en compte lors du déploiement de votre application, si vous décidez par exemple de déployer sur un serveur Apache, il vous faudra configurer le même comportement.

## Vérification de l'outillage et des liens

Maintenant qu'on a mis en place les outils de base, nous allons vérifier que tout fonctionne.

Premièrement, ajoutez le code suivant dans le fichier *src/app.js*
```js
// src/app.js
console.log("Tout fonctionne");
```

Puis modifiez le fichier *index.html* afin d'ajouter à la toute fin du `<body>` la balise suivante, qui permettra de demander au navigateur de télécharger le fichier construit par webpack :

```html
<script src="dist/app.js"></script>
```

Notez que pour l'instant, le fichier dist/app.js ne contient absolument pas le code qu'on a ajouté à src/app.js, pour cela il faudrait appeler Webpack afin qu'il prenne en compte le changement : `npm run dev`

Vous pouvez maintenant aller sur votre navigateur et ouvrir la console de développement afin de vous assurer que vous obtenez bien le message **"Tout fonctionne"**.

# Ce que vous avez appris :
* Utiliser NPM afin de gérer les dépendances (librairies ou outils) nécessaires à votre projet ;
* Utiliser NPM afin de configurer des tâches que vous pourrez lancer ;
* Utiliser Webpack pour "bundler" vos fichiers JS en un fichier unique (et bien plus) ;
* Utiliser live-server afin de créer un petit serveur web et d'afficher votre application dans le navigateur ;

[Revenir au sommaire](../README.md) ou [Passer à la suite : Afficher la liste des tâches](display-list.md)
