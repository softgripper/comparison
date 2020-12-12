# Svelte vs React

This is a comparison for the implementation of a Todo List app in Svelte and React.

The Todo List app is multiple components and shared state.

Everything is typescript, and there are no "type" warnings.

Hotmodule reload is included.

# Install

## Svelte

Svelte install was using the svelte@next version of svelte kit. This is very similar to standard svelte with typescript - but comes with routing and SSR out of the box.

Prefixing a filename with a '$' is the only implementation difference.

```bash
npm init svelte@next
```

There is a current bug in the svelte@next package that stops our pages from loading.

```
Error: Failed to load /_app/assets/generated/root.js: NOT_FOUND
```

Which can be rectified by adding the following to your package.json

```json
{
    ...,
    "resolutions": {
        "snowpack": "2.17.0"
    }
}
```

Finally,

```bash
npm install
npm dev
```

Server should be running at http://localhost:3000/

When editing files, the components should update.

## React

```bash
npx create-react-app react-todos --template typescript
```

There is a bug somewhere (either VSCode or React) where the typescript versions do not match between the app and the editor.

```
Cannot use JSX unless the '--jsx' flag is provided
```

You have to set VSCode typescript to "Use workspace Version".

There are a lot of mentions of this issue for years.

```bash
npm install
npm start
```

Server should be running at http://localhost:3000/

When editing files, the components should update.

### Winner

Svelte

- package.json has less cruft.
- only has development dependencies - no runtime dependencies.
- supports scss out of the box.

# Basic Tooling

Both react and svelte dev tools allow you to see the components in the DOM, and check the values of their state. The svelte dev tools are noticably more responsive for this. React tools have a bit more polish.

### Winner

Tie

# Component State

## React

React uses "useState" for components - and you pass this state around to share it (parent to child), and call methods to update that state.

### App - React

```typescript
import React, { useState } from 'react';

import { TodoList } from '../components/TodoList';
import { AddTodoForm } from '../components/AddTodoForm';
import { generateTestTodos, ToggleTodo, Todo, AddTodo } from '../types';

import './App.css';

const initialTodos = generateTestTodos();

function App() {
  const [todos, setTodos] = useState(initialTodos);

  const toggleTodo: ToggleTodo = (selectedTodo: Todo) => {
    const newTodos = todos.map((todo) => {
      if (todo === selectedTodo) {
        return {
          ...todo,
          complete: !todo.complete,
        };
      }
      return todo;
    });
    setTodos(newTodos);
  };

  const addTodo: AddTodo = (text) => {
    const newTodo = { text, complete: false };
    setTodos([...todos, newTodo]);
  };

  return (
    <main>
      <h1>react-todos</h1>
      <TodoList todos={todos} toggleTodo={toggleTodo} />
      <AddTodoForm addTodo={addTodo} />
    </main>
  );
}

export default App;
```

We define the state at the app level. Both our children (TodoList and AddTodoForm) need to share the scope, and do so via handlers or state props.

For our handlers to be strongly typed, we need to create an interfaces.

```js
export type ToggleTodo = (selectedTodo: Todo) => void;
```

and reference these wherever the handlers are used.

### App - Svelte

```html
<script>
  import AddTodoForm from '$components/AddTodoForm.svelte';
  import TodoList from '$components/TodoList.svelte';
</script>

<style lang="scss">
  ...
</style>

<main>
  <h1>svelte-todos</h1>
  <TodoList />
  <AddTodoForm />
</main>
```

There is no need to explicitly share state, so components in Svelte are much simpler than react, can be typed and involve a lot less boilerplate. There is no need for handler interfaces.

### TodoList.tsx

```javascript
import React from 'react';
import { Todo, ToggleTodo } from '../types';

import './TodoList.css';

interface Props {
  todos: Todo[];
  toggleTodo: ToggleTodo;
}

export const TodoList: React.FC<Props> = ({ todos, toggleTodo }) => {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.text}>
          <label className={`${todo.complete ? 'complete' : ''}`}>
            <input
              type='checkbox'
              checked={todo.complete}
              onChange={() => {
                toggleTodo(todo);
              }}
            />
            {todo.text}
          </label>
        </li>
      ))}
    </ul>
  );
};
```

Here we have to create a Props interface so we can get strong typing for the component. We also tie the toggleTodo handler to the onChange event for the input.

We also have to key our list items or we get console warnings, and react can't figure out which items are unique.

### TodoList.svelte

```html
<script lang="ts">
  import todoListStore from '../stores/todoListStore';
</script>

<style lang="scss">
  ...
</style>

<ul>
  {#each $todoListStore as todo}
  <li>
    <label class:complete={todo.complete}>
      <input type="checkbox" bind:checked={todo.complete} />{todo.text}
    </label>
  </li>
  {/each}
</ul>
```

Using the $syntax to refer to the store - we can bind components and values directly to the store value. When the store is updated, the components will automatically reflect this. It is using a subscriber pattern, and making that easily accessible with the $ syntax. When the store value changes, the subscribers are updated.

We bind the checked attribute of our checkbox to the todo.complete property of our todo. This gives us 2 way binding - from component to store, from store to component.

There is no need to key our list items.

### AddTodoForm.tsx

```javascript
import React, { useState } from 'react';
import { AddTodo } from '../types';

interface Props {
  addTodo: AddTodo;
}

export const AddTodoForm: React.FC<Props> = ({ addTodo }) => {
  const [text, setText] = useState('');

  return (
    <form>
      <input
        type='text'
        value={text}
        onChange={(e) => {
          setText(e.target.value);
        }}
      />
      <button
        type='submit'
        onClick={(e) => {
          e.preventDefault();
          addTodo(text);
          setText('');
        }}
      >
        Add Todo
      </button>
    </form>
  );
};
```

Again, we need to define an interface for our props and setup the state.

We set the value of the input to our text state, and when it changes, call to update our state.

When the button is pressed, we call the addTodo handler which was passed down from App.tsx

### AddTodoForm.svelte

```html
<script lang="ts">
  import todoListStore from "../stores/todoListStore";

  let text = "";
</script>

<input type="text" bind:value={text} /><button
  on:click|preventDefault={() => {
    $todoListStore = [...$todoListStore, { text, complete: false }];
    text = '';
  }}>
  Add Todo
</button>
```

This is too simple.

```js
let text = '';
```

This is our component state. It's just a local variable.

Also, we do not need a surrounding parent tag (**form** was used in React).

We bind the "value" property of the input to our text variable - which gives us 2 way binding.

The on:click handler appends our new todo to the store, then sets our text to ''.

Important node - Svelte's reactivity is triggered on reassign. You can do this a couple of ways

```js
function addNumber() {
  // this does not trigger update
  numbers.push(numbers.length + 1);
  // this triggers update
  numbers = numbers;
}

function addNumber() {
  // this triggers update
  numbers = [...numbers, numbers.length + 1];
}

function addNumber() {
  // this triggers update
  numbers[numbers.length] = numbers.length + 1;
}
```

A simple rule of thumb: the name of the updated variable must appear on the left hand side of the assignment.

todoListStore.ts

```js
import { writable } from 'svelte/store';
import type { Todo } from '../types';

const initialTodos: generateTestTodos();

export default writable(initialTodos);
```

This is a strongly typed store Todo[] store. You can easily implement your own to use localStorage or other.

### Winner

Svelte

- Strongly typed without needing extra interfaces everywhere
- State easily shared without needing the "passing down" of state or handlers.
- Way less code. You actually only write what you're doing - the boilerplate is gone.

# Styling

CSS, with all it's leaps and bounds is still very hard to maintain, or expand, and there can be unintended side effects of changing properties, removing classes etc. It is exponentially worse as time goes on, or more people become involved.

React allows you to include a CSS file and style your component. It still suffers from CSS entropy, and unintended side effects and bloat.

Svelte allows you to define your CSS (or scss) inside your component.
You can include some defaults in base (via global), and define your own rules for the component.

The CSS is also scoped to the component

```html
<style lang="scss">
  .complete {
    text-decoration: line-through;
  }

  ul {
    list-style: none;
  }
</style>

<ul>
  {#each $todoListStore as todo}
  <li>
    <label class:complete={todo.complete}>
      <input type="checkbox" bind:checked={todo.complete} />{todo.text}
    </label>
  </li>
  {/each}
</ul>
```

The **.complete** and **ul** will only be effected within this component.

Also, in the label component, the **complete** class will only get added if **todo.complete** evaulates to true.

The VSCode highlights which classes are not being used, and also svelte will cull them when it's compiled.

This means no more

```css
div#outer > div.containerItem.selected,
div#anotherComponent > div.photoItem.selected {
  font-weight: bold !important;
  margin-left: -200px;
}
```

and no more of negative margin, or padding, or positioning effecting other components.

### Winner

Svelte - by a mile

- out of the box scss
- less css
- less clobbering of other css (isolated)
- less time debugging or testing css
- easier to understand css
- easier to fix css
- smaller css for the client

This is an amazing feature.

# Testing

## React
Testing is achieved with jest, and @testing-library/react

I also tried to test with Enzyme.  However there was a rabbit hole of dependencies to install, and configuration.  It probably works reasonably well - and allows you to force state on components.

Because our code had to pass down handlers for adding new todos, and sharing state between items we need to test some things at the parent level.  In this case, this is App.spec.tsx

Not ideal.

We may be able to get around this using Enzyme, but  we're still left with injecting handlers or state.  We could go down the "context" path also.

For setup of data, I think Enzyme would be the way to go.

Below is a test of adding a new todo.

### App.spec.ts

``` ts
import React from 'react';
import { fireEvent, render, screen } from '@testing-library/react';
import App from './App';

test('adds a new todo', () => {
  const app = render(<App />);

  const input = app.getByRole('textbox') as HTMLInputElement;
  const button = app.getByText('Add Todo');

  const testText = 'New Todo';

  fireEvent.change(input, { target: { value: testText } });
  fireEvent.click(button);

  // with enzyme we could probably query the state directly
  const checkbox = app.getByLabelText(testText) as HTMLInputElement;
  expect(checkbox.checked).toBeFalsy();
});

```

### TodoList.spec.ts
```js
import { fireEvent, render, screen } from '@testing-library/react';
import React from 'react';
import { Todo } from '../types';
import { TodoList } from './TodoList';

test('renders the todo list', () => {
  render(<TodoList todos={[]} toggleTodo={() => {}} />);
});

test('press toggle item', () => {
  const todos: Todo[] = [
    {
      text: 'test',
      complete: false,
    },
  ];

  let toggleClicked = false;

  const result = render(
    <TodoList
      todos={todos}
      toggleTodo={() => {
        toggleClicked = true;
      }}
    />
  );

  const checkbox = result.getByLabelText(/test/i) as HTMLInputElement;
  fireEvent.click(checkbox);

  // alternately use mocha service
  expect(toggleClicked).toBeTruthy();
});
```


## Svelte

The svelte test is directly next to the component being tested.

### AddTodoForm.spec.ts

```js
import { render, fireEvent } from '@testing-library/svelte';
import userEvent from '@testing-library/user-event';
import todoListStore from '../stores/todoListStore';
import AddTodoForm from './AddTodoForm.svelte';
import {get} from 'svelte/store'

// initialize the store with data for the test
beforeEach(() => todoListStore.set([]));

test('it should add a todo', async () => {
  const form = render(AddTodoForm);

  const input = form.getByRole('textbox') as HTMLInputElement;
  const button = form.getByText('Add Todo');

  // we can check the store directly
  expect(get(todoListStore).length).toBe(0);

  const testText = 'New Todo';
  userEvent.type(input, testText);
  fireEvent.click(button);
 
  expect(get(todoListStore)[0].text).toBe(testText);
});
```

### TodoList.spec.ts
```js
import { render, fireEvent } from '@testing-library/svelte';
import todoListStore from '../stores/todoListStore';
import TodoList from './TodoList.svelte';
import { get } from 'svelte/store';

test('it should render the todo list and toggle', async () => {
  const list = render(TodoList);

  todoListStore.set([
    {
      text: 'Test Store Todo',
      complete: false,
    },
  ]);

  expect(get(todoListStore).length).toBe(1);

  const checkbox = await list.findByLabelText('Test Store Todo');
  fireEvent.click(checkbox);

  expect(get(todoListStore)[0].complete).toBeTruthy();
});
```
### Winner

Svelte

* Tests live next to the functionality being tested (eg the components)
* Don't have to mock interfaces to instantiate various types of tests.
* Tests run FAST 0.25s for two tests, vs 1.5s for three in React
* We can easily use the store without having to expose other frameworks, export inner methods etc.

Con - I do not have "debug" working in tests.  I suspect it's something to do with my jest setup.  It proved a non issue - the tests run fast, they are simple, and everything is strongly typed.  


# Files

## React

Taken from the best bits of 3 tutorials, and simplified with learnings from Svelte project.

| File                       | Lines | Size |
| -------------------------- | ----: | ---: |
| components/AddTodoForm.tsx |    33 |  593 |
| components/TodoList.tsx    |    31 |  653 |
| components/TodoList.css    |     8 |   75 |
| app/App.tsx                |    42 |  956 |
| app/App.css                |    31 |  466 |
| types.tsx                  |    26 |  442 |
| TOTAL                      |   171 | 3185 |

Test code not included in counts - it's large.

**Served to client browser in production: 142 kB**

## Svelte

Implementation based on react

| File                          | Lines | Size |
| ----------------------------- | ----: | ---: |
| components/AddTodoForm.svelte |    15 |  297 |
| components/TodoList.svelte    |    25 |  423 |
| routes/index.svelte           |    41 |  775 |
| stores/todoListStore.ts       |    20 |  335 |
| types.d.ts                    |     5 |   63 |
| TOTAL                         |   106 | 1893 |

Test code not included in counts.

**Served to client browser in production: 22.7kB**

# Conclusion

Svelte and React are both decent - there is a fair chunk of boilerplate overhead for strong typing in React, and a lot of boilerplate for testing.

## React

It's hard to list pros for React when comparing against Svelte because it does each thing (html, css, databinding, testing, eventhandling) in a less developer friendly way, requiring more code, and giving the end user either the same, or slightly worse product.

Pros
* If you've stuffed something up - someone on stackoverflow can probably help you.
* There are a TON of libraries and tools.

Cons
* Strongly typed needed more boilerplate than I'd like.
* It's a bit overwhelming, and a lot of reliance on other libraries.

## Svelte

Achievements
* 38% less lines of code
* 41% less bytes in source (eg, less complex)
* 74% less bytes served to the client
* 83% faster test running

Pros
* Fast to develop - and a joy!
* Isolated components for development (logic, html, css) and testing.
* Fast!
* Has routing out of the box (eg /my-path/component) for serverside hydration or preload
* Strips out unused CSS rules


Cons
* Less likely to get help on stackoverflow.
* Couldn't get debugger working in test case - didn't try to fix.


