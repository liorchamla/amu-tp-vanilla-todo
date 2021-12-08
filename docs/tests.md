# Tester son code pour éviter les régressions

## Installation de Jest

`npm install --save-dev jest babel-jest @babel/core @babel/preset-env @types/jest`

```js
// babel.config.js
module.exports = {
  presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
};
```

```json
"scripts": {
    "test": "jest",
    "dev": "webpack --mode development",
    "serve": "live-server --entry-file=./index.html",
    "watch": "webpack --mode development --watch"
  },
```

## Premier test

```js
// tests/app.test.js
// test("it should work fine", () => {})
it("should work fine", () => {
    expect(true).toBe(true);
})
```

## Aller plus loin et tester notre application
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

```js
// tests/app.test.js

import { loadTodoFromApi } from "../src/todo.service.js";
import { displayTodos } from "../src/app.js";

jest.mock("../src/todo.service");

test("it displays todo items from API", async () => {
  document.body.innerHTML = `
    <main></main>
  `;

  loadTodoFromApi.mockResolvedValue([
    { id: 1, text: "MOCK_TODO", done: false },
    { id: 2, text: "MOCK_TODO_2", done: true },
  ]);

  await displayTodos();

  expect(document.querySelector("main ul")).not.toBeNull();
  expect(document.querySelectorAll("ul li").length).toBe(2);
  expect(document.querySelector("ul li").textContent).toContain("MOCK_TODO");
  expect(document.querySelector("ul li:nth-child(2)").textContent).toContain("MOCK_TODO_2");
});
```