# React

**Table of contents**

 * [Directory structure](#directory-structure)
 * [Destructure props and state](#destructure-props-and-state)
 * [PropTypes](#proptypes)
 * [Event listeners](#event-listeners)
 * [Stateless functional components](#stateless-functional-components)
 * [Pull out state from reusable components](#pull-out-state-from-reusable-components)
 * [Labels](#labels)
 * [Flux &amp; Redux](#flux--redux)
    * [Actions](#actions)
    * [Store structure](#store-structure)
    * [Reducers](#reducers)

---

## Directory structure

Mostly incluenced by the seperation of [containers and presentational components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

```
src
  containers
    These are the only components that are allowed to receive data
    from a store or dispatch actions. In most cases these are also
    route handlers, but to avoid passing props through too much levels,
    it can be helpful to also connect other components to the store.

    An exception to this convention are components that are specific to
    only one container. They can be placed as siblings to the container
    managing them. The container can always easily be identified,
    because it has the same name as the folder it lies within. This can
    be helpful to not bloat the `components` directory unnecessarily.

  components
    Are mostly concerned with how things look and are often used in multiple
    trees that originate from a container. Can also contain some UI state.

  actions
    Here are all the actions that can be dispatched within the app.

  store
    The store holds shared global state. In this directory are
    stores (reducers + getters), middlewares which are being executed
    before an action reaches the reducers and initialization logic.

  styles
    Here are shared mixins and variables that can be
    used throughout the app to achieve shared styling.

  utils
    Utility functions that need to be shared
    across the app (e.g. StringUtils, DOMUtils, …).

  config
    Global configuration that needs to be shared across the app.

  __testUtils__
    Shared stubs, mocks and utilities for testing.
```

## Destructure props and state

A function of a React component should use [object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring) to enable short names for variables within the function (e.g. `value` instead of `this.props.value`):

```
render() {
  const {value, showMarker, isEditing} = this.props;
  // ...
}
```

This applies to `props` as well as `state`.

However, if it's a very short function and only one or two properties from `props` or `state` are used, it's ok to just write `this.props.value`.

For [stateless functional components](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) the destructuring of `props` can happen directly in the argument list:

```
const Person = ({firstname, lastname}) => (
  <div>
    {firstname} {lastname}
  </div>
);
```

## PropTypes

All components should specify all the props they could possibly get within `propTypes`. If a component only forwards props to a child, the prop should still be explicitly specified instead of passing all received props down to a child.

If a component has some internal configuration, this should also be put into `propTypes` with `defaultProps`. This should eliminate the need for other static properties completely. What's good about this pattern, is that it increases the testability of such components.

## Event listeners

Usually event listeners don't have the class instance set as the context which calls the function. To avoid issues with `Cannot read property x of …` these functions should be defined as class property arrow functions:

```
onButtonClick = e => {
  console.log(e, this.props);
};
```

For event listeners that need to be attached manually, this also allows to remove them during `componentWillUnmount` since the function reference stays the same during the component lifecycle.

## [Stateless functional components](https://reactjs.org/docs/components-and-props.html#functional-and-class-components)

These kind of components are generally preferred to the stateful ones, because there's less boilerplate and they're more performant.

However when one of the following aspects is applicable to a component, it needs to be a stateful one:

 - It needs lifecycle hooks.
 - It needs event handlers.
 - It needs local state.
 - Its render method is split across multiple functions (passing around props can be cumbersome).

So basically they can only be used, if they're just rendering a bit of HTML.

## Pull out state from reusable components

For reusable components that need state, the state can be pulled out in a separate component.

E.g. when implementing a table that provides client-side sorting functionality it's a good idea to build a `Table` component that is a [pure function](https://en.wikipedia.org/wiki/Pure_function) and doesn't hold any internal state (no `setState(…)`). In addition to this, a component `SortableTable` can be added which provides an easy API where sorting should be available as a feature but shouldn't bother the consumer.

Other examples of this pattern are `Button` and `AsyncButton` (disables itself if the `onClick` callback returns a Promise) or `Dropdown` and `DropdownManager` (handles the `isOpen` state based on a supplied button).

Benefits:

 - Better testability of the components since different rendering outputs can be tested without simulating inputs like clicks.
 - More concise components that are easier to understand.
 - Using parent components that compose the stateless ones makes sure that consumer components are not concerned with repetitive state logic (e.g. is the `Dropdown` open?).
 - It allows to implement different kinds of state manager components to be used independently.

This pattern is mostly useful for components that are reused very frequently and shouldn't be overdone for more specific components, as it will likely feel over-engineered.

## Labels

UI labels should be separated from render logic.

For apps that support a single locale, these can be put into a sibling module:

```js
// src/components/ShopList/labels.js

export default {
  title: 'Shop results',

  // Related labels can be grouped by nesting them within an object.
  sorting: {
    alphabeticalIncreasing: 'Alphabetical A-Z',
    alphabeticalDecreasing: 'Alphabetical Z-A'
  }
};

export const ShopItem = {
  // Variables can be interpolated into strings by using functions.
  address: address => `${address.zipCode} ${address.place}, ${address.country}`
};
```

For apps that support multiple locales, a library like [react-intl](https://github.com/yahoo/react-intl) is useful.

## Flux & Redux

For some kind of state it can be helpful to move it to a [Redux](http://redux.js.org/) store. However, for most state the local component state of React should be sufficient.

The following types of components should use local state (i.e. `setState(...)`):

 * Reusable components within the `components` directory.
 * Components within the `containers` directory, that have an UI state that is only used within the component itself or by child components.
 * If a component needs to keep its state when unmounting, this can also be achieved with local state (e.g. [react-keep-state](https://github.com/amannn/react-keep-state)).

These are indicators where a Flux store can be helpful:

 * Complex state transitions that should be well tested.
 * Containers share UI state with other sibling containers and it's not practical to lift the state to a shared parent.
 * Containers that have an UI state that is only affected by actions (e.g. a global loading indicator based on API actions).
 * Taking advantage of features that are typically hard to implement in React (persisting to local storage or booting up from it, passing sophisticated state from a server to a client, etc. – see [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367) for a good collection).

API data should be handled by the GraphQL client, so there's no need of putting this data in a Flux store.

Therefore, most UI state will go into the local component state. This saves a lot of boilerplate and increases the cohesion of the components.

### Actions

Actions should follow the [flux-standard-action](https://github.com/acdlite/flux-standard-action) pattern.

```
{
 type: 'DATE_PICKER_SELECT',
 payload: {id: 2, value: 'ONE'},
 meta: {analytics: …}
}
```

Action types follow the pattern `[entity]_[verb](_[state])`.

### Store structure

Generally a store should export the following shape:

```js
export default {
  // The key the reducer should be registered in the root state
  stateKey: string,

  // Components shouldn't touch the root state directly,
  // but should rather use getters offered by a store.
  get: {[name]: (rootState: object, ...args?: Array<any>) => any},

  // The reducer to be registered by Redux
  reducer: (state: object, action: object) => object
}
```

### Reducers

Only array and object functions should be used, that don't manipulate the original data, but return a new copy. This is a requirement from Redux and also improves performance, since easier checks can be done, to see if the state has changed (`oldState !== newState` instead of a deep comparison).

JavaScript offers methods for creating shallow copies of objects and arrays which avoid mutation (e.g. `[].concat`, `[].slice`, …). If this becomes too verbose, a library can help here (e.g. [`seamless-immutable`](https://github.com/rtfeldman/seamless-immutable)). The goal for picking an immutability library should be that components don't have to know about it and can use regular JavaScript for accessing properties etc.
