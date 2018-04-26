# More Decorator Recipes

![For the love of coffee: a collection](images/new-ideas.jpg)

> "The entire history of Mankind's relationship with coffee is a futile attempt to have the reality of its taste live up to the promise of its aroma."
## After Method Advice {#after}

Consider the bare bones of this `Todo` class that we might use as part of a ViewModel in a front-end application. Many front-end libraries have special features that allow views or other models to persist changes to one or more actual models and/or data stores.

We'll just hand-wave by pretending there is a `persist` method. It could be mixed in or inherited, we'll sketch it in for illustration:

{:lang="js"}
~~~~~~~~
class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false
  }

  do () { this.done = true; }

  undo () { this.done = false; }

  setName (name) { this.name = name; }

  persist () {
    // presist changes to model(s) and/or
    // data stores...
  }
}
~~~~~~~~

Naturally, updating a todo should persist changes. So we could write:

{:lang="js"}
~~~~~~~~
class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false
  }

  do () {
    this.done = true;
    this.persist();
  }

  undo () {
    this.done = false;
    this.persist();
  }

  setName (name) {
    this.name = name;
    this.persist();
  }

  persist () {
    // presist changes to model(s) and/or
    // data stores...
  }
}
~~~~~~~~

This is very similar to making methods fluent. We're obscuring the primary responsibility of the method with cross-cutting concerns. We can and should abstract persistence into an ES.later decorator:

{:lang="js"}
~~~~~~~~
const persists = function (target, name, descriptor) {
  const method = descriptor.value;

  descriptor.value = function (...args) {
    const value = method.apply(this, args);

    this.persist();
    return value;
  }
}

class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false
  }

  @persists
  do () {
    this.done = true;
  }

  @persists
  undo () {
    this.done = false;
  }

  @persists
  setName (name) {
    this.name = name;
  }

  persist () {
    console.log(`persisting ${this.name}`);
  }
}
~~~~~~~~

### after decorators

Combinators for functions come in an unlimited panoply of purposes and effects. So do method combinators, but whether from intrinsic utility or custom, certain themes have emerged. One of them that forms a core part of the original [Lisp Flavors][flavors] system and also the [Aspect-Oriented Programming][aop] movement, is decorating a method with some functionality to be performed *after* the method's body is evaluated.

[flavors]: https://en.wikipedia.org/wiki/Flavors_(programming_language)
[aop]: https://en.wikipedia.org/wiki/Aspect-oriented_programming

What we see above is this pattern: We want to decorate a method with some functionality. Instead of writing a decorator from scratch each time, let's abstract the wrapping into a combinator that makes an ES.later method decorator:

{:lang="js"}
~~~~~~~~
const after = (...fns) =>
  function (target, name, descriptor) {
    const method = descriptor.value;

    descriptor.value = function (...args) {
      const value = method.apply(this, args);

      for (let fn of fns) {
        fn.apply(this, args);
      }
      return value;
    }
  }
~~~~~~~~

Now we could write:

{:lang="js"}
~~~~~~~~
const persists = after(function () { this.persist(); });
~~~~~~~~

Or we could write:

{:lang="js"}
~~~~~~~~
const persists = after(Todo.prototype.persist);
~~~~~~~~

Or we could even write these things inline:

{:lang="js"}
~~~~~~~~
class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false
  }

  @after(Todo.prototype.persist)
  do () {
    this.done = true;
  }

  @after(Todo.prototype.persist)
  undo () {
    this.done = false;
  }

  @after(Todo.prototype.persist)
  setName (name) {
    this.name = name;
  }

  persist () {
    console.log(`persisting ${this.name}`);
  }
}
~~~~~~~~

`Todo.prototype.persist` is a little clunky. We could special-case `after` to allow us to write `@after('persist')` as some libraries do, but the beauty of combinators is that they, well, *combine*. Recall [`send`](#send). It's perfect for this:

{:lang="js"}
~~~~~~~~
const send = (methodName, ...args) =>
  (instance) => instance[methodName].apply(instance, args);

class Todo {
  constructor (name) {
    this.name = name || 'Untitled';
    this.done = false
  }

  @after(send('persist'))
  do () {
    this.done = true;
  }

  @after(send('persist'))
  undo () {
    this.done = false;
  }

  @after(send('persist'))
  setName (name) {
    this.name = name;
  }

  persist () {
    console.log(`persisting ${this.name}`);
  }
}
~~~~~~~~

`after` is a combinator that makes ES.later method decorators, and it's handy for separating concerns.
## Before Method Advice {#before}

Just as we often wish to decorate a method with [after advice](#after), we also often wish to decorate methods with some behaviour that happens before a method is invoked. The canonical (and greatly overused) example is logging invocations. But let's consider another example, a `Person`:

[after advice]: #after

{:lang="js"}
~~~~~~~~
const firstName = Symbol('firstName'),
      lastName = Symbol('lastName');

class Person {
  constructor (first, last) {
    this[firstName] = first;
    this[lastName] = last;
  }

  fullName () {
    return this[firstName] + " " + this[lastName];
  }

  rename (first, last) {
    this[firstName] = first;
    this[lastName] = last;
    return this;
  }
};
~~~~~~~~

What if we wish to make `rename` an undoable action? Let's add a stack. For reasons known only to a secret cabal of enterprisey architects, we wish to make the undo stack something that is lazily initialized, like this:

{:lang="js"}
~~~~~~~~
const firstName = Symbol('firstName'),
      lastName = Symbol('lastName'),
      undoStack = Symbol('undoStack'),
      redoStack = Symbol('redoStack');

class Person {
  constructor (first, last) {
    this[firstName] = first;
    this[lastName] = last;
  }

  fullName () {
    return this[firstName] + " " + this[lastName];
  }

  rename (first, last) {
    this[undoStack] || (this[undoStack] = []);
    this[undoStack].push({
      [firstName]: this[firstName],
      [lastName]: this[lastName]
    });
    this[firstName] = first;
    this[lastName] = last;
    return this;
  }

  undo () {
    this[undoStack] || (this[undoStack] = []);
    let oldState = this[undoStack].pop();

    if (oldState != null) Object.assign(this, oldState);
    return this;
  }
};

const b = new Person('barak', 'obama');

b.rename('Barak', 'Obama');
b.fullName()
  //=> 'Barak Obama'

b.undo();
b.fullName()
  //=> 'barak obama'
~~~~~~~~

We can follow the same pattern as we did with [after advice]: Extract the common functionality into a decorator. We'll write the `before` combinator to help:

{:lang="js"}
~~~~~~~~
const before = (...fns) =>
  function (target, name, descriptor) {
    const method = descriptor.value;

    descriptor.value = function (...args) {
      for (let fn of fns) {
        fn.apply(this, args);
      }
      return method.apply(this, args);
    }
  }

const firstName = Symbol('firstName'),
      lastName = Symbol('lastName'),
      undoStack = Symbol('undoStack'),
      redoStack = Symbol('redoStack');

const usingUndoStack = before(function () {
  this[undoStack] || (this[undoStack] = []);
})

class Person {
  constructor (first, last) {
    this[firstName] = first;
    this[lastName] = last;
  }

  fullName () {
    return this[firstName] + " " + this[lastName];
  }

  @usingUndoStack
  rename (first, last) {
    this[undoStack].push({
      [firstName]: this[firstName],
      [lastName]: this[lastName]
    });
    this[firstName] = first;
    this[lastName] = last;
    return this;
  }

  @usingUndoStack
  undo () {
    let oldState = this[undoStack].pop();

    if (oldState != null) Object.assign(this, oldState);
    return this;
  }
};

const b = new Person('barak', 'obama')
b.rename('Barak', 'Obama')
console.log(b.fullName())
b.undo()
console.log(b.fullName())
~~~~~~~~

We could, of course, also abstract functionality into a method that we invoke with `@after(send('usingUndoStack'))` just as we did with our [after advice] examples.
## Provided and Unless {#provided}

Neither the [before](#before) and [after](#after) ES.later method decorators actually terminate evaluation without throwing something. Normal execution always results in the base method being evaluated. The `provided` and `unless` recipes are combinators that produce method decorators that apply a precondition to evaluating the base method body.

The `provided` combinator turns a function into an ES.later method decorator. The function (or functions) is passed the method arguments before the base method, and it must evaluate to truthy for the base method to be evaluated. The `unless` combinator does the same thing, but the logic is reversed, the decorating function must not evaluate to truthy:

{:lang="js"}
~~~~~~~~
const provided = (...fns) =>
  function (target, name, descriptor) {
    const method = descriptor.value;

    descriptor.value = function (...args) {
      for (let fn of fns) {
        const result = fn.apply(this, args);

        if (!result) return;
      }
      return method.apply(this, args);
    }
  }

const unless = (...fns) =>
  function (target, name, descriptor) {
    const method = descriptor.value;

    descriptor.value = function (...args) {
      for (let fn of fns) {
        const result = fn.apply(this, args);

        if (result) return;
      }
      return method.apply(this, args);
    }
  }
~~~~~~~~

`provided` can be used to check that non-empty strings are provided for names:[^beware]

{:lang="js"}
~~~~~~~~
const firstName = Symbol('firstName'),
      lastName = Symbol('lastName');

const nonEmptyStrings = (...strs) =>
  strs.reduce((truth, str) =>
    truth
      && (str instanceof String || typeof str === 'string')
      && str !== '',
  true);

class Person {
  constructor (first, last) {
    this[firstName] = first;
    this[lastName] = last;
  }

  fullName () {
    return this[firstName] + " " + this[lastName];
  }

  @provided(nonEmptyStrings)
  rename (first, last) {
    this[firstName] = first;
    this[lastName] = last;
    return this;
  }
};

const b = new Person('barak', 'obama')
b.rename('Barak', 'Obama')
console.log(b.fullName())
b.undo()
console.log(b.fullName())
~~~~~~~~

[^beware]: Beware, validating names is a stygian task. Read [falsehoods programmers believe about names](http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/) before proceeding with ideas like this in production. For example, many people do NOT have both a first and last name.

You may wonder why we didn't decorate the `constructor`. Alas, we can't use a method decorator on a constructor, because it isn't a method. It just *looks like one*. It's still a constructor function, and if we want to modify it, we have to either write a class decorator, or punt all the work of construction to a method, like this:

{:lang="js"}
~~~~~~~~
class Person {
  constructor (first, last) {
    this.rename(first, last);
  }

  fullName () {
    return this[firstName] + " " + this[lastName];
  }

  @provided(nonEmptyStrings)
  rename (first, last) {
    this[firstName] = first;
    this[lastName] = last;
    return this;
  }
};
~~~~~~~~

There are many variations on decorators that check preconditions for methods. For example, a decorator can be made that throws an exception if the preconditions fail rather than silently skipping the method invocation.

We can use these patterns in many ways. JavaScript is very flexible!
## Method Advice {#method-advice}

We've [previously](#stateful-method-decorators) looked at method decorators like this:

{:lang="js"}
~~~~~~~~
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
~~~~~~~~

We also saw that if our tooling supports ES.later[^ESdotlater] decorators, we can write:

{:lang="js"}
~~~~~~~~
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
~~~~~~~~

[^ESdotlater]: By "ES.later," we mean some future version of ECMAScript that is likely to be approved eventually, but for the moment exists only in transpilers like [Babel](http://babeljs.io). Obviously, using any ES.later feature in production is a complex decision requiring many more considerations than can be enumerated in a book.

The `wrapWith` function takes an ordinary method decorator and turns it into an ES.later method decorator.

### what question do method decorators answer?

ES.later method decorators put the decorations right next to the method body. This makes it easy to answer the question "What is the precise behaviour of this method?"

But sometimes, this is not what you want. Consider a responsibility like authentication. Let's imagine that we validate permissions in our model classes. We might write something like this:

{:lang="js"}
~~~~~~~~
const wrapWith = (decorator) =>
  function (target, name, descriptor) {
    descriptor.value = decorator(descriptor.value);
  }

const mustBeMe = (method) =>
  function (...args) {
    if (currentUser() && currentUser().person().equals(this))
      return method.apply(this, args);
    else throw new PermissionsException("Must be me!");
  }

class Person {

  @wrapWith(mustBeMe)
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

  @wrapWith(mustBeMe)
  setAge (age) {
    this.age = age;
  }

  @wrapWith(mustBeMe)
  age () {
    return this.age;
  }

};
~~~~~~~~

(Obviously real permissions systems involve roles and all sorts of other important things.)

Now we can look at `setName` and see that users can only set their own name, likewise if we look at `setAge`, we see that users can only set their own age.

In a tiny toy example the next question is easy to answer: *What methods can only be invoked by the person themselves?* We see at a glance that the answer is `setName`, `setAge`, and `age`.

But as classes grow, this becomes more difficult to answer. This especially becomes difficult if we decompose classes using mixins. For example, what if `setAge` and `age` come from a [class mixin](#class-mixins):

{:lang="js"}
~~~~~~~~
const Person = HasAge(class {

  @wrapWith(mustBeMe)
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

});
~~~~~~~~

Are the methods provided by `HasAge` wrapped with `mustBeMe`? Quite possibly not, because the mixin is responsible for defining the behaviour. It's up to the model class to decide the permissions required. But how would you know if they were?

Method decorators make it easy to answer the question "what is the behaviour of this method?" But they don't make it easy to answer the question "what methods share this behaviour?"

That question matters, because when decomposing responsibilities, we often decide that a *cross-cutting* responsibility like permissions should be distinct from an implementation responsibility like storing a name.

### cross-cutting method decorators

There is another way to decorate methods: We can decorate multiple methods in a single declaration. This is called providing *method advice*.

In JavaScript, we can implement method advice by decorating the entire class. We already have a combinator for making class mixins, it's a  function that takes a class as an argument and returns the same or different class. We can use the same technique to write a class decorator that decorates one or more methods of the class being passed in. (We'll use ES.later syntax, but it works just as well with functional syntax):

{:lang="js"}
~~~~~~~~
const aroundAll = (behaviour, ...methodNames) =>
  (clazz) => {
    for (let methodName of methodNames)
      Object.defineProperty(clazz.prototype, property, {
        value: behaviour(clazz.prototype[methodName]),
        writable: true
      });
    return clazz;
  }

@HasAge
@aroundAll(mustBeMe, 'setName', 'setAge', 'age')
class Person {

  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

};
~~~~~~~~

Now when you look at `setName`, you don't see what permissions apply. However, when we look at `@aroundAll(mustBeMe, 'setName', 'setAge', 'age')`, we see that we're wrapping `setName`, `setAge` and `age` with `mustBeMe`.

This focuses the responsibility for permissions in one place. Of course, we could make things simpler. For one thing, some actions are only performed *before* a method, and some only *after* a method. We can make class decorators that work just like our [before](#before) and [after](#after) method decorators:

{:lang="js"}
~~~~~~~~
const beforeAll = (behaviour, ...methodNames) =>
  (clazz) => {
    for (let methodName of methodNames) {
      const method = clazz.prototype[methodName];

      Object.defineProperty(clazz.prototype, property, {
        value: function (...args) {
          behaviour.apply(this, args);
          return method.apply(this, args);
        },
        writable: true
      });
    }
    return clazz;
  }

const afterAll = (behaviour, ...methodNames) =>
  (clazz) => {
    for (let methodName of methodNames) {
      const method = clazz.prototype[methodName];

      Object.defineProperty(clazz.prototype, property, {
        value: function (...args) {
          const returnValue = method.apply(this, args);

          behaviour.apply(this, args);
          return returnValue;
        },
        writable: true
      });
    }
    return clazz;
  }
~~~~~~~~

Precondition checks like `mustBeMe` are good candidates for `beforeAll`. Here's `mustBeLoggedIn` and `mustBeMe` set up to use `beforeAll`. They're far simpler since `beforeAll` handles the wrapping:

{:lang="js"}
~~~~~~~~
const mustBeLoggedIn = () => {
    if (currentUser() == null)
      throw new PermissionsException("Must be logged in!");
  }

const mustBeMe = () => {
    if (currentUser() == null || !currentUser().person().equals(this))
      throw new PermissionsException("Must be me!");
  }

@HasAge
@beforeAll(mustBeMe, 'setName', 'setAge', 'age')
@beforeAll(mustBeLoggedIn, 'fullName')
class Person {

  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }

};
~~~~~~~~

This style of moving the responsibility for decorating methods to a single declaration will appear familiar to Ruby on Rails developers. As you can see, it does not require "deep magic" or complex libraries, it is a pattern that can be written out in just a few lines of code.

Mind you, there's always room for polish and gold plate. We could enhance `beforeAll`, `afterAll`, and `aroundAll` to include conveniences like regular expressions to match method names, or special declarations like `except:` or `only:` if we so desired.

Although decorating methods in bulk has appeared in other languages and paradigms, it's not something special and alien to JavaScript, it's really the same pattern we see over and over again: Programming by composing small and single-responsibility entities, and using functions to transform and combine the entities into their final form.

### a word about es6

If we don't want to use ES.later decorators, we can use the exact same decorators as *ordinary functions*, like this:

{:lang="js"}
~~~~~~~~
const mustBeLoggedIn = () => {
    if (currentUser() == null)
      throw new PermissionsException("Must be logged in!");
  }

const mustBeMe = () => {
    if (currentUser() == null || !currentUser().person().equals(this))
      throw new PermissionsException("Must be me!");
  }

const Person =
  HasAge(
  beforeAll(mustBeMe, 'setName', 'setAge', 'age')(
  beforeAll(mustBeLoggedIn, 'fullName')(
    class {
      setName (first, last) {
        this.firstName = first;
        this.lastName = last;
      }

      fullName () {
        return this.firstName + " " + this.lastName;
      }
    }
  )
  )
);
~~~~~~~~

Composition could also help:

{:lang="js"}
~~~~~~~~
const mustBeLoggedIn = () => {
    if (currentUser() == null)
      throw new PermissionsException("Must be logged in!");
  }

const mustBeMe = () => {
    if (currentUser() == null || !currentUser().person().equals(this))
      throw new PermissionsException("Must be me!");
  }

const Person = compose(
  HasAge,
  beforeAll(mustBeMe, 'setName', 'setAge', 'age'),
  beforeAll(mustBeLoggedIn, 'fullName'),
)(class {
  setName (first, last) {
    this.firstName = first;
    this.lastName = last;
  }

  fullName () {
    return this.firstName + " " + this.lastName;
  }
});
~~~~~~~~
