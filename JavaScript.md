# JavaScript

**Table of contents**

 * [Features](#features)
 * [Iteration](#iteration)
 * [Immutability](#immutability)
 * [Comments](#comments)
 * [Tests](#tests)

---

The [Molindo ESLint](https://github.com/molindo/eslint-config-molindo) rules should be used to increase code quality.

## Features

It is encouraged to use newer ECMAScript features to increase expressiveness and reduce the amount of code. It's ok to use ECMAScript proposals if they've already reached stage 3 of the standardization process (see [current TC39 proposals](https://github.com/tc39/proposals)).

Beside syntax it's also encouraged to use new standardized additions to the JavaScript standard library. If they aren't available in all supported browsers, they can usually be polyfilled. This should be preferred to including a library when possible.

Small utility functions how they are offered by lodash, underscore, etc. should be proxied with utility modules like `ObjectUtils.deepMerge`. This makes the codebase flexible to switching them out for a better implementation or maybe eventually a browser built in function.

## Iteration

Iterating through arrays should whenever possible be achieved by using an expressive method like `[].map`, `[].filter`, `[].every`, `[].find`, `[].findIndex`, `[].reduce` or `[].some`. This provides a clearer intent in comparison to iterating with counter variables and they work great with chaining. When side effects are necessary, using `[].forEach` is a good idea.

Iterating through objects is easiest when converting the object to an array with `Object.keys`, `Object.entries` or `Object.values` and then using an array function. This avoids `hasOwnProperty` checks as they'd be necessary with `for in` loops and also enables chaining.

## Immutability

Immutable operations are generally preferred to their mutative alternatives (e.g. `[].slice` instead of `[].splice`).

Mutative functions should only be used when there are no immutable alternatives (e.g. `[].sort`). For these cases it can be helpful to make a shallow copy of the data structure first (e.g. `[].slice().sort()`). Mutation should at least be limited to a single function scope. Shared mutable state can lead to very hard to debug errors.

## Comments

Comments are useful when the code isn't self documenting. They should be omitted however if they don't provide additional value.

When it's possible to remove a comment by using a meaningful identifier name, this should be preferred.

E.g. consider this code:

```js
if (
 anObject.someBoolean
 && anotherVariable.someNestedString.includes('a')
 && anArray.filter(cur => cur.isValid).length > 2
) {
 // Is invalid state
 this.recoverFromBadState();
 this.doSomethingElse();
}
```

This can be turned into the following code in order to remove the comment and be self-documenting:

```js
const isInvalidState = anObject.someBoolean
 && anotherVariable.someNestedString.includes('a')
 && anArray.filter(cur => cur.isValid).length > 2;

if (isInvalidState) {
 this.recoverFromBadState();
 this.doSomethingElse();
}
```

## Tests

The baseline for tests that should be written is the following:

1. Unit tests for modules with sophisticated logic.
2. Unit tests for documenting expected behaviour that might not be obvious in the future.
3. Integration tests for modules which depend on many smaller modules so the less complex modules are also covered at least once.
