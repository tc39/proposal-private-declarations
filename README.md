# Private Declarations

A proposal to add Private Declarations, allowing trusted code _outside_ of the class lexical scope to access private state.

```js
private #hello;
class Example {
  outer #hello = 'world!';

  hello() {
    return this.#hello;
  }
}

const ex = new Example();
console.log(ex.hello()); // => 'world!'
console.log(ex.#hello); // => 'world!'
```

This also allows us to bring private state to regular objects!

```js
private #hello;
function Example() {
  return {
    outer #hello: 'world',

    hello() {
      return this.#hello;
    }
  }
}

const ex = Example();
console.log(ex.hello()); // => 'world!'
console.log(ex.#hello); // => 'world!'
```

Possible [later proposals](https://docs.google.com/presentation/d/1Zu9uCFMUU4zLwBVSd3OOxtsm-CYyYvJIryLVGW5leoA/edit#slide=id.g4d82425673_0_79)
can allow sharing private declarations to friendly module.

## Champions

- Justin Ridgewell ([@jridgewell](https://github.com/jridgewell/))

## Status

Current [Stage](https://tc39.es/process-document/): 1

## Guiding Use Cases

### "Protected" state

Protected state is a valuable visibility state when implementing class
hierarchies. For for instance, a hook pattern might be used to allow
subclasses to override code:

```js
// https://github.com/Polymer/lit-html/blob/1a51eb54/src/lib/parts.ts

private #createPart;

class AttributeCommitter {
  //...

  outer #createPart() {
    return new AttributePart(this);
  }
}

class PropertyCommitter extends AttributeCommitter {
  outer #createPart() {
    return new PropertyPart(this);
  }
}
```

Here, `AttributeCommitter` explicitly allows trusted (written in the
same source file) subclasses to override the behavior of the
`#createPart` method. By default, a normal `AttributePart` is returned.
But `PropertyCommitter` works only on properties and would return a
`PropertyPart`. All other code is free to be inherited via normal
publicly visible fields/methods from `AttributeCommitter`.

**Note** that this does not privilege code outside the file to override
the `#createPart` method, as the `#createPart` private declaration is
visible only in the scope where it is declared.

### Friend classes/functions

For prior art in C++, see https://en.wikipedia.org/wiki/Friend_function.

The AMP Project has a particular staged linting pattern that works well
with friendly functions that guard access to restricted code. To begin
with, statically used functions are considerably easier to lint for than
object-scoped method calls because we do not need to know the objects
type. So its much easier to determine if this is a restricted call or
just a non-restricted call that uses the same method name. Eg, it's
easier to tell that a static export `registerExtendedTemplate` is
restricted vs `obj.registerExtendedTemplate`.

```js
// https://github.com/ampproject/amphtml/blob/18baa9da/src/service/template-impl.js

private #registerTemplate;

// Exported so that it may be intalled on the global and shared
// across split bundles.
export class Templates {
  outer #registerTemplate() {
    //...
  }
}

// The code privileged to register templates with the shared class
// instance. Importing and using is statically analyzable, and must pass
// a linter.
export function registerExtendedTemplate() {
  const templatesService = getService('templates');
  return templatesService.#registerTemplate(...arguments);
}
```
