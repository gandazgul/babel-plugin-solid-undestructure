# babel-plugin-solid-undestructure

This Babel plugin allows you to destructure your props in your Solid components without losing reactivity.

The plugin will "un-destructure" your props at build time, so the code you pass into the Solid compiler will not have destructured props at runtime. Instead the props will be accessed the normal way with `props.someProp`.

> **Note**  
> This plugin is compatible with Solid 1.6 and is likely to work with every version of Solid 1.x.

Usage with examples:

```jsx
// Use the `Component` type to mark components that will be transformed by the plugin
import type { Component } from 'solid-js'
const MyComp: Component<...> = ({ a, b, c }) => {a; b; c;};

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓  The above code will compile to
import type { Component } from 'solid-js'
const MyComp: Component<...> = props => {props.a; props.b; props.c;}



// Also works with the `ParentComponent` type
import type { ParentComponent } from 'solid-js'
const MyComp: ParentComponent<...> = ({ a, b, c }) => {a; b; c;};

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
import type { ParentComponent } from 'solid-js'
const MyComp: ParentComponent<...> = props => {props.a; props.b; props.c;}



// You can use a compile time function instead of using the `Component` type (needed for vanilla JS)
import { component } from 'undestructure-macros'
const MyComp = component(({ a, b, c }) => {a; b; c;})

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
const MyComp = props => {props.a; props.b; props.c;}



// Default props using `mergeProps`
import type { Component } from 'solid-js'
const MyComp: Component<...> = (
  { a = 1, b = 2, c = 3 } = defaultProps
) => {a; b; c;}

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
import { type Component, mergeProps } from 'solid-js'
const MyComp: Component<...> = props => {
  props = mergeProps(defaultProps, { a: 1, b: 2, c: 3 }, props);
  props.a; props.b; props.c;
}



// Rename props
import type { Component } from 'solid-js'
const MyComp: Component<...> = ({ a: d, b: e, c: f }) => {d; e; f;}

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
import { type Component, mergeProps } from 'solid-js'
const MyComp: Component<...> = props => {props.a; props.b; props.c;}



// Rest element destructuring using `splitProps`
import type { Component } from 'solid-js'
const MyComp: Component<...> = ({ a, b, c, ...other }) => {a; b; c; other;}

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
import { type Component, splitProps } from 'solid-js'
const MyComp: Component<...> = props => {
  let other;
  [props, other] = splitProps(props, ["a", "b", "c"]);
  props.a; props.b; props.c; other;
}



// You can nest components
import type { Component } from 'solid-js'
const Parent: Component<...> = ({ a, b }) => {
  const Child: Component<...> = ({ c, d }) => {
    a; b; c; d;
  }
}

//  ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
import type { Component } from 'solid-js'
const Parent: Component<...> = props1 => {
  const Child: Component<...> = props2 => {
    props1.a; props1.b; props2.c; props2.d;
  }
}
```

See also [undestructure-example](https://github.com/orenelbaum/undestructure-example) (or [Open in Stackblitz](https://stackblitz.com/github/orenelbaum/undestructure-example)).


## `Component` Type Annotation

When this option is enabled (it is enabled by default), the plugin transforms all arrow function components which are part of a variable declaration with a `Component` type annotation.

The type annotation must be a direct reference to the Solid `Component` type import, or an annotation of the form `Solid.Component` where `Solid` is a reference to the default import of Solid.

In both cases you can use the 'import as' syntax.

Examples:

```tsx
import type { Component } from 'solid-js'

const MyComponent: Component = // ...
```

```tsx
import Solid from 'solid-js'

const MyComponent: Solid.Component = // ...
```

```tsx
import type { Component as ComponentAlias } from 'solid-js'

const MyComponent: ComponentAlias = // ...
```

This example won't work:

```tsx
import type { Component } from 'solid-js'

type ComponentAlias = Component

const MyComponent: ComponentAlias = // ...
```

In this last example, `MyComponent` won't be transformed.


## Compile Time Function Annotation (Needed for Vanilla JS)

This option can be used if you are using vanilla JS or you don't want to rely on types to manipulate runtime behavior like the type annotation does.
When this option is enabled (it is enabled by default), the plugin transforms all component that are wrapped in the `component` compile-time function provided by this plugin.

The `component` compile-time function must be a direct reference to the `component` named export from `undestructure-macros`

You can also use the 'import as' syntax.

Examples:

```jsx
import { component } from 'undestructure-macros'

const MyComponent = component(/* your component goes here. */)
```

```tsx
import { component as c } from 'undestructure-macros'

const MyComponent = c(/* your component goes here. */)
```

This example won't work:

```tsx
import { component } from 'undestructure-macros'

const c = component

const MyComponent = c(/* your component goes here. */)
```

In this last example, `MyComponent` won't be transformed.

## uppercaseFuncNames option

By setting this option to `true`, the plugin will assume function with names that start with an uppercase letter are components and will transform them accordingly.

This option is disabled by default.

Example:

```javascript
const MyComponent = ({ a, b, c }) => { ... }
```

```javascript
function MyComponent({ a, b, c }) { ... }
```

## Installation and Configuring Vite

Install the plugin and the macro placeholder package with

```sh
npm i -D babel-plugin-solid-undestructure undestructure-macros
```

In your Vite config, import `undestructurePlugin` from `babel-plugin-solid-undestructure`

```js
import { undestructurePlugin } from "babel-plugin-solid-undestructure"
```

### TS with Type Annotation Support

If your'e working with TypeScript code and you want to use the `Component` type annotation, add this to the top of the plugin list in your Vite config:

```js
...undestructurePlugin("ts")
```

With this configuration you can use both the `Component` type and the `component` compile time function to annotate components.

### TS or Vanilla JS, No Type Annotation Support

In your Vite config, find the your vite-plugin-solid initialization (in the default Solid template it will be imported as `solidPlugin`).

The first argument this initialization function takes, is the options object.

Add this field to the initializer options:
```js
babel: {
	plugins: [undestructurePlugin("vanilla-js")]
} 
```

With this configuration you can use both the `Component` type and the `component` compile time function to annotate components.


## Roadmap

- Support for `ParentComponent` type annotation
- A pragma to annotate components

## Features Under Consideration

- Nested destructuring
- Destructure CTF (probably not but maybe)

If you want a feature to be added to this plugin, whether it's on this list or not, please open a feature request or tell me [in this discussion](https://github.com/orenelbaum/babel-plugin-solid-undestructure/discussions/5).


## Other Cool Plugins for Solid:

- https://github.com/orenelbaum/babel-plugin-reactivars-solid - A Svelte-like "reactive variables" plugin for Solid that lets you pass reactive variables (getter + setter) around in a concise way (also made by me).
- https://github.com/LXSMNSYC/babel-plugin-solid-labels - Solid labels is more of an all in one plugin. It has Svelte-like reactive variables, prop destructuring (like this plugin) and more.
- https://github.com/LXSMNSYC/solid-sfc - An experimental SFC compiler for SolidJS.


## Development

The project is written in TypeScript and I think that it's relatively well documented and written in a way that makes it easy to understand as long as you're familiar with Babel plugins. If you're not, Babel plugins are much more straight forward than they look, I recommend playing a bit with [AST Explorer](https://astexplorer.net/) to get a feel for how they work, and you can also take a look at [the Babel plugin handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md).

It's recommended to use pnpm for development.

Get started with:
```sh
pnpm i
```

You can test your changes either by adding a test or by using the playground.
A test should be added for every new feature anyway, but you can use the playground if you don't want to start by writing a test, you can also skip writing a test altogether, in this case I'll add a test myself.

For working with the playground and the tests, if you're on VSCode I recommend installing the extension [es6-string-javascript](https://marketplace.visualstudio.com/items?itemName=zjcompt.es6-string-javascript) which will highlight JS code inside template literals using a pragma, this way:
```js
/*javascript*/`code goes here`
```

The playground is a file called `playground.cjs`. To work with the playground just go to the file and edit the code in the `src` variable.
Your can run the playground with:
```sh
pnpm run playground
```

Writing a test is a bit trickier since uvu doesn't support snapshots and I'm too lazy to set it up in a better way.
So I won't document the process right now, I'll just mention that you can run the tests with:
```sh
pnpm run test
```
