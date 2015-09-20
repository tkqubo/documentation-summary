# redux

from:
- https://github.com/rackt/redux
- https://github.com/rackt/redux/tree/master/docs

## Motivation

**Redux attempts to make state mutations predictable** by imposing certain restrictions on how and when updates can happen.

## Threee Principles

- Single source of truth
- State is read-only
- Mutations are written as pure functions

## Ecosystem

- [docs/introductino/Ecosystem](https://github.com/rackt/redux/blob/master/docs/introduction/Ecosystem.md)
- [awesome-redux](https://github.com/xgrommx/awesome-redux)

## Actions & Reducers

Sample guideline

- [flux-standard-action](https://github.com/acdlite/flux-standard-action)


### Terms

- `Actions`: payloads of information that send data from your application to the store
- `Action Creators`: function that returns action
- `Reducer`: specifies how the application's state changes, which:
  - never mutate its arguments
  - never performa side effects like API calls and routing transitions

> "reducer" is the type of function you would pass to `Array.prototype.reduce(reducer, initialValue)`

### reducer example

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return Object.assign({}, state, {
      visibilityFilter: action.filter
    });
  default:
    return state;
  }
}
```

## reducer composition

```js
// reducer 1
function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    ...
  case COMPLETE_TODO:
    ...
  default:
    return state;
  }
}

// reducer 2
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    ...
  default:
    return state;
  }
}

// reducer composition (reducer 1 + reducer 2)
export function dotoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}

// or
import { combineReducers } from 'redux';
const todoApp = combineReducers({
  visibilityFilter, todos
});
export default todoApp;
```

## Store

- `getState()`
- `dispatch(action)`
- `subscribe(listener)`

```js
import { createStore } from 'redux';
import reducers from './reducers';

let store = createStore(reducers);
```

## Usage with React

### Smart and Dumb Components

- from: https://github.com/rackt/redux/blob/master/docs/basics/UsageWithReact.md#smart-and-dumb-components

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">“Smart” Components</th>
            <th scope="col" style="text-align:left">“Dumb” Components</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">Location</th>
          <td>Top level, route handlers</td>
          <td>Middle and leaf components</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Aware of Redux</th>
          <td>Yes</th>
          <td>No</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To read data</th>
          <td>Subscribe to Redux state</td>
          <td>Read data from props</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To change data</th>
          <td>Dispatch Redux actions</td>
          <td>Invoke callbacks from props</td>
        </tr>
    </tbody>
</table>

### Connecting to Redux

- index.js

```jsx
import React from 'react';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import reducers from './reducers';
import App from './components/App';

let store = createStore(reducers);

React.render(
  <Provider stoer={store}>
    {() => <App />}
  </Provider>,
  document.body
);
```

- components/App.js

```js
import React, { Component } from 'react';
import { connect } from 'react-redux';

export class App extends Component {
  ...
}

function select(state) {
  return {
      ...
  };
}

export default connect(select)(App);
```

## Async Actions

Example of async actions:

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_REQUEST_FAILURE', error: '...' }
{ type: 'FETCH_POSTS_REQUEST_SUCCESS', response: { ... } }
```

See [redux-actions](https://github.com/acdlite/redux-actions) for action creators and reducers generation

## Async Action creators

- index.js

```js
import thunkMiddleware from 'redux-thunk';
import { createStore, applyMiddleware } from 'redux';
import reducers from './reducers';

const createStoreWithMiddleware = applyMiddleware(thunkMiddleware)(createStore);
const store = createStoreWithMiddleware(reducers);
```

- actions.js

```js
export function requestPosts(options) {
  ...
};

export function receivePosts(options) {
  ...
};

fetchPosts = options => (dispatch, getState) => {
  dispatch(requestPosts(options));
  return fetch(URL)
    .then(response => response.json())
    .then(json => dispatch(receivePosts(options, json)));
};

export fetchPosts;
```

## Middleware

```js
const logger = store => next => action => {
  console.group(action.type);
  console.info('dispatching', action);
  let result = next(action);
  console.info('next state', store.getState());
  console.groupEnd(action.type);
  return result;
};
```

## Reducing Boilerplate

### Generating Action Creators

- [redux-action-utils](https://github.com/insin/redux-action-utils)
- [redux-actions](https://github.com/acdlite/redux-actions)

### Async Action Creators

```js
export function loadPosts(userId) {
  return {
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    shouldCallAPI: state => !state.users[userId],
    callAPI: () => fetch(`${base}/users/${userId}/posts`),
    payload: { userId }
  }
}

callAPIMiddleware = ({ dispatch, getState }) => next => action => {
  const {
    types: [requestType, successType, failureType],
    callAPI, shouldCallAPI, payload
  } = action;
  ...
  if (!shouldCallAPI(getState())) {
    return;
  }

  dispatch(Object.assign({}, payload, {
    response, type: requestType
  }));
  return callAPI().then(
    response => dispatch(Object.assign({}, payload, { response, type: successType }})),
    error => dispatch(Object.assign({}, payload, { error, type: failureType }}))
  );
}
export callAPIMiddleware;
```

## Server Rendering

> Redux's ***only*** job on the server side is to provide the initial state of our app

c.f. [serve-static](https://github.com/expressjs/serve-static)

- server.js

```js
function renderFullPage(html, initialState) {
  return `
    <!doctype html>
    <html>
      <body>
        <div id="app">${html}</div>
        <script>
          window.__INITIAL_STATE__ = ${JSON.stringify(initialState)};
        </script>
      </body>
    </html>
  `;
}
```

- client.js

```js
const initialState = window.__INITIAL_STATE__;
```

## Writing Tests

- [React shallow renderer](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering)

```js
import jsdomReact from './jsdomReact';
...
const { TestUtils } = React.addons;

function setup() {
  let props = {
    addTodo: expect.createSpy()
  };

  let renderer = TestUtils.createRenderer();
  renderer.render(<Header {...props} />);
  let output = renderer.getRenderOutput();

  return {
    props,
    output,
    renderer
  };
}

describe('components', () => {
  jsdomReact();

  it('should render', () => {
    const { output, props } = setup();
    ...
    let input = output.props.children[1];
    expect(props.addTodo.calls.length).toBe(0);
    input.props.onSave();
    expect(props.addTodo.calls.length).toBe(1);
  }):
});
```

### Fixing Broken `setState()`

> Shallow rendering currently throws an error if setState is called.
> To work around the issue, we use jsdom so React doesn’t throw the exception when DOM isn’t available.

c.f. https://github.com/facebook/react/issues/4019

```js
import ExecutionEnvironment from 'react/lib/ExecutionEnvironment';
import jsdom from 'mocha-jsdom';

export default function jsdomReact() {
  jsdom();
  ExecutionEnvironment.canUseDOM = true;
}
```

### Connected Components

```js
import { connect } from 'react-redux';

// unconnected component
export class App extends Component { ... }

// connected component
export default connect(mapProps)(App);
```

## Computing Derived Data

- [reselect](https://github.com/faassen/reselect)

```js
import { createSelector } from 'reselect';
import { VisibilityFilters } from './actions';

function selectTodos(todos, filter) {
  ...
}

const visibilityFilterSelector = (state) => state.visibilityFilter;
const todosSelector = (state) => state.todos;

export const visibleTodosSelector = createSelector(
  [visibilityFilterSelector, todosSelector],
  (visibilityFilter, todos) => {
    return {
      visibleTodos: selectTodos(todos, visibilityFilter),
      visibilityFilter
    };
  }
);
```
