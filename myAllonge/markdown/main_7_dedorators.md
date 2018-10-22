# More Decorators
(*this bonus chapter is a work-in-progress*)
## Stateful Method Decorators {#stateful-method-decorators}

As noted in [Method Decorators](#method-decorators), and again in [Symmetry, Colour, and Charm](#symmetry), simple function decorators work and work well for ordinary functions. But in JavaScript, functions can be invoked in different ways, and some of those ways are slightly incompatible with each other.

Of great interest to us are *methods* in JavaScript, functions that are used to define the behaviour of instances. When a function is invoked as a method, the name `this` is bound to the instance, and most methods rely on that binding to work properly.

Consider, for example the simple decorator `requireAll`, that raises an exception if a function is invoked without at least as many arguments as declared parameters:
```js
const requireAll = (fn) =>
  function (...args) {
    if (args.length < fn.length)
      throw new Error('missing required arguments');
    else
      return fn(...args);
  }
```
`requireAll` works perfectly with ordinary functions, what we called "blue" invocations. But if we want to use `requireAll` with methods, we have to write it in such a way that it preserves `this` when it invokes the underlying function:
```js
const requireAll = (fn) =>
  function (...args) {
    if (args.length < fn.length)
      throw new Error('missing required arguments');
    else
      return fn.apply(this, args);
  }
```
It now works properly, including ignoring invocations that do not pass all the arguments. But you have to be very careful when writing higher-order functions to make sure they work as both function decorators and as method decorators.

We called this style of decorator a "green" decorator, because it handles blue (ordinary function) and yellow (method) invocations.

### the problem with state

Handling `this` properly is not the only way in which ordinary function decorators differ from method decorators. Some decorators are stateful, like `once`. Here's a version that correctly sets `this`:
```js
const once = (fn) => {
  let hasRun = false;

  return function (...args) {
    if (hasRun) return;
    hasRun = true;
    return fn.apply(this, args);
  }
}
```
Imagining for a moment that we wish to only allow a person to have their name set once, we might write:
```js
const once = (fn) => {
  let hasRun = false;

  return function (...args) {
    if (hasRun) return;
    hasRun = true;
    return fn.apply(this, args);
  }
}

class Person {
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
};

Object.defineProperty(Person.prototype, 'setName', { value: once(Person.prototype.setName) });

const logician = new Person()
                   .setName('Raymond', 'Smullyan')
                   .setName('Haskell', 'Curry');

logician.fullName()
  //=> Raymond Smullyan
```
As we expect, only the first call to `.setName` has any effect, and it works on a method. But there is a subtle bug that could easily evade naïve attempts to write unit tests:
```js
const logician = new Person()
                   .setName('Raymond', 'Smullyan');

const musician = new Person()
                   .setName('Miles', 'Davis');

logician.fullName()
  //=> Raymond Smullyan

musician.fullName()
  //=> Raymond Smullyan
```
!?!?!?!

What has happened here is that when we write `Object.defineProperty(Person.prototype, 'setName', { value: once(Person.prototype.setName) });`, we wrapped a function bound to `Person.prototype`. That function is shared between all instances of `Person`. That's deliberate, it's the whole point of prototypical inheritance (and the "class-based inheritance" JavaScript builds with prototypes).

Since our `once` decorator returns a decorated function with private state (the `hasRun` variable), all the instances share the same private state, and thus the bug.

### writing stateful method decorators

If we don't need to use the same decorator for functions and for methods, we can rewrite our decorator to use a [WeakSet] to track whether a method has been invoked for an instance:

[WeakSet]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet
```js
const once = (fn) => {
  let invocations = new WeakSet();

  return function (...args) {
    if (invocations.has(this)) return;
    invocations.add(this);
    return fn.apply(this, args);
  }
}

const logician = new Person()
                   .setName('Raymond', 'Smullyan');

logician.setName('Haskell', 'Curry');

const musician = new Person()
                   .setName('Miles', 'Davis');

logician.fullName()
  //=> Raymond Smullyan

musician.fullName()
  //=> Miles Davis
```
Now each instance stores whether `.setName` has been invoked on each instance a `WeakSet`, so `logician` and `musician` can share the method without sharing its state.

### incompatibility

To handle methods, we have introduced "accidental complexity" to handle `this` and to handle state. Worse, our implementation of `once` for methods won't work properly with ordinary functions in "strict" mode:
```js
"use strict"

const hello = once(() => 'hello!');

hello()
  //=> undefined is not an object!
```
If you haven't invoked it as a method, `this` is bound to `undefined` in strict mode, and `undefined` cannot be added to a `WeakSet`.

Correcting our decorator to deal with `undefined` is straightforward:
```js
const once = (fn) => {
  let invocations = new WeakSet(),
      undefinedContext = Symbol('undefined-context');

  return function (...args) {
    const context = this === undefined
                    ? undefinedContext
                    : this;
    if (invocations.has(context)) return;
    invocations.add(context);
    return fn.apply(this, args);
  }
}
```
However, we're adding more accidental complexity to handle the fact that function invocation is <span style="color: blue;">blue</span>, and method invocation is <span style="color: #999900;">khaki</span>.[^colours]

[^colours]: See the aforelinked [The Symmetry of JavaScript Functions](/2015/03/12/symmetry.html)

In the end, we can either write specialized decorators designed specifically for methods, or tolerate the additional complexity of trying to handle method invocation and function invocation in the same decorator.

## Class Decorators beyond ES6/ECMAScript 2015

In [Functional Mixins](#functional-mixins), we discussed mixing functionality *into* JavaScript classes, changing the class. We observed that this has pitfalls when applied to a class that might already be in use elsewhere, but is perfectly cromulant when used as a technique to build a class from scratch. When used strictly to build a class, mixins help us decompose classes into smaller entities with focused responsibilities that can be shared between classes as necessary.

Let's recall our helper for making a functional mixin:
```js
function FunctionalMixin (behaviour, sharedBehaviour = {}) {
  const instanceKeys = Reflect.ownKeys(behaviour);
  const sharedKeys = Reflect.ownKeys(sharedBehaviour);
  const typeTag = Symbol("isA");

  function mixin (target) {
    for (let property of instanceKeys)
      if (!target[property])
        Object.defineProperty(target, property, {
          value: behaviour[property],
          writable: true
        })
    target[typeTag] = true;
    return target;
  }
  for (let property of sharedKeys)
    Object.defineProperty(mixin, property, {
      value: sharedBehaviour[property],
      enumerable: sharedBehaviour.propertyIsEnumerable(property)
    });
  Object.defineProperty(mixin, Symbol.hasInstance, { value: (instance) => !!instance[typeTag] });
  return mixin;
}
```
This creates a function that mixes behaviour into any target, be it a class prototype or a standalone object. There is a convenience capability of making "static" or "shared" properties of the the function, and it even adds some simple `hasInstance` handling so that the `instanceof` operator will work.

Here we are using it on a class' prototype:
```js
const BookCollector = FunctionalMixin({
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._collected_books || (this._collected_books = []);
  }
});

class Person {
  constructor (first, last) {
    this.rename(first, last);
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};

BookCollector(Person.prototype);

const president = new Person('Barak', 'Obama')

president
  .addToCollection("JavaScript Allongé")
  .addToCollection("Kestrels, Quirky Birds, and Hopeless Egocentricity");

president.collection()
  //=> ["JavaScript Allongé","Kestrels, Quirky Birds, and Hopeless Egocentricity"]
```
### mixins that target classes

It's very nice that our mixins support any kind of target, but let's make them class-specific:
```js
function ClassMixin (behaviour, sharedBehaviour = {}) {
  const instanceKeys = Reflect.ownKeys(behaviour);
  const sharedKeys = Reflect.ownKeys(sharedBehaviour);
  const typeTag = Symbol("isA");

  function mixin (clazz) {
    for (let property of instanceKeys)
      if (!clazz.prototype[property])
        Object.defineProperty(clazz.prototype, property, {
          value: behaviour[property],
          writable: true
        });
    clazz.prototype[typeTag] = true;
    return clazz;
  }
  for (let property of sharedKeys)
    Object.defineProperty(mixin, property, {
      value: sharedBehaviour[property],
      enumerable: sharedBehaviour.propertyIsEnumerable(property)
    });
  Object.defineProperty(mixin, Symbol.hasInstance, { value: (instance) => !!instance[typeTag] });
  return mixin;
}
```
This version's `mixin` function mixes instance behaviour into a class's prototype, so we gain convenience at the expense of flexibility:
```js
const BookCollector = ClassMixin({
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._collected_books || (this._collected_books = []);
  }
});

class Person {
  constructor (first, last) {
    this.rename(first, last);
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};

BookCollector(Person);

const president = new Person('Barak', 'Obama')

president
  .addToCollection("JavaScript Allongé")
  .addToCollection("Kestrels, Quirky Birds, and Hopeless Egocentricity");

president.collection()
  //=> ["JavaScript Allongé","Kestrels, Quirky Birds, and Hopeless Egocentricity"]
```
So far, nice, but it feels a bit bolted-on-after-the-fact. Let's take advantage of the fact that [Classes are Expressions]:

[Classes are Expressions]: http://raganwald.com/2015/06/04/classes-are-expressions.html
```js
const BookCollector = ClassMixin({
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._collected_books || (this._collected_books = []);
  }
});

const Person = BookCollector(class {
  constructor (first, last) {
    this.rename(first, last);
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
});
```
This is structurally nicer, it binds the mixing in of behaviour with the class declaration in one expression, so we're getting away from this idea of mixing things into classes after they're created.

But (there's always a but), our pattern has three different elements (the name being bound, the mixin, and the class being declared). And if we wanted to mix two or more behaviours in, we'd have to nest the functions like this:
```js
const Author = ClassMixin({
  writeBook (name) {
    this.books().push(name);
    return this;
  },
  books () {
    return this._books_written || (this._books_written = []);
  }
});

const Person = Author(BookCollector(class {
  // ...
}));
```
Some people find this "clear as day," arguing that this is a simple expression taking advantage of JavaScript's simplicity. The code behind `mixin` is simple and easy to read, and if you understand prototypes, you understand everything in this expression.

But others want a language to give them "magic," an abstraction that they learn on the outside. At the moment, JavaScript has no "magic" for mixing functionality into classes. But what if there were?

### class decorators

There is a well-regarded [proposal] to add Python-style class decorators to JavaScript in the future, nicknamed "ES.later."[^ESdotlater]

[^ESdotlater]: By "ES.later," we mean some future version of ECMAScript that is likely to be approved eventually, but for the moment exists only in transpilers like [Babel](http://babeljs.io). Obviously, using any ES.later feature in production is a complex decision requiring many more considerations than can be enumerated in a book.

[proposal]: https://github.com/wycats/javascript-decorators

A decorator is a function that operates on a class. Here's a very simple example from the aforelinked implementation:
```js
function annotation(target) {
   // Add a property on target
   target.annotated = true;
}

@annotation
class MyClass {
  // ...
}

MyClass.annotated
  //=> true
```
As you can see, `annotation` is a class decorator, and it takes a class as an argument. The function can do anything, including modifying the class or the class's prototype. If the decorator function doesn't return anything, the class' name is bound to the modified class.[^adv]

[^adv]: Although this example doesn't show it, if it returns a constructor function, that is what will be assigned to the class' name. This allows the creation of purely functional mixins and other interesting techniques that are beyond the scope of this post.

A class is "decorated" with the function by preceding the definition with `@` and an expression evaluating to the decorator. in the simple example, we use a variable name.

Hmmm. A function that modifies a class, you say? Let's try it:
```js
const BookCollector = ClassMixin({
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._collected_books || (this._collected_books = []);
  }
});

@BookCollector
class Person {
  constructor (first, last) {
    this.rename(first, last);
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};

const president = new Person('Barak', 'Obama')

president
  .addToCollection("JavaScript Allongé")
  .addToCollection("Kestrels, Quirky Birds, and Hopeless Egocentricity");

president.collection()
  //=> ["JavaScript Allongé","Kestrels, Quirky Birds, and Hopeless Egocentricity"]
```
You can also mix in multiple behaviours with decorators:
```js
const BookCollector = ClassMixin({
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._collected_books || (this._collected_books = []);
  }
});

const Author = ClassMixin({
  writeBook (name) {
    this.books().push(name);
    return this;
  },
  books () {
    return this._books_written || (this._books_written = []);
  }
});

@BookCollector @Author
class Person {
  constructor (first, last) {
    this.rename(first, last);
  }
  fullName () {
    return this.firstName + " " + this.lastName;
  }
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};
```
Class decorators provide a compact, "magic" syntax that is closely tied to the construction of the class. They also require understanding one more kind of syntax. But some argue that having different syntax for different things aids understandability, and that having both `@foo` for decoration and `bar(...)` for function invocation is a win.

Decorators have not been formally approved, however there are various implementations available for transpiling decorator syntax to ES5 syntax. These examples were evaluated with [Babel](http://babeljs.io).

## Method Decorators beyond ES6/ECMAScript 2015

Before ES6/ECMAScript 2015, we decorated a method in a simple and direct way. Given a method decorator like `fluent` (a/k/a `chain`):
```js
const fluent = (method) =>
  function (...args) {
    method.apply(this, args);
    return this;
  }
```
We would wrap functions in our decorator and bind them to names to create methods, like this:
```js
const Person = function () {};

Person.prototype.setName = fluent(function setName (first, last) {
  this.firstName = first;
  this.lastName = last;
});

Person.prototype.fullName = function fullName () {
  return this.firstName + " " + this.lastName;
};
```
With the `class` keyword, we have a more elegant way to do everything in one step:
```js
class Person {

  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

}
```
Since the ECMAScript 2015 syntaxes for classes doesn't give us any way to decorate a method where we are declaring it, we have to introduce this ugly "post-decoration" after we've declared `Person`:
```js
Object.defineProperty(Person.prototype, 'setName', { value: fluent(Person.prototype.setName) });
```
This is weak for two reasons. First, it's fugly and full of accidental complexity. Second, modifying the prototype after defining the class separates two things that conceptually ought to be together. The `class` keyword giveth, but it also taketh away.

### es.later method decorators

To solve a problem created by ECMAScript 2015, [method decorators] have been proposed for a future version of JavaScript (nicknamed "ES.later."[^ESdotlater] The syntax is similar to [class decorators](#es-later-class-decorators), but where a class decorator takes a class as an argument and returns the same (or a different) class, a method decorator actually intercedes when a property is defined on the prototype.

[^ESdotlater]: By "ES.later," we mean some future version of ECMAScript that is likely to be approved eventually, but for the moment exists only in transpilers like [Babel](http://babeljs.io). Obviously, using any ES.later feature in production is a complex decision requiring many more considerations than can be enumerated in a book.

An ES.later decorator version of `fluent` would look like this:
```js
function fluent (target, name, descriptor) {
  const method = descriptor.value;

  descriptor.value = function (...args) {
    method.apply(this, args);
    return this;
  }
}
```
And we'd use it like this:
```js
class Person {

  @fluent
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

};
```
That is much nicer: It lets us use the new class syntax, and it also lets us decompose functionality with method decorators. Best of all, when we write our classes in a "declarative" way, we also write our decorators in a declarative way.

Mind you, we are once again creating two kinds of decorators: One for functions, and one for methods, with different structures. We need a new [colour](#symmetry)!

But all elegance is not lost. Since decorators are expressions, we can alleviate the pain with an adaptor:
```js
const wrapWith = (decorator) =>
  function (target, name, descriptor) {
    descriptor.value = decorator(descriptor.value);
  }

function fluent (method) {
  return function (...args) {
    method.apply(this, args);
    return this;
  }
}

class Person {

  @wrapWith(fluent)
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

};
```
Or if we prefer:
```js
const wrapWith = (decorator) =>
  function (target, name, descriptor) {
    descriptor.value = decorator(descriptor.value);
  }

const returnsItself = wrapWith(
  function fluent (method) {
    return function (...args) {
      method.apply(this, args);
      return this;
    }
  }
);

class Person {

  @returnsItself
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

};
```
[method decorators]: https://github.com/wycats/javascript-decorators

(Although ES.later has not been approved, there is extensive support for ES.later method decorators in transpilation tools. The examples in this post were evaluated with [Babel](http://babeljs.io).)

## Lightweight Traits

> A **trait** is a concept used in object-oriented programming: a trait represents a collection of methods that can be used to extend the functionality of a class. Essentially a trait is similar to a class made only of concrete methods that is used to extend another class with a mechanism similar to multiple inheritance, but paying attention to name conflicts, hence with some support from the language for a name-conflict resolution policy to use when merging.—[Wikipedia][wikitrait]

[wikitrait]: https://en.wikipedia.org/wiki/Trait_(computer_programming)

A trait is like a [mixin](#classes-and-mixins), however with a trait, we can not just define new behaviour, but also define ways to extend or override existing behaviour. Traits are a first-class feature languages like [Scala](http://www.scala-lang.org). Traits are also available as a standard library in other languages, like [Racket](http://docs.racket-lang.org/reference/trait.html). Most interestingly, traits are a feature of the [Self][self] programming language, one of the inspirations for JavaScript.

[self]: https://en.wikipedia.org/wiki/Self_(programming_language)#Traits

Traits are not a JavaScript feature as this essay is being written, but we can easily make lightweight traits out of the features JavaScript already has.

> Our problem is that we want to be able to override or extend functionality from shared behaviour, whether that shared behaviour is defined as a class or as functionality to be mixed in.

### our toy problem

Here's a toy problem we solved elsewhere with a [subclass factory](#mi) that in turn is made out of an extremely simple mixin.[^extremely-simple]

[^extremely-simple]: The implementations given here are extremely simple in order to illustrate a larger principle of how the pieces fit together. A production library based on these principles would handle needs we've seen elsewhere, like defining "class" or "static" properties, making `instanceof` work, and appeasing the V8 compiler's optimizations.

To recapitulate from the very beginning, we have a `Todo` class:
```js
class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false;
  }

  do () {
    this.done = true;
    return this;
  }

  undo () {
    this.done = false;
    return this;
  }

  toHTML () {
    return this.name; // highly insecure
  }
}
```
And we have the idea of "things that are coloured:"
```js
let toSixteen = (c) => '0123456789ABCDEF'.indexOf(c),
    toTwoFiftyFive = (cc) => toSixteen(cc[0]) * 16 + toSixteen(cc[1]);

class Coloured {
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  }

  luminosity () {
    let {r, g, b} = this.getColourRGB();

    return 0.21 * toTwoFiftyFive(r) +
           0.72 * toTwoFiftyFive(g) +
           0.07 * toTwoFiftyFive(b);
  }

  getColourRGB () {
    return this.colourCode;
  }
}
```
And we want to create a time-sensitive to-do that has colour according to whether it is overdue, close to its deadline, or has plenty of time left. If we had multiple inheritance, we would write:
```js
let yellow = {r: 'FF', g: 'FF', b: '00'},
    red    = {r: 'FF', g: '00', b: '00'},
    green  = {r: '00', g: 'FF', b: '00'},
    grey   = {r: '80', g: '80', b: '80'};

let oneDayInMilliseconds = 1000 * 60 * 60 * 24;

class TimeSensitiveTodo extends Todo, Coloured {
  constructor (name, deadline) {
    super(name);
    this.deadline = deadline;
  }

  getColourRGB () {
    let slack = this.deadline - Date.now();

    if (this.done) {
      return grey;
    }
    else if (slack <= 0) {
      return red;
    }
    else if (slack <= oneDayInMilliseconds){
      return yellow;
    }
    else return green;
  }

  toHTML () {
    let rgb = this.getColourRGB();

    return `<span style="color: #${rgb.r}${rgb.g}${rgb.b};">${super.toHTML()}</span>`;
  }
}
```
But we don't have multiple inheritance. In languages where mixing in functionality is difficult, we can fake a solution by having `ColouredTodo` inherit from `Todo`:
```js
class ColouredTodo extends Todo {
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  }

  luminosity () {
    let {r, g, b} = this.getColourRGB();

    return 0.21 * toTwoFiftyFive(r) +
           0.72 * toTwoFiftyFive(g) +
           0.07 * toTwoFiftyFive(b);
  }

  getColourRGB () {
    return this.colourCode;
  }
}

class TimeSensitiveTodo extends ColouredTodo {
  constructor (name, deadline) {
    super(name);
    this.deadline = deadline;
  }

  getColourRGB () {
    let slack = this.deadline - Date.now();

    if (this.done) {
      return grey;
    }
    else if (slack <= 0) {
      return red;
    }
    else if (slack <= oneDayInMilliseconds){
      return yellow;
    }
    else return green;
  }

  toHTML () {
    let rgb = this.getColourRGB();

    return `<span style="color: #${rgb.r}${rgb.g}${rgb.b};">${super.toHTML()}</span>`;
  }
}
```
The drawback of this approach is that we can no longer make other kinds of things "coloured" without making them also todos. For example, if we had coloured meetings in a time management application, we'd have to write:
```js
class Meeting {
  // ...
}

class ColouredMeeting extends Meeting {
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  }

  luminosity () {
    let {r, g, b} = this.getColourRGB();

    return 0.21 * toTwoFiftyFive(r) +
           0.72 * toTwoFiftyFive(g) +
           0.07 * toTwoFiftyFive(b);
  }

  getColourRGB () {
    return this.colourCode;
  }
}
```
This forces us to duplicate "coloured" functionality throughout our code base. But thanks to mixins, we can have our cake and eat it to: We can make `ColouredAsWellAs` a kind of mixin that makes a new subclass and then mixes into the subclass. We call this a "subclass factory:"
```js
function ClassMixin (behaviour) {
  const instanceKeys = Reflect.ownKeys(behaviour);

  return function mixin (clazz) {
    for (let property of instanceKeys)
      Object.defineProperty(clazz.prototype, property, {
        value: behaviour[property],
        writable: true
      });
    return clazz;
  }
}

const SubclassFactory = (behaviour) =>
  (superclazz) => ClassMixin(behaviour)(class extends superclazz {});

const ColouredAsWellAs = SubclassFactory({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },

  luminosity () {
    let {r, g, b} = this.getColourRGB();

    return 0.21 * toTwoFiftyFive(r) +
           0.72 * toTwoFiftyFive(g) +
           0.07 * toTwoFiftyFive(b);
  },

  getColourRGB () {
    return this.colourCode;
  }
});

class TimeSensitiveTodo extends ColouredAsWellAs(Todo) {
  constructor (name, deadline) {
    super(name);
    this.deadline = deadline;
  }

  getColourRGB () {
    let slack = this.deadline - Date.now();

    if (this.done) {
      return grey;
    }
    else if (slack <= 0) {
      return red;
    }
    else if (slack <= oneDayInMilliseconds){
      return yellow;
    }
    else return green;
  }

  toHTML () {
    let rgb = this.getColourRGB();

    return `<span style="color: #${rgb.r}${rgb.g}${rgb.b};">${super.toHTML()}</span>`;
  }
}
```
This allows us to override both our `Todo` methods and the `ColourAsWellAs` methods. And elsewhere, we can write:
```js
const ColouredMeeting = ColouredAsWellAs(Meeting);
```
Or perhaps:
```js
class TimeSensitiveMeeting extends ColouredAsWellAs(Meeting) {
  // ...
}
```
To summarize, our problem is that we want to be able to override or extend functionality from shared behaviour, whether that shared behaviour is defined as a class or as functionality to be mixed in. Subclass factories are one way to solve that problem.

Now we'll solve the same problem with traits.

### defining lightweight traits

Let's start with our `ClassMixin`. We'll modify it slightly to insist that it never attempt to define a method that already exists, and we'll use that to create `Coloured`, a function that defines two methods:
```js
function Define (behaviour) {
  const instanceKeys = Reflect.ownKeys(behaviour);

  return function define (clazz) {
    for (let property of instanceKeys)
      if (!clazz.prototype[property]) {
        Object.defineProperty(clazz.prototype, property, {
          value: behaviour[property],
          writable: true
        });
      }
      else throw `illegal attempt to override ${property}, which already exists.
  }
}

const Coloured = Define({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },

  luminosity () {
    let {r, g, b} = this.getColourRGB();

    return 0.21 * toTwoFiftyFive(r) +
           0.72 * toTwoFiftyFive(g) +
           0.07 * toTwoFiftyFive(b);
  },

  getColourRGB () {
    return this.colourCode;
  }
});
```
`Coloured` is now a function that modifies a class, adding two methods provided that they don't already exist in the class.

But we need a variation that "overrides" `getColourRGB`. We can write a variation of `Define` that always overrides the target's methods, and passes in the original method as the first parameter. This is similar to "around" [method advice][ma-mj]:
```js
function Override (behaviour) {
  const instanceKeys = Reflect.ownKeys(behaviour);

  return function overrides (clazz) {
    for (let property of instanceKeys)
      if (!!clazz.prototype[property]) {
        let overriddenMethodFunction = clazz.prototype[property];

        Object.defineProperty(clazz.prototype, property, {
          value: function (...args) {
            return behaviour[property].call(this, overriddenMethodFunction.bind(this), ...args);
          },
          writable: true
        });
      }
      else throw `attempt to override non-existant method ${property}`;
    return clazz;
  }
}

const DeadlineSensitive = Override({
  getColourRGB () {
    let slack = this.deadline - Date.now();

    if (this.done) {
      return grey;
    }
    else if (slack <= 0) {
      return red;
    }
    else if (slack <= oneDayInMilliseconds){
      return yellow;
    }
    else return green;
  },

  toHTML (original) {
    let rgb = this.getColourRGB();

    return `<span style="color: #${rgb.r}${rgb.g}${rgb.b};">${original()}</span>`;
  }
});
```
`Define` and `Override` are *protocols*: They define whether methods may conflict, and if they do, how that conflict is resolved. `Define` prohibits conflicts, forcing us to pick another protocol. `Override` permits us to write a method that overrides an existing method and (optionally) call the original.

### composing protocols

We *could* now write:
```js
const TimeSensitiveTodo = DeadlineSensitive(
  Coloured(
    class TimeSensitiveTodo extends Todo {
      constructor (name, deadline) {
        super(name);
        this.deadline = deadline;
      }
    }
  )
);
```
Or:
```js
@DeadlineSensitive
@Coloured
class TimeSensitiveTodo extends Todo {
  constructor (name, deadline) {
    super(name);
    this.deadline = deadline;
  }
}
```
But if we want to use `DeadlineSensitive` and `Coloured` together more than once, we can make a lightweight trait with the [`pipeline`](#pipeline) function:
```js
const SensitizeTodos = pipeline(Coloured, DeadlineSensitive);

@SensitizeTodos
class TimeSensitiveTodo extends Todo {
  constructor (name, deadline) {
    super(name);
    this.deadline = deadline;
  }
}
```
Now `SensitizeTodos` combines defining methods with overriding existing methods: We've built a lightweight trait by composing protocols.

And that's all a trait is: The composition of protocols. And we don't need a bunch of new keywords or decorators (like @overrides) to do it, we just use the functional composition that is so easy and natural in JavaScript.

### other protocols

We can incorporate other protocols. Two of the most common are prepending behaviour to an existing method, or appending behaviour to an existing method:
```js
function Prepends (behaviour) {
  const instanceKeys = Reflect.ownKeys(behaviour);

  return function prepend (clazz) {
    for (let property of instanceKeys)
      if (!!clazz.prototype[property]) {
        let overriddenMethodFunction = clazz.prototype[property];

        Object.defineProperty(clazz.prototype, property, {
          value: function (...args) {
            const prependValue = behaviour[property].apply(this, args);

            if (prependValue === undefined || !!prependValue) {
              return overriddenMethodFunction.apply(this, args);;
            }
          },
          writable: true
        });
      }
      else throw `attempt to override non-existant method ${property}`;
    return clazz;
  }
}

function Append (behaviour) {
  const instanceKeys = Reflect.ownKeys(behaviour);

  function append (clazz) {
    for (let property of instanceKeys)
      if (!!clazz.prototype[property]) {
        let overriddenMethodFunction = clazz.prototype[property];

        Object.defineProperty(clazz.prototype, property, {
          value: function (...args) {
            const returnValue = overriddenMethodFunction.apply(this, args);

            behaviour[property].apply(this, args);
            return returnValue;
          },
          writable: true
        });
      }
      else throw `attempt to override non-existant method ${property}`;
    return clazz;
  }
}
```
We can compose a lightweight trait using any combination of `Define`, `Override`, `Prepend`, and `Append`, and the composition is handled by `pipeline`, a plain old function composition tool.

Lightweight traits are nothing more than protocols, composed in a simple and easy-to-understand way. And then applied to simple classes, in a direct and obvious manner.
