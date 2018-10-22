# Con Panna: Composing Class Behaviour

Because prototypes are just objects, and because "classes" actually use prototypes under the hood, we can use all of the techniques we've learned about working with objects, when working with prototypes.

## Extending Classes with Mixins

We've seen that a "class" is simply a constructor function that is associated with a prototype, and that the `class` keyword is a declarative way to write our own constructor functions and prototypes. When we use the `new` keyword, we are invoking a mechanism that creates a new object that delegates to a prototype, just like `Object.create`, and then the constructor function takes over and performs any initialization we desire.

Because "classes" use the exact same model of delegating behaviour to prototypes, all the things we learned about prototypes apply to classes. We saw that we can create "subclasses" by chaining prototypes.

We can also share behaviour between classes in a more flexible way by mixing functionality into classes. This is the exact same thing as mixing functionality into prototypes, of course.

Recall `Person`:
```js
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
    }

    const misterRogers = new Person('Fred', 'Rogers');
    misterRogers.fullName()
      //=> Fred Rogers
```
We might be building some enterprisey thing and need `Manager` and `Worker`:
```js
    class Manager extends Person {
      constructor (first, last) {
        super(first, last)
      }
      addReport (report) {
        this.reports().add(report);
        return this;
      }
      removeReport (report) {
        this.reports().delete(report);
        return this;
      }
      reports () {
        return this._reports || (this._reports = new Set());
      }
    }

    class Worker extends Person {
      constructor (first, last) {
        super(first, last);
      }
      setManager (manager) {
        this.removeManager();
        this.manager = manager;
        manager.addReport(this);
        return this;
      }
      removeManager () {
        if (this.manager) {
          this.manager.removeReport(this);
          this.manager = undefined;
        }
        return this;
      }
    }
```
This works for our company, so well that we grow and develop the dreaded "Middle Manager," who both manages people and has a manager of their own. We could subclass `Manager` with `MiddleManager`, but how do `Worker` and `MiddleManager` share the functionality for having a manager?

With a mixin, of course:
```js
    const HasManager = {
      function setManager (manager) {
        this.removeManager();
        this.manager = manager;
        manager.addReport(this);
        return this;
      },
      function removeManager () {
        if (this.manager) {
          this.manager.removeReport(this);
          this.manager = undefined;
        }
        return this;
      }
    };
    
    class Manager extends Person {
      constructor (first, last) {
        super(first, last)
      }
      addReport (report) {
        this.reports().add(report);
        return this;
      }
      removeReport (report) {
        this.reports().delete(report);
        return this;
      }
      reports () {
        return this._reports || (this._reports = new Set());
      }
    }
    
    class MiddleManager extends Manager {
      constructor (first, last) {
        super(first, last);
      }
    }
    Object.assign(MiddleManager.prototype, HasManager);
    
    class Worker extends Person {
      constructor (first, last) {
        super(first, last);
      }
    }
    Object.assign(Worker.prototype, HasManager);
```    
We can mix functionality into the prototypes of "classes" just as easily as we can mix functionality directly into objects, because prototypes *are* objects, and JavaScript builds its "classes" out of prototypes.

Were classes "something else," like they are in other languages, we would gain many advantages that we do not enjoy in JavaScript, but we would also give up the flexibility of being able to use the same tools and techniques on prototypes that we do on objects.

## Functional Mixins

In [Extending Classes with Mixins](#classes-and-mixins), we saw that you can emulate "mixins" using `Object.assign` on classes. We'll revisit this subject now and spend more time looking at mixing functionality into classes.

First, a quick recap: In JavaScript, a "class" is implemented as a constructor function and its prototype, whether you write it directly, or use the `class` keyword. Instances of the class are created by calling the constructor with `new`. They "inherit" shared behaviour from the constructor's `prototype` property.[^delegate]

[^delegate]: A much better way to put it is that objects with a prototype *delegate* behaviour to their prototype (and that may in turn delegate behaviour to its prototype if it has one, and so on).

### the object mixin pattern

One way to share behaviour scattered across multiple classes, or to untangle behaviour by factoring it out of an overweight prototype, is to extend a prototype with a *mixin*.

Here's a class of todo items:
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
}
```
And a "mixin" that is responsible for colour-coding:
```js
const Coloured = {
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },
  getColourRGB () {
    return this.colourCode;
  }
};
```
Mixing colour coding into our Todo prototype is straightforward:
```js
Object.assign(Todo.prototype, Coloured);

new Todo('test')
  .setColourRGB({r: 1, g: 2, b: 3})
  //=> {"name":"test","done":false,"colourCode":{"r":1,"g":2,"b":3}}
```
So far, very easy and very simple. This is a *pattern*, a recipe for solving a certain problem using a particular organization of code.

### functional mixins

The object mixin we have above works properly, but our little recipe had two distinct steps: Define the mixin and then extend the class prototype. Angus Croll pointed out that it's more elegant to define a mixin as a function rather than an object. He calls this a [functional mixin][fm]. Here's `Coloured` again, recast in functional form:
```js
const Coloured = (target) =>
  Object.assign(target, {
    setColourRGB ({r, g, b}) {
      this.colourCode = {r, g, b};
      return this;
    },
    getColourRGB () {
      return this.colourCode;
    }
  });

Coloured(Todo.prototype);
```
We can make ourselves a *factory function* that also names the pattern:
```js
const FunctionalMixin = (behaviour) =>
  target => Object.assign(target, behaviour);
```
This allows us to define functional mixins neatly:
```js
const Coloured = FunctionalMixin({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },
  getColourRGB () {
    return this.colourCode;
  }
});
```
### enumerability

If we look at the way `class` defines prototypes, we find that the methods defined are not enumerable by default. This works around a common error where programmers iterate over the keys of an instance and fail to test for `.hasOwnProperty`.

Our object mixin pattern does not work this way, the methods defined in a mixin *are* enumerable by default, and if we carefully defined them to be non-enumerable, `Object.assign` wouldn't mix them into the target prototype, because `Object.assign` only assigns enumerable properties.

And thus:
```js
Coloured(Todo.prototype)

const urgent = new Todo("finish blog post");
urgent.setColourRGB({r: 256, g: 0, b: 0});

for (let property in urgent) console.log(property);
  // =>
    name
    done
    colourCode
    setColourRGB
    getColourRGB
```
As we can see, the `setColourRGB` and `getColourRGB` methods are enumerated, although the `do` and `undo` methods are not. This can be a problem with naïve code: we can't always rewrite all the *other* code to carefully use `.hasOwnProperty`.

One benefit of functional mixins is that we can solve this problem and transparently make mixins behave like `class`:
```js
const FunctionalMixin = (behaviour) =>
  function (target) {
    for (let property of Reflect.ownKeys(behaviour))
      if (!target[property])
        Object.defineProperty(target, property, {
          value: behaviour[property],
          writable: true
        })
    return target;
  }
```
Writing this out as a pattern would be tedious and error-prone. Encapsulating the behaviour into a function is a small win.

### mixin responsibilities

Like classes, mixins are metaobjects: They define behaviour for instances. In addition to defining behaviour in the form of methods, classes are also responsible for initializing instances. But sometimes, classes and metaobjects handle additional responsibilities.

For example, sometimes a particular concept is associated with some well-known constants. When using a class, can be handy to namespace such values in the class itself:
```js
class Todo {
  constructor (name) {
    this.name = name || Todo.DEFAULT_NAME;
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
}

Todo.DEFAULT_NAME = 'Untitled';

// If we are sticklers for read-only constants, we could write:
// Object.defineProperty(Todo, 'DEFAULT_NAME', {value: 'Untitled'});
```
We can't really do the same thing with simple mixins, because all of the properties in a simple mixin end up being mixed into the prototype of instances we create by default. For example, let's say we want to define `Coloured.RED`, `Coloured.GREEN`, and `Coloured.BLUE`. But we don't want any specific coloured instance to define `RED`, `GREEN`, or `BLUE`.

Again, we can solve this problem by building a functional mixin. Our `FunctionalMixin` factory function will accept an optional dictionary of read-only mixin properties:
```js
function FunctionalMixin (behaviour, sharedBehaviour = {}) {
  const instanceKeys = Reflect.ownKeys(behaviour);
  const sharedKeys = Reflect.ownKeys(sharedBehaviour);

  function mixin (target) {
    for (let property of instanceKeys)
      if (!target[property])
        Object.defineProperty(target, property, {
          value: behaviour[property],
          writable: true
        });
    return target;
  }
  for (let property of sharedKeys)
    Object.defineProperty(mixin, property, {
      value: sharedBehaviour[property],
      enumerable: sharedBehaviour.propertyIsEnumerable(property)
    });
  return mixin;
}
```
And now we can write:
```js
const Coloured = FunctionalMixin({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },
  getColourRGB () {
    return this.colourCode;
  }
}, {
  RED:   { r: 255, g: 0,   b: 0   },
  GREEN: { r: 0,   g: 255, b: 0   },
  BLUE:  { r: 0,   g: 0,   b: 255 },
});

Coloured(Todo.prototype)

const urgent = new Todo("finish blog post");
urgent.setColourRGB(Coloured.RED);

urgent.getColourRGB()
  //=> {"r":255,"g":0,"b":0}
```
### mixin methods

Such properties need not be values. Sometimes, classes have methods. And likewise, sometimes it makes sense for a mixin to have its own methods. One example concerns `instanceof`.

In earlier versions of ECMAScript, `instanceof` is an operator that checks to see whether the prototype of an instance matches the prototype of a constructor function. It works just fine with "classes," but it does not work "out of the box" with mixins:
```js
urgent instanceof Todo
  //=> true

urgent instanceof Coloured
  //=> false
```
To handle this and some other issues where programmers are creating their own notion of dynamic types, or managing prototypes directly with `Object.create` and `Object.setPrototypeOf`, ECMAScript 2015 provides a way to override the built-in `instanceof` behaviour: An object can define a method associated with a well-known symbol, `Symbol.hasInstance`.

We can test this quickly:[^but]

[^but]: This may **not** work with various transpilers and other incomplete ECMAScript 2015 implementations. Check the documentation. For example, you must enable the "high compliancy" mode in [BabelJS](http://babeljs.io). This is off by default to provide the highest possible performance for code bases that do not need to use features like this.
```js
Coloured[Symbol.hasInstance] = (instance) => true
urgent instanceof Coloured
  //=> true
{} instanceof Coloured
  //=> true
```
Of course, that is not semantically correct. But using this technique, we can write:
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

urgent instanceof Coloured
  //=> true
{} instanceof Coloured
  //=> false
```
Do you need to implement `instanceof`? Quite possibly not. "Rolling your own polymorphism" is usually a last resort. But it can be handy for writing test cases, and a few daring framework developers might be working on multiple dispatch and pattern-matching for functions.

### summary

The charm of the object mixin pattern is its simplicity: It really does not need an abstraction wrapped around an object literal and `Object.assign`.

However, behaviour defined with the mixin pattern is *slightly* different than behaviour defined with the `class` keyword. Two examples of these differences are enumerability and mixin properties (such as constants and mixin methods like `[Symbol.hasInstance]`).

Functional mixins provide an opportunity to implement such functionality, at the cost of some complexity in the `FunctionalMixin` function that creates functional mixins.

As a general rule, it's best to have things behave as similarly as possible in the domain code, and this sometimes does involve some extra complexity in the infrastructure code. But that is more of a guideline than a hard-and-fast rule, and for this reason there is a place for both the object mixin pattern *and* functional mixins in JavaScript.

[fm]: https://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/ "A fresh look at JavaScript Mixins"
[Flight]: http://flightjs.github.io/

## Emulating Multiple Inheritance

If you want to mix behaviour into a class, mixins do the job very nicely. But sometimes, people want more. They want **multiple inheritance**. Meaning, what they really want is to create a new class that inherits from both `Todo` *and* from `Coloured`.

If JavaScript had multiple inheritance, we could accomplish this by extending a class with more than one superclass:
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

class Coloured {
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  }

  getColourRGB () {
    return this.colourCode;
  }
}

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
This hypothetical `TimeSensitiveTodo` extends both `Todo` and `Coloured`, and it overrides `toHTML` from `Todo` as well as overriding `getColourRGB` from `Coloured`.

### subclass factories

However, JavaScript does not have "true" multiple inheritance, and therefore this code does not work. But we can simulate multiple inheritance for cases like this. The way it works is to step back and ask ourselves, "What would we do if we didn't have mixins or multiple inheritance?"

The answer is, we'd force a square multiple inheritance peg into a round single inheritance hole, like this:
```js
class Todo {
  // ...
}

class ColouredTodo extends Todo {
  // ...
}

class TimeSensitiveTodo extends ColouredTodo {
  // ...
}
```
By making `ColouredTodo` extend `Todo`, `TimeSensitiveTodo` can extend `ColouredTodo` and override methods from both. This is exactly what most programmers do, and we know that it is an anti-pattern, as it leads to duplicated class behaviour and deep class hierarchies.

But.

What if, instead of manually creating this hierarchy, we use our simple mixins to do the work for us? We can take advantage of the fact that [classes are expressions](http://raganwald.com/2015/06/04/classes-are-expressions.html), like this:
```js
let Coloured = FunctionalMixin({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },

  getColourRGB () {
    return this.colourCode;
  }
});

let ColouredTodo = Coloured(class extends Todo {});
```
Thus, we have a `ColouredTodo` that we can extend and override, but we also have our `Coloured` behaviour in a mixin we can use anywhere we like without duplicating its functionality in our code. The full solution looks like this:
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

let Coloured = FunctionalMixin({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },

  getColourRGB () {
    return this.colourCode;
  }
});

let ColouredTodo = Coloured(class extends Todo {});

let yellow = {r: 'FF', g: 'FF', b: '00'},
    red    = {r: 'FF', g: '00', b: '00'},
    green  = {r: '00', g: 'FF', b: '00'},
    grey   = {r: '80', g: '80', b: '80'};

let oneDayInMilliseconds = 1000 * 60 * 60 * 24;

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

let task = new TimeSensitiveTodo('Finish JavaScript Allongé', Date.now() + oneDayInMilliseconds);

task.toHTML()
  //=> <span style="color: #FFFF00;">Finish JavaScript Allongé</span>
```
The key snippet is `let ColouredTodo = Coloured(class extends Todo {});`, it turns behaviour into a subclass that can be extended and overridden.

### subclass factories

We can turn this pattern into a function:
```js
const SubclassFactory = (behaviour) => {
  let mixBehaviourInto = FunctionalMixin(behaviour);

  return (superclazz) => mixBehaviourInto(class extends superclazz {});
}
```
Using `SubclassFactory`, we wrap the class we want to extend, instead of the class we are declaring. Like this:
```js
const SubclassFactory = (behaviour) => {
  let mixBehaviourInto = FunctionalMixin(behaviour);

  return (superclazz) => mixBehaviourInto(class extends superclazz {});
}

const ColouredAsWellAs = SubclassFactory({
  setColourRGB ({r, g, b}) {
    this.colourCode = {r, g, b};
    return this;
  },

  getColourRGB () {
    return this.colourCode;
  }
});

class TimeSensitiveTodo extends ColouredAsWellAs(ToDo) {
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
The syntax of `class TimeSensitiveTodo extends ColouredAsWellAs(ToDo)` says exactly what we mean: We are extending our `Coloured` behaviour as well as extending `ToDo`.[^fagnani]

[^fagnani]: Justin Fagnani named this pattern "subclass factory" in his essay ["Real" Mixins with JavaScript Classes](http://justinfagnani.com/2015/12/21/real-mixins-with-javascript-classes/). It's well worth a read, and his implementation touches on other matters such as optimizing performance on modern JavaScript engines.

## Preventing Property Conflicts

When mixing behaviour onto classes, (and equally, when chaining prototypes, or extending classes in a hierarchy), we are engaging in [open recursion][or]. The methods in each mixin (or prototype in a chain) all have the same context, and therefore refer to the same properties.

[or]: https://en.wikipedia.org/wiki/Open_recursion#Open_recursion

When chaining prototypes or extending classes, this does not typically result in two functions accidentally using the same property for two different purposes. For example, if we write:
```js
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

class Bibliophile extends Person {
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._books || (this._books = []);
  }
}
```
And later we wanted to write:
```js
class Author extends Bibliophile {
  // ...
}
```
It is very unlikely that we would attempt to use the same `._books` property to refer to both the books an author writes and the books a bibliophile collects. For some odd reason, our ontology has it that all authors are also bibliophiles, so it's natural that we would inspect the `Bibliophile` superclass when designing `Author`, and all of our tests for `Author` would be performed on objects that are instances of `Bibliophile`, by definition.

However, this is not the case for mixins. If we wrote:
```js
const IsBibliophile = {
  addToCollection (name) {
    this.collection().push(name);
    return this;
  },
  collection () {
    return this._books || (this._books = []);
  }
};
```
And a colleague wrote:
```js
const IsAuthor = {
  addBook (name) {
    this.books().push(name);
    return this;
  },
  books () {
    return this._books || (this._books = []);
  }
};
```
This code could easily work for months or years. `IsAuthor` could be tested independently of `Bibliophile`, and both would appear to behave correctly. Until the fateful day someone wrote something like:
```js
class BookLovingAuthor extends Person {
}

Object.assign(BookLovingAuthor.prototype, IsBibliophile, IsAuthor);

new BookLovingAuthor('Isaac', 'Asimov')
  .addBook('I Robot')
  .addToCollection('The Mysterious Affair at Styles')
  .collection()
    //=> ["I Robot","The Mysterious Affair at Styles"]
```
And bam! We have a property conflict: The books Isaac Asimov has written and collects have become intermingled, because the two mixins refer to the same property.

### decoupling mixins with symbols

The simplest way to avoid these property conflicts is to use symbols for property names:
```js
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

const IsAuthor = (function () {
  const books = Symbol();

  return {
    addBook (name) {
      this.books().push(name);
      return this;
    },
    books () {
      return this[books] || (this[books] = []);
    }
  };
})();

const IsBibliophile = (function () {
  const books = Symbol();

  return {
    addToCollection (name) {
      this.collection().push(name);
      return this;
    },
    collection () {
      return this[books] || (this[books] = []);
    }
  };
})();

class BookLovingAuthor extends Person {
}

Object.assign(BookLovingAuthor.prototype, IsBibliophile, IsAuthor);

new BookLovingAuthor('Isaac', 'Asimov')
  .addBook('I Robot')
  .addToCollection('The Mysterious Affair at Styles')
  .collection()
    //=> ["The Mysterious Affair at Styles"]
  .books().
    //=> ["I Robot"]
```
Using symbols for property keys eliminates property conflicts between mixins.

## Reducing Coupling 

When classes are built in a hierarchy, or mixins are distributed across a code base, coupling arises over time. Typically, as a code base evolves, each iteration of programmer uses whatever methods or properties have been made available by the accumulated efforts of previous iterations.

As time goes on, the number of methods and properties increases, and each new piece of behaviour touches more and more methods and properties. When it comes time to refactor the code base, it can be very difficult to tease behaviour apart, since so many pieces naturally end up depending on each other.

One way to resist this natural tendency toward coupling is by making sure that each metaobject exposes only the methods it confers upon its receivers. All other methods and properties should be kept private.

Note that making properties private is not an ideological issue: It's not a question of "purity in OO theory." It's a practical issue: It's a question of minimizing the surface area of the metaobject in order to minimize the ways in which it can become coupled to other objects.

### using symbols to reduce coupled properties

We have seen that using symbols as property keys prevents mixins from accidentally sharing the same property name for different purposes. They can also help prevent programmers from *deliberately* using the same property name for different purposes.

Here's why we care about that. Consider:
```js
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
}

class Bibliophile extends Person {
  constructor (first, last) {
    super(first, last);
    this._books = [];
  }
  addToCollection (name) {
    this._books.push(name);
    return this;
  }
  hasInCollection (name) {
    return this._books.indexOf(name) >= 0;
  }
}

const bezos = new Bibliophile('jeff', 'bezos')
  .addToCollection("The Everything Store: Jeff Bezos and the Age of Amazon")
  .hasInCollection("Matthew and the Wellington Boots")
    //=> false
    
bezos
  .hasInCollection("The Everything Store: Jeff Bezos and the Age of Amazon")
    //=> true
```
Note that `._books` is an array. Now consider:
```js
class BookGlutten extends Bibliophile {
  buyInBulk (...names) {
    this.books().push(...names);
    return this;
  }
}
```
Book gluttons can buy books in bulk, ordinary bibliophiles cannot. So far, so good. But we have a very naïve implementation of book collections: an array is a linear data structure, the performance of `hasInCollection` is order `n`. The moment we have a bibliophile with a really large collection, the operation becomes excruciatingly slow.

Simplifying greatly, what if we refactor `Bibliophile` to use a `Set`?
```js
class Bibliophile extends Person {
  constructor (first, last) {
    super(first, last);
    this._books = new Set();
  }
  addToCollection (name) {
    this._books.add(name);
    return this;
  }
  hasInCollection (name) {
    return this._books.has(name);
  }
}
```
Much faster, but we just broke our `BookGlutten` subclass. This is a very small and contrived example, but the phenomenon is very real, and the larger the class hierarchy, the more it occurs. The author of our `BookGlutton` subclass coupled `BookGlutton` to an implementation detail of `Bibliophile`. That's a "feature" of open recursion, but it is far wiser to prevent this from happening.

Naturally, we can use the same technique to prevent deliberate coupling of subclasses that we used to prevent accidental property conflicts: Symbols.
```js
const Bibliophile = (function () {
  const books = Symbol("books");
  
  return class Bibliophile extends Person {
    constructor (first, last) {
      super(first, last);
      this[books] = [];
    }
    addToCollection (name) {
      this[books].push(name);
      return this;
    }
    hasInCollection (name) {
      return this[books].indexOf(name) >= 0;
    }
  }
})();
```
Now anyone subclassing `Bibliophile` is strongly discouraged from directly accessing the "books" property:
```js
class BookGlutten extends Bibliophile {
  buyInBulk (...names) {
    for (let name of names) {
      this.addToCollection(name);
    }
    return this;
  }
}
```
Problem solved.
