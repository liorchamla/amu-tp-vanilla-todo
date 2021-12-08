
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