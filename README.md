
# redux-simple-router

[Redux](https://github.com/rackt/redux) is cool.
[react-router](https://github.com/rackt/react-router) is neat. The
problem is that react-router manages an important piece
of your application state: the URL. If you are using redux, you want
your app state to fully represent your UI; if you snapshotted the app
state, you should be able to load it up later and see the same thing.

react-router automatically maps the current URL to a path down your
component tree, and continually does so with any URL changes. This is
very useful, but we really want to store this state in redux as well.

The entire state that we are interested in boils down to one thing:
the URL. This is an extremely simple library that just puts the URL in
redux state and keeps it in sync with any react-router changes.
Additionally, you can change the URL via redux and react-router will
change accordingly.

```js
npm install redux-simple-router
```

## Isn't there already redux-router?

[redux-router](https://github.com/rackt/redux-router) is another
project which solves the same problem. However, it's far more complex.
Just look at this code: the whole thing is only 68 lines of JS.
redux-router is much bigger and more complex.

That said, redux-router is a fine project and has features this
doesn't provide. Use it if you like it better.

The differences between this and redux-router:

* Much smaller and simpler. You don't need to learn another library on
  top of everything else.
* We encourage direct access of react-router APIs. Need server-side
  rendering, or something else advanced? Just read react-router's
  docs.
* The piece of state is just a URL string, whereas redux-router stores
  the full location object from react-router.

This only depends on the `history.listen` function from react-router
and the `store.getState` and `store.subscribe` functions from redux.
It should be very future-proof with any versions of either libraries.

## How to Use

The idea of this library is to use react-router's functionality exactly
like its documentation tells you to. You can access all of its APIs in
routing components. Additionally, you can use redux like you normally
would, with a single app state and "connected" components. It's even
possible for a single component to be both if needed.

This library provides a single point of synchronization: the
`routing.path` piece of state which is the current URL. You can read
it, and also change it with an action.

Here's some code:

```js
const { createStore, combineReducers } = require('redux');
const { Provider } = require('react-redux');
const { Router, Route } = require('react-router');
const createBrowserHistory = require('history/lib/createBrowserHistory');
const { syncReduxAndRouter, routeReducer } = require('redux-simple-router');
const reducers = require('<project-path>/reducers');

const reducer = combineReducers(Object.assign({}, reducers, {
  routing: routeReducer
}));
const store = createStore(reducer);
const history = createBrowserHistory();

syncReduxAndRouter(history, store);

React.render(
  <Provider store={store}>
    <Router history={history}>
      <Route path="/" component={App}>
        <Route path="foo" component={Foo}/>
        <Route path="bar" component={Bar}/>
      </Route>
    </Router>
  </Provider>,
  document.getElementById('mount')
);
```

Now you can read from `state.routing.path` to get the URL. It's far
more likely that you want to change the URL more often, however. You
can use the `updatePath` action creator the we provide:

```js
const { updatePath } = require ('redux-simple-router');

function MyComponent({ dispatch }) {
  return <Button onClick={() => dispatch(updatePath('/foo'))}/>;
}
```

This will change the state, which will trigger a change in react-router.

Additionally, if you want to respond to the path update action, just
handle the `UPDATE_PATH` constant that we provide:

```js
const { UPDATE_PATH } = require('redux-simple-router');

function update(state, action) {
  switch(action.type) {
  case UPDATE_PATH:
    // do something here
  }
}
```

## API

### `syncReduxAndRouter(history, store)`

Call this with a react-router and a redux store instance to install
hooks that always keep both of them in sync. When one changes, so will
the other.

### `routeReducer`

A reducer function that keeps track of the router state. You need to
add this reducer to your app reducers when creating the store. **The
piece of state must be named `routing`**

### `UPDATE_PATH`

An action type that you can listen for in your reducers.

### `updatePath(path, noRouterUpdate, replace)`

An action creator that you can use to update the current URL. Just
pass it a string like `/foo/bar?param=5`.

The `noRouterUpdate`, if `true`, will stop react-router from reacting
to this and all future URL changes. Pass `false` to make it start
reacting again. This is useful if replaying snapshots while using the
`forceRefresh` option of the browser history which forces full
reloads. It's a rare edge case.

The `replace` flag can be set to trigger a `replaceState` instead of a
`pushState`.
