# CSS

**Table of contents**

 * [Naming](#naming)
 * [Shared styling](#shared-styling)
 * [Variables](#variables)
 * [Properties](#properties)
 * [Responsive](#responsive)
 * [Reset](#reset)

---

Styles use SASS with the SCSS syntax. Scoping styles to a component can be achieved with [CSS modules](https://github.com/css-modules/css-modules).

The only thing used from CSS modules are the local classnames. Other features like `compose` are avoided and solved with well-known SASS features like mixins.

## Naming

The structure is inspired by [BEM (Block, Element, Modifier)](https://en.bem.info/methodology/quick-start/). Since all class names are scoped to a component, we can optimize the naming a bit:

```css
// Component: 'B' of BEM, should always be called "root".
.root {
  padding: 12px;

  // Modifier of a component: 'M' of BEM. The name should be scoped
  // to the component by prefixing it and separating the modifier
  // name with an underscore. Referencing the parent name can be
  // achieved with the ampersand operator, which also leads to
  // meaningful nesting (`&_hidden` compiles to `root_hidden`).
  &_hidden {
    display: none;
  }
}

// Elements used within the component: 'E'
// of BEM; use camelCase for multiple words.
.nestedButton {
  color: blue;

  // Modifiers can also be set on elements.
  &_inactive {
    opacity: 0.25;
  }
}
```

The naming allows easy access from JavaScript: `styles.nestedButton_inactive` vs. `styles['nestedButton--inactive']`.

If multiple elements use the same modifier, it's usually a good idea to apply the modifier to the component and modify the children with a child selector (`.root_hidden > .button `).

Older projects where CSS modules aren't available use actual BEM syntax like `SomeComponent__nested-element--with-modifier`.

## Shared styling

Usually shared styling should be achieved by reusing components. If this abstraction doesn't fit the usecase, shared files can be put in a `src/styles` folder.

This is helpful for:

 - Animation mixins
 - Configuration variables like colors, border radii, …
 - Font mixins (So far it seems like this works better than a component, since most text elements need a class anyway for margins, etc.. By using a mixin there's less styling information in the markup.)
 - Media queries
 - Shadows
 - etc.

All names used in this folder should be unique across the whole codebase, therefore use more specific names when in doubt.

## Variables

When a value occurs multiple times in a stylesheet, it's a good idea to put it in a variable:

```css
.badge {
  $height: 45px;

  height: $height;
  border-radius: $height / 2;
}
```

It's also encouraged to put variables in SASS closures if they are only used in a specific rule set, as this scopes the variable name.

## Properties

Declarations in rule sets should be ordered as follows:

1. Properties that control the layout behaviour: `display`, `position`, `box-sizing`, …
2. Layout properties: `margin`, `top`, `padding`, …
3. Purely stylistic properties. `color`, `background`, `opacity`, …

Mixins should be placed based on which properties they affect, e.g.:

```css
.button {
  display: inline-block;
  position: relative;
  @include apply-some-padding(20px);
  background-color: aliceblue;
  color: darkslategrey;
}
```

Vendor prefixes are automatically added by Autoprefixer, hence there shouldn't be a need for vendor prefixes in styles.

In terms of component boundaries it has proven valuable that a component doesn't define margins on its own, but rather lets a parent component control this. This improves reusability of the component without overrides.

Within a component its useful that margins are controlled in a single direction, e.g. setting only margin-top values from elements that are placed underneath each other, instead of sometimes using marign-top and sometimes margin-bottom.

## Responsive

Stylesheets should be set up mobile first with larger viewports overriding properties from smaller viewports. E.g.:

```css
.button {
  padding: 10px;

  @include match-medium {
    padding: 20px;

    @include match-large {
      padding: 30px;
    }
  }
}
```

As shown in this example, consistent media queries can be achieved with small SASS mixins.

Media queries like `match-only-medium` should be avoided, but if they lead to cleaner code it's ok to use them.

## Reset

Every app should use [Eric Meyers reset](https://www.npmjs.com/package/node-reset-scss) to get consistent styling across browsers. From there on, there should be no global styles at all, but only styled components.
