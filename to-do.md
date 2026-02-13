
#  Mini Project: Desktop To-Do List

**Goal:** Build a simple persistent To-Do app.


* Electron architecture
* IPC communication
* CRUD basics (Create, Read, Update, Delete)
* Local storage with `electron-store`

---

#  Step 1 — Create the Project

```bash
npx create-electron-app todo-app
cd todo-app
npm install electron-store@7
```

Test it:

```bash
npm start
```

Close the app.

---

#  Project Structure

Inside `src/`:

```
src/
├── main.js
├── preload.js
├── index.html
└── renderer.js
```

---

#  Step 2 — Setup Storage (Main Process)

Open:

```
src/main.js
```

Add at the top:

```js
const { app, BrowserWindow, ipcMain } = require('electron');
const path = require('path');
const Store = require('electron-store');

const store = new Store({
  name: 'todo-data'
});
```

---

## Add IPC Handlers

Below your window setup:

```js
ipcMain.handle('get-todos', () => {
  return store.get('todos', []);
});

ipcMain.handle('add-todo', (event, text) => {
  const todos = store.get('todos', []);

  const newTodo = {
    id: Date.now(),
    text,
    completed: false
  };

  todos.push(newTodo);
  store.set('todos', todos);

  return todos;
});

ipcMain.handle('toggle-todo', (event, id) => {
  let todos = store.get('todos', []);

  todos = todos.map(todo =>
    todo.id === id
      ? { ...todo, completed: !todo.completed }
      : todo
  );

  store.set('todos', todos);
  return todos;
});

ipcMain.handle('delete-todo', (event, id) => {
  let todos = store.get('todos', []);
  todos = todos.filter(todo => todo.id !== id);
  store.set('todos', todos);
  return todos;
});
```

Now we support:

* Add
* Toggle complete
* Delete

---

#  Step 3 — Update Preload Bridge

Open:

```
src/preload.js
```

Replace with:

```js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('api', {
  getTodos: () => ipcRenderer.invoke('get-todos'),
  addTodo: (text) => ipcRenderer.invoke('add-todo', text),
  toggleTodo: (id) => ipcRenderer.invoke('toggle-todo', id),
  deleteTodo: (id) => ipcRenderer.invoke('delete-todo', id)
});
```

---

#  Step 4 — Create UI

Open:

```
src/index.html
```

Replace with:

```html
<!DOCTYPE html>
<html>
<head>
  <title>To-Do App</title>
  <style>
    body {
      font-family: Arial;
      padding: 20px;
    }

    input {
      padding: 8px;
      width: 60%;
    }

    button {
      padding: 6px 10px;
      margin-left: 5px;
    }

    li {
      margin-top: 12px;
      list-style: none;
    }

    .completed {
      text-decoration: line-through;
      color: gray;
    }
  </style>
</head>
<body>
  <h1> To-Do List</h1>

  <input id="todoInput" placeholder="Add a task..." />
  <button id="addBtn">Add</button>

  <ul id="todoList"></ul>

  <script src="renderer.js"></script>
</body>
</html>
```

---

#  Step 5 — Renderer Logic

Create:

```
src/renderer.js
```

Add:

```js
const input = document.getElementById('todoInput');
const addBtn = document.getElementById('addBtn');
const list = document.getElementById('todoList');

async function loadTodos() {
  const todos = await window.api.getTodos();
  renderTodos(todos);
}

function renderTodos(todos) {
  list.innerHTML = '';

  todos.forEach(todo => {
    const li = document.createElement('li');

    const span = document.createElement('span');
    span.textContent = todo.text;

    if (todo.completed) {
      span.classList.add('completed');
    }

    const toggleBtn = document.createElement('button');
    toggleBtn.textContent = '✔';

    const deleteBtn = document.createElement('button');
    deleteBtn.textContent = 'Delete';

    toggleBtn.addEventListener('click', async () => {
      const updated = await window.api.toggleTodo(todo.id);
      renderTodos(updated);
    });

    deleteBtn.addEventListener('click', async () => {
      const updated = await window.api.deleteTodo(todo.id);
      renderTodos(updated);
    });

    li.appendChild(span);
    li.appendChild(toggleBtn);
    li.appendChild(deleteBtn);

    list.appendChild(li);
  });
}

addBtn.addEventListener('click', async () => {
  const text = input.value.trim();
  if (!text) return;

  const updated = await window.api.addTodo(text);
  renderTodos(updated);
  input.value = '';
});

loadTodos();
```

---

#  Step 6 — Run the App

```bash
npm start
```

Test:

* Add task
* Mark complete
* Delete task
* Close app
* Reopen app

Tasks are still saved 

---

# Concept

| Feature         | Concept                       |
| --------------- | ----------------------------- |
| Add task        | Saving data                   |
| Toggle complete | Updating object state         |
| Delete task     | Array filtering               |
| Persistence     | Local JSON storage            |
| IPC             | Main ↔ Renderer communication |

---
