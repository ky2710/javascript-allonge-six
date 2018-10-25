# Finish the Cup: Constructors and Classes

As discussed in [Encapsulating State](main_2_objects.md#encapsulating-state-with-closures), JavaScript objects are very simple, yet the combination of objects, functions, and closures can create powerful data structures. We've also seen how to use [Metaobjects](main_4_metaobjects.md) to separate behaviour from domain properties, and to share functionality amongst many different objects. And finally, we saw that one particular type of metaobject, a *prototype*, provides us with a robust model for delegation.

In this section, we will return to prototypes, and see how to use JavaScript's `class` keyword to write one style of "object-oriented" JavaScript.

## Constructors and `new`

> This is an important but subtle distinction: there's really no such thing as "constructor functions", but rather **construction calls of functions.** : [YDK-JS](https://github.com/kiyounglee/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#new-binding)

Let's strip a function down to the bare essentials:
```js
    const Ur = function () {};
```
Or the equivalent:
```js
    function Ur () {};
```
This doesn't look like it has anything to do with objects and constructing things: It doesn't have an expression that yields a Plain Old JavaScript Object when the function is applied. Yet, there is a way to make an object out of it. Behold the power of the `new` keyword:
```js
    new Ur()
      //=> {}
```
We got an object back! What can we find out about this object?
```js
    new Ur() === new Ur()
      //=> false
```
Every time we call `new` with a function and get an object back, we get a unique object. We could call these "Objects created with the `new` keyword," but this would be cumbersome. So we're going to call them *instances*. Instances of what? Instances of the function that creates them. So given `const i = new Ur()`, we say that `i` is an instance of `Ur`.

We also say that `Ur` is the *constructor* of `i`, and that `Ur` is a *constructor function*. Therefore, an instance is an object created by using the `new` keyword on a constructor function, and that function is the instance's constructor.

> An instance is an object created by using the `new` keyword on a constructor function, and that function is the instance's constructor.

### constructors, instances, and prototypes

There's more. Here's something you may not know about functions, every function has a `.prototype` property by default:
```js
    Ur.prototype
      //=> {}
```
We remember [prototypes](#prototypes). What do we know about the prototype property of every function? Let's run our standard test:
```js
    (function () {}).prototype === (function () {}).prototype
      //=> false
```
Every function is initialized with its own unique value for the `.prototype` property. What does it do? Is it related to the prototypes we saw with Metaobjects? Let's try something:
```js
    Ur.prototype.language = 'JavaScript';

    const continent = new Ur();
      //=> {}
    continent.language
      //=> 'JavaScript'
```
That's very interesting! Instances seem to behave as if they *delegate* to their constructors prototype, just as if we'd created them using `Object.create(Ur.prototype)`.

We can actually test this directly:
```js
    Ur.prototype.isPrototypeOf(continent)
      //=> true
```
And we can inspect the prototype of our `continent` directly:
```js
    Object.getPrototypeOf(continent) === Ur.prototype
      //=> true
```
Let's try a few things:
```js
    continent.language = 'CoffeeScript';
    continent
      //=> {language: 'CoffeeScript'}
    continent.language
      //=> 'CoffeeScript'
    Ur.prototype.language
      'JavaScript'
```
You can set elements of an instance, and they "override" the constructor's prototype, but they don't actually change the constructor's prototype. Let's make another instance and try something else.
```js
    const another = new Ur();
      //=> {}
    another.language
      //=> 'JavaScript'
```
New instances don't acquire any changes made to other instances. Makes sense. And:
```js
    Ur.prototype.language = 'Sumerian'
    another.language
      //=> 'Sumerian'
```
Even more interesting: Changing the constructor's prototype changes the behaviour of all of its instances. This *is* the prototype/delegation relationship we have already seen with `Object.create`.

Speaking of prototypes, here's something else that's very interesting:
```js
    continent.constructor
      //=> [Function]

    continent.constructor === Ur
      //=> true
```
Every instance we create with `new` acquires a `constructor` element that is initialized to their constructor function. Objects we don't create with `new` still have a `constructor` element, it's a built-in function:
```js
    {}.constructor
      //=> [Function: Object]
```
If that's true, what about prototypes? Do they have constructors?
```js
    Ur.prototype.constructor
      //=> [Function]
    Ur.prototype.constructor === Ur
      //=> true
```
Very interesting!

### revisiting `this` idea of queues

Let's rewrite our Queue to use `new` and `.prototype`, using `this` and `Object.assign`:
```js
    const Queue = function () {
      Object.assign(this, {
        array: [],
        head: 0,
        tail: -1
      })
    };

    Object.assign(Queue.prototype, {
      pushTail (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead () {
        let value;

        if (!this.isEmpty()) {
          value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty () {
        return this.tail < this.head
      }
    });
```
You recall that when we first looked at `this`, we only covered the case where a function that belongs to an object is invoked. Now we see another case: When a function is invoked by the `new` operator, `this` is set to the new object being created. Thus, our code for `Queue` initializes the queue.

You can see why `this` is so handy in JavaScript: We wouldn't be able to define functions in the prototype that worked on the instance if JavaScript didn't give us an easy way to refer to the instance itself.

### how do constructors compare to `Object.create`?

Let's summarize what we know:

When we use the `new` keyword with a function, we *construct* an object. The function is called with its context (`this`) set to the new object, and the new object delegates behaviour to whatever object is in the function's `.prototype` property.

When we use `Object.create`, we create a new object and that object delegates its behaviour to whatever object we pass to `Object.create`. If we want to do any other initialization with the object, we can do that in a separate step.

Roughly speaking, we could use `Object.create` to emulate the obvious features of the `new` keyword. Let's try it. We'll start with `worksLikeNew`, a function that takes a constructor and some optional arguments, and acts like the `new` keyword:
```js
    function worksLikeNew (constructor, ...args) {
      const instance = Object.create(constructor.prototype);

      instance.constructor = constructor;

      const result = constructor.apply(instance, args);

      return result === undefined ? instance : result;
    }

    function NamedContinent (name) {
      this.name = name;
    }
    NamedContinent.prototype.description = function () { return `A continent named "${this.name}"` };

    const na = worksLikeNew(NamedContinent, "North America");

    na.description()
      //=> A continent named "North America"
```
So do we *need* the `new` keyword, given that we can emulate it? Well, one could argue that we don't *need* multiplication for positive integers:
```js
    const times = (a, b) =>
      a === 0
        ? 0
        : b + times(a-1, b);
```
Programming is a process of choosing and making abstractions, and combining constructor functions with the `new` keyword provides a single abstraction that handles several duties:

- The constructor's prototype provides a metaobject for describing the behaviour of every instance created with the constructor.
- The `.constructor` property of each instance provides an identifier for associating instances with constructors.
- The constructor's own code provides initialization for each instance.

We can do all these things with `Object.create`, but if we want to do exactly these things, and little else, `new` and a constructor function are easier, simpler, and familiar at a glance to other JavaScript programmers.

But when we want to do more, or different things, it might be better to use `Object.create` directly.

## Why Classes in JavaScript?

JavaScript programmers have been using constructors for a very long time. Long enough to notice several drawbacks with them:

1. There are too many "moving parts." Why is it necessary to define a constructor function, then manipulate its `prototype` property in a separate step?
2. Why is chaining prototypes so complicated?

**Experienced JavaScript programmers generally responded by moving in either of two directions:** :one: Some programmers noticed that working directly with prototypes was simpler than doing everything with constructors, and gravitated towards using `Object.create` directly, using the techniques we've discussed in the section on [Metaobjects](#metaobjects).

This approach is more flexible and powerful than using constructors, however it often seems *unfamiliar* to people who have been taught that objects should always be associated with a hierarchy of classes.

### abstractioneering

:two: Other experienced JavaScript programmers embraced classes, but paved over the awkwardness of constructors and prototypes by building their own class abstractions. For example:
```js
    const clazz = (...args) => {
      let superclazz, properties, constructor;

      if (args.length === 1) {
        [superclazz, properties] = [Object, args[0]];
      }
      else [superclazz, properties] = args;

      if (properties.constructor) {
        constructor = function (...args) {
          return properties.constructor.apply(this, args)
        }
      }
      else constructor = function () {};

      constructor.prototype = Object.create(superclazz.prototype);
      Object.assign(constructor.prototype, properties);
      Object.defineProperty(
        constructor.prototype,
        'constructor',
        { value: constructor }
      );

      return constructor;
    }
```
With this `clazz` function, we can write a `Queue` like this:
```js
    const Queue = clazz({
      constructor: function () {
        Object.assign(this, {
          array: [],
          head: 0,
          tail: -1
        });
      },
      pushTail: function (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead: function () {
        if (!this.isEmpty()) {
          let value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty: function () {
        return this.tail < this.head
      }
    });
```
And we can write a `Dequeue` that "subclasses" a `Queue` like this:
```js
    const Dequeue = clazz(Queue, {
      constructor: function () {
        Queue.prototype.constructor.call(this)
      },
      size: function () {
        return this.tail - this.head + 1
      },
      pullTail: function () {
        if (!this.isEmpty()) {
          let value = this.array[this.tail];
          this.array[this.tail] = void 0;
          this.tail -= 1;
          return value
        }
      },
      pushHead: function (value) {
        if (this.head === 0) {
          for (let i = this.tail; i >= this.head; --i) {
            this.array[i + this.constructor.INCREMENT] = this.array[i]
          }
          this.tail += this.constructor.INCREMENT;
          this.head += this.constructor.INCREMENT
        }
        this.array[this.head -= 1] = value
      }
    });

    Dequeue.INCREMENT = 4;
```
Chaining prototypes is handled for us, and we can set up the constructor function and the prototype's methods in one step. And there's a lot to be said for making "classes" out of prototypes. Because prototypes are "just objects," and methods are "just functions," we can re-use a lot of the techniques we've already developed for objects and functions with our prototypes and methods.

### why prototypes being objects is a win

For example, we can use `Object.assign` to mix functionality into our classes:
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

    const Manager = clazz(Person, {
      constructor: function (first, last) {
        Person.call(this, first, last);
      },
      function addReport (report) {
        this.reports().add(report);
        return this;
      },
      function removeReport (report) {
        this.reports().delete(report);
        return this;
      },
      function reports () {
        return this._reports || (this._reports = new Set());
      }
    });

    const MiddleManager = clazz(Manager, {
      constructor: function (first, last) {
        Manager.call(this, first, last);
      }
    });
    Object.assign(MiddleManager.prototype, HasManager);

    const Worker = clazz(Person, {
      constructor: function (first, last) {
        Person.call(this, first, last);
      }
    });
    Object.assign(Worker.prototype, HasManager);
```
Or even more declaratively:
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

    const Manager = clazz(Person, {
      constructor: function (first, last) {
        Person.call(this, first, last);
      },
      function addReport (report) {
        this.reports().add(report);
        return this;
      },
      function removeReport (report) {
        this.reports().delete(report);
        return this;
      },
      function reports () {
        return this._reports || (this._reports = new Set());
      }
    });

    const MiddleManager = clazz(Manager, Object.assign({
      constructor: function (first, last) {
        Manager.call(this, first, last);
      }
    }, HasManager));

    const Worker = clazz(Person, Object.assign({
      constructor: function (first, last) {
        Person.call(this, first, last);
      }
    }, HasManager));
```
Likewise, decorating methods is as easy with these "classes" as it is with any other method:
```js
    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }

    const Manager = clazz(Person, {
      constructor: function (first, last) {
        Person.call(this, first, last);
      },
      addReport: fluent(function (report) {
        this.reports().add(report);
      }),
      removeReport: fluent(function (report) {
        this.reports().delete(report);
      }),
      function reports () {
        return this._reports || (this._reports = new Set());
      }
    });

    const MiddleManager = clazz(Manager, Object.assign({
      constructor: function (first, last) {
        Manager.call(this, first, last);
      }
    }, HasManager));

    const Worker = clazz(Person, Object.assign({
      constructor: function (first, last) {
        Person.call(this, first, last);
      }
    }, HasManager));
```
### the problem with rolling our own classes

Building abstractions is a fundamental activity in programming. So it is not wrong to take basic tools like prototypes and build upwards from them.

However.
JavaScript is a simple and elegant language, and being able to write something like `clazz` in 20-ish lines of code is wonderful. It is not a hardship to read 20 lines of code to figure out how something works. Unless you have to read twenty lines of code every time you read a new program.

If everyone, or a very large number of people, are building roughly the same abstractions, but doing them in slightly different ways, each program is nice, but the ecosystem as a whole is a mess. Every time we read a new program, we have to figure out whether they are using raw constructors, rolling their own class abstraction, or using classes from various libraries.

For this reason (and perhaps others), the `class` keyword was added to the JavaScript language.

## Classes with `class`

![Vac Pot Upper Chamber](images/vacuum-upper.jpg)

JavaScript now has a simple way to write a "class." Here's a simple class written with `clazz`:
```js
const Person = clazz({
  constructor: function (first, last) {
    this.rename(first, last);
    },
  fullName: function () {
    return this.firstName + " " + this.lastName;
  },
  rename: function (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
});
```
And here it is with the `class` keyword:
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
```
And here's a `Dequeue` to show "inheritance:"
```js
    class Dequeue extends Queue {
      constructor: function () {
        Queue.prototype.constructor.call(this)
      },
      size: function () {
        return this.tail - this.head + 1
      },
      pullTail: function () {
        if (!this.isEmpty()) {
          let value = this.array[this.tail];
          this.array[this.tail] = void 0;
          this.tail -= 1;
          return value
        }
      },
      pushHead: function (value) {
        if (this.head === 0) {
          for (let i = this.tail; i >= this.head; --i) {
            this.array[i + this.constructor.INCREMENT] = this.array[i]
          }
          this.tail += this.constructor.INCREMENT;
          this.head += this.constructor.INCREMENT
        }
        this.array[this.head -= 1] = value
      }
    });

    Dequeue.INCREMENT = 4;
```
The interesting thing about `Dequeue` is that it works whether we write our `Queue` like this:
```js
    function Queue () {
      Object.assign(this, {
        array: [],
        head: 0,
        tail: -1
      });
    }

    Object.assign(Queue.prototype, {
      pushTail: function (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead: function () {
        if (!this.isEmpty()) {
          let value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty: function () {
        return this.tail < this.head
      }
    });
```
Or like this:
```js
    const Queue = clazz({
      constructor: function () {
        Object.assign(this, {
          array: [],
          head: 0,
          tail: -1
        });
      },
      pushTail: function (value) {
        return this.array[this.tail += 1] = value
      },
      pullHead: function () {
        if (!this.isEmpty()) {
          let value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      },
      isEmpty: function () {
        return this.tail < this.head
      }
    });
```
Or even like this:
```js
    class Queue {
      constructor () {
        Object.assign(this, {
          array: [],
          head: 0,
          tail: -1
        });
      }
      pushTail (value) {
        return this.array[this.tail += 1] = value
      }
      pullHead () {
        if (!this.isEmpty()) {
          let value = this.array[this.head]
          this.array[this.head] = void 0;
          this.head += 1;
          return value
        }
      }
      isEmpty () {
        return this.tail < this.head
      }
    }
```
It turns out that "classes" in JavaScript are fully compatible with constructors and prototypes. That's because behind the scenes, *they're almost indistinguishable*. In basic use, the `class` keyword is syntactic sugar for writing constructor functions with prototypes.

There is some extra magic for handling `super` (and a few other nice-to-have features like getters and setters), but by design, and to maximize compatibility with existing code bases, the `class` keyword is a declarative way to write functions and prototypes.

### classes are values

When we write:
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
```
It looks like we are creating a global class named `Person`. Some other languages sometimes have this idea that class names have a special significance and that they're always global, although you can namespace them in certain ways, and the mechanism behind class names and namespaces if different than the mechanism behind variable bindings.

JavaScript does not do this. `Person` is a name bound in the environment where we evaluate the code. So yes, at the topmost level, that code creates a global binding.

But we could also write something like this, taking advantage of [privacy with symbols](#privacy-with-symbols):
```js
const PrivatePerson = (() => {
  const firstName = Symbol('firstName'),
        lastName  = Symbol('lastName');

  return class Person {
    constructor (first, last) {
      ++population;
      this.rename(first, last);
    }
    fullName () {
      return this[firstName] + " " + this[firstName];
    }
    rename (first, last) {
      this[firstName] = first;
      this[firstName] = last;
      return this;
    }
  };
})();
```
What does this do? It creates some symbols, then creates a class (also named person) within the same environment and uses those symbols to create private properties. It then returns the newly created class, which we bind to the name `PrivatePerson`. This hides the symbols `firstName` and `lastName` from other code.

Notice also that we *returned* the class. This implies (correctly) that the `class` keyword creates a *class expression*, and an expression is a value that can be used everywhere, just like a named function expression.

Of course, we could have bound the value returned from the IIFE to any name we like, even `Person`, but we give it a different name just to show that we have a value, just like any other value, and we bind it to a name in the environment, just like any other name in the environment. In this case, even the name `Person` is encapsulated within the IIFE.

In JavaScript, "classes" and "class expressions" are values just like any other value, and that means we can do anything with them that we can do with other values, like return them from functions, pass them to functions, and bind them to different names as we see fit.

## Object Methods

An ***instance method*** is [a function](#aa) defined in the constructor's prototype. Every instance acquires this behaviour unless otherwise "overridden." Instance methods usually have some interaction with the instance, such as references to `this` or to other methods that interact with the instance. A ***constructor method*** is [a function](#aa) belonging to the constructor itself.

There is a third kind of method, one that any object (obviously including all instances) can have. An ***object method*** is [a function](#aa) defined in the object itself. Like instance methods, object methods usually have some interaction with the object, such as references to `this` or to other methods that interact with the object.

Object methods are really easy to create with Plain Old JavaScript Objects, because they're the only kind of method you can use. Recall from [This and That](#this):
```js
    const BetterQueue = () =>
      ({
        array: [], 
        head: 0, 
        tail: -1,
        pushTail: function (value) {
          return this.array[this.tail += 1] = value
        },
        pullHead: function () {
          if (this.tail >= this.head) {
            let value = this.array[this.head];
            
            this.array[this.head] = void 0;
            this.head += 1;
            return value
          }
        },
        isEmpty: function () {
          this.tail < this.head
        }
      });
```        
`pushTail`, `pullHead`, and `isEmpty` are object methods. Also, from [encapsulation](#hiding-state):
```js
    const stack = (() => {
      const obj = {
        array: [],
        index: -1,
        push: (value) => obj.array[obj.index += 1] = value,
        pop: () => {
          const value = obj.array[obj.index];
          
          obj.array[obj.index] = undefined;
          if (obj.index >= 0) { 
            obj.index -= 1 
          }
          return value
        },
        isEmpty: () => obj.index < 0
      };
      
      return obj;
    })();
```
Although they don't refer to the object, `push`, `pop`, and `isEmpty` semantically interact with the opaque data structure represented by the object, so they are object methods too.

### object methods within instances

Instances of constructors can have object methods as well. Typically, object methods are added in the constructor. Here's a gratuitous example, a widget model that has a read-only `id`:
```js
    const WidgetModel = function (id, attrs) {
      Object.assign(this, attrs || {});
      this.id = function () { return id }
    }
    
    Object.assign(WidgetModel.prototype, {
      set: function (attr, value) {
        this[attr] = value;
        return this;
      },
      get: function (attr) {
        return this[attr]
      }
    });
```
`set` and `get` are instance methods, but `id` is an object method: Each object has its own `id` closure, where `id` is bound to the id of the widget by the argument `id` in the constructor. The advantage of this approach is that instances can have different object methods, or object methods with their own closures as in this case. The disadvantage is that every object has its own methods, which uses up much more memory than instance methods, which are shared amongst all instances.

> Object methods are defined within the object. So if you have several different "instances" of the same object, there will be an object method for each object. Object methods can be associated with any object, not just those created with the `new` keyword. Instance methods apply  to instances, objects created with the `new` keyword. Instance methods are defined in a  prototype and are shared by all instances.

## Why Not Classes?

Classes are popular, and if classes map neatly to the way we wish to model something, we should use them. That being said, there are some caveats to understand.

### the `class` keyword is a minimal notation

By design, the `class` keyword provides the very minimum set of features needed to implement "classes." Everything else must be done in some other way. For example, if you write constructors or prototypes directly, you can use method decorators (as we saw [earlier](#prototype-is-a-win)):
```js
    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }
      
    const Manager = clazz(Person, {
      constructor: function (first, last) {
        Person.call(this, first, last);
      },
      addReport: fluent(function (report) {
        this.reports || (this.reports = new Set());
        this.reports.add(report);
      }),
      removeReport: fluent(function (report) {
        this.reports || (this.reports = new Set());
        this.reports.delete(report);
      }),
      reports: function () {
        return this.reports || (this.reports = new Set());
      }
    });
```    
But at this time, you cannot use method decorators when you use the `class` syntax. There are plans to introduce a new, purpose-built decorator syntax for this purpose, which highlights one of the issues with the `class` syntax: By writing what amounts to a new language on top of JavaScript, it must inevitably reinvent all of the things that are already possible in JavaScript.

### classes encourage the construction of class hierarchies

The easy thing to do with classes is to create [class hierarchies][ch]. These are implemented by chaining prototypes. And there is a problem with chained prototypes: They couple classes to each other.

![A class hierarchy](images/tree.jpg)

When one class extends another, its methods can access any of the properties and methods defined anywhere on the prototype chain. Given hierarchies designed as trees, a change to a class can break the behaviour of any of the classes below it or above it on the tree.

When two or more metaobjects all have access to the same base object via [open recursion][or], they become tightly coupled because they can interact via setting and reading all the base object's properties. It is impossible to restrict their interaction to a well-defined set of methods.

This coupling exists for all metaobject patterns that include open recursion, such as mixins, delegation, and delegation through prototypes. In particular, when chains of naive prototypes form class hierarchies, this coupling leads to the [fragile base class problem][fbc].

> The **fragile base class problem** is a fundamental architectural problem of object-oriented programming systems where base classes (superclasses) are considered "fragile" because seemingly safe modifications to a base class, when inherited by the derived classes, may cause the derived classes to malfunction. The programmer cannot determine whether a base class change is safe simply by examining in isolation the methods of the base class.--[Wikipedia](https://en.wikipedia.org/wiki/Fragile_base_class)

In JavaScript, prototype chains are vulnerable because changes to one prototype's behaviour may break another prototype's behaviour in the same chain.

[fbc]: https://en.wikipedia.org/wiki/Fragile_base_class

[or]: https://en.wikipedia.org/wiki/Open_recursion#Open_recursion

[ch]: https://en.wikipedia.org/wiki/Class_hierarchy

In the next section we will look at a technique for [reducing coupling between classes](main_6_classes.md#reducing-coupling). And we will look at avoiding deep hierarchies with mixins.

## Summary
 ### Instances and Classes
* The `new` keyword turns any function into a *constructor* for creating *instances*.
* All functions have a `prototype` element.
* Instances behave as if the elements of their constructor's prototype are their elements.
* The `class` keyword acts as *syntactic sugar* for writing constructor functions.
* Classes created with the class keyword are actually constructor functions with optionally chained prototypes.
* Classes should be used in moderation, the syntax deliberately limits the flexibility and class hierarchies can lead to overly coupled code.
