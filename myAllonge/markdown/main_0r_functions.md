# Recipes with Basic Functions

![Before combining ingredients, begin with implements so clean, they gleam.](images/mirage.jpg)

Having looked at basic pure functions and closures, we're going to see some practical recipes that focus on the premise of functions that return functions.

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.
## Partial Application {#simple-partial}

In [Building Blocks](#buildingblocks), we discussed partial application, but we didn't write a generalized recipe for it. This is such a common tool that many libraries provide some form of partial application. You'll find examples in [Lemonad](https://github.com/fogus/lemonad) from Michael Fogus, [Functional JavaScript](http://osteele.com/sources/javascript/functional/) from Oliver Steele and the terse but handy [node-ap](https://github.com/substack/node-ap) from James Halliday.

These two recipes are for quickly and simply applying a single argument, either the leftmost or rightmost.[^inspired] If you want to bind more than one argument, or you want to leave a "hole" in the argument list, you will need to either use a [generalized partial recipe](#partial), or you will need to repeatedly apply arguments. They are [context](#context)-agnostic.

    const callFirst = (fn, larg) =>
      function (...rest) {
        return fn.call(this, larg, ...rest);
      }
    
    const callLast = (fn, rarg) =>
      function (...rest) {
        return fn.call(this, ...rest, rarg);
      }
    
    const greet = (me, you) =>
      `Hello, ${you}, my name is ${me}`;
      
    const heliosSaysHello = callFirst(greet, 'Helios');
    
    heliosSaysHello('Eartha')
      //=> 'Hello, Eartha, my name is Helios'
      
    const sayHelloToCeline = callLast(greet, 'Celine');
    
    sayHelloToCeline('Eartha')
      //=> 'Hello, Celine, my name is Eartha'
      
As noted above, our partial recipe allows us to create functions that are partial applications of functions that are context aware. We'd need a different recipe if we wish to create partial applications of object methods.

We take it a step further, and can use gathering and spreading to allow for partial application with more than one argument:

    const callLeft = (fn, ...args) =>
        (...remainingArgs) =>
          fn(...args, ...remainingArgs);

    const callRight = (fn, ...args) =>
        (...remainingArgs) =>
          fn(...remainingArgs, ...args);

[^inspired]: `callFirst` and `callLast` were inspired by Michael Fogus' [Lemonad](https://github.com/fogus/lemonad). Thanks!
## Unary

"Unary" is a function decorator that modifies the number of arguments a function takes: Unary takes any function and turns it into a function taking exactly one argument.

The most common use case is to fix a problem. JavaScript has a `.map` method for arrays, and many libraries offer a `map` function with the same semantics. Here it is in action:

    ['1', '2', '3'].map(parseFloat)
      //=> [1, 2, 3]
      
In that example, it looks exactly like the mapping function you'll find in most languages: You pass it a function, and it calls the function with one argument, the element of the array. However, that's not the whole story. JavaScript's `map` actually calls each function with *three* arguments: The element, the index of the element in the array, and the array itself.

Let's try it:

    [1, 2, 3].map(function (element, index, arr) {
      console.log({element: element, index: index, arr: arr})
    })
      //=> { element: 1, index: 0, arr: [ 1, 2, 3 ] }
      //   { element: 2, index: 1, arr: [ 1, 2, 3 ] }
      //   { element: 3, index: 2, arr: [ 1, 2, 3 ] }
      
If you pass in a function taking only one argument, it simply ignores the additional arguments. But some functions have optional second or even third arguments. For example:

    ['1', '2', '3'].map(parseInt)
      //=> [1, NaN, NaN]

This doesn't work because `parseInt` is defined as `parseInt(string[, radix])`. It takes an optional radix argument. And when you call `parseInt` with `map`, the index is interpreted as a radix. Not good! What we want is to convert `parseInt` into a function taking only one argument.

We could write `['1', '2', '3'].map((s) => parseInt(s))`, or we could come up with a decorator to do the job for us:

    const unary = (fn) =>
      fn.length === 1
        ? fn
        : function (something) {
            return fn.call(this, something)
          }

And now we can write:

    ['1', '2', '3'].map(unary(parseInt))
      //=> [1, 2, 3]
      
Presto!
## Tap {#tap}

One of the most basic combinators is the "K Combinator," nicknamed the "Kestrel:"

    const K = (x) => (y) => x;

It has some surprising applications. One is when you want to do something with a value for side-effects, but keep the value around. Behold:

    const tap = (value) =>
      (fn) => (
        typeof(fn) === 'function' && fn(value),
        value
      )

`tap` is a traditional name borrowed from various Unix shell commands. It takes a value and returns a function that always returns the value, but if you pass it a function, it executes the function for side-effects. Let's see it in action as a poor-man's debugger:

    tap('espresso')((it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    

It's easy to turn off:

    tap('espresso')();
      //=> 'espresso'

Libraries like [Underscore] use a version of `tap` that is "uncurried:"

    _.tap('espresso', (it) =>
      console.log(`Our drink is '${it}'`) 
    );
      //=> Our drink is 'espresso'
           'espresso'
    
Let's enhance our recipe so that it works both ways:

    const tap = (value, fn) => {
      const curried = (fn) => (
          typeof(fn) === 'function' && fn(value),
          value
        );
      
      return fn === undefined
             ? curried
             : curried(fn);
    }

Now we can write:

    tap('espresso')((it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    
Or:

    tap('espresso', (it) => {
      console.log(`Our drink is '${it}'`) 
    });
    //=> Our drink is 'espresso'
         'espresso'
    
And if we wish it to do nothing at all, We can write either `tap('espresso')()` or `tap('espresso', null)`

p.s. `tap` can do more than just act as a debugging aid. It's also useful for working with [object and instance methods](#tap-methods).

[Underscore]: http://underscorejs.org
## Maybe {#maybe}

A common problem in programming is checking for `null` or `undefined` (hereafter called "nothing," while all other values including `0`, `[]` and `false` will be called "something"). Languages like JavaScript do not strongly enforce the notion that a particular variable or particular property be something, so programs are often written to account for values that may be nothing.

This recipe concerns a pattern that is very common: A function `fn` takes a value as a parameter, and its behaviour by design is to do nothing if the parameter is nothing:

    const isSomething = (value) =>
      value !== null && value !== void 0;

    const checksForSomething = (value) => {
      if (isSomething(value)) {
        // function's true logic
      }
    }

Alternately, the function may be intended to work with any value, but the code calling the function wishes to emulate the behaviour of doing nothing by design when given nothing:

    var something =
      isSomething(value)
        ? doesntCheckForSomething(value)
        : value;

Naturally, there's a function decorator recipe for that, borrowed from Haskell's [maybe monad][maybe], Ruby's [andand], and CoffeeScript's existential method invocation:

    const maybe = (fn) =>
      function (...args) {
        if (args.length === 0) {
          return
        }
        else {
          for (let arg of args) {
            if (arg == null) return;
          }
          return fn.apply(this, args)
        }
      }

`maybe` reduces the logic of checking for nothing to a function call:

    maybe((a, b, c) => a + b + c)(1, 2, 3)
      //=> 6

    maybe((a, b, c) => a + b + c)(1, null, 3)
      //=> undefined

As a bonus, `maybe` plays very nicely with instance methods, we'll discuss those [later](#classes):

    function Model () {};

    Model.prototype.setSomething = maybe(function (value) {
      this.something = value;
    });

If some code ever tries to call `model.setSomething` with nothing, the operation will be skipped.

[andand]: https://github.com/raganwald/andand
[maybe]: https://en.wikipedia.org/wiki/Monad_(functional_programming)#The_Maybe_monad
## Once

`once` is an extremely helpful combinator. It ensures that a function can only be called, well, *once*. Here's the recipe:

    const once = (fn) => {
      let done = false;

      return function () {
        return done ? void 0 : ((done = true), fn.apply(this, arguments))
      }
    }

Very simple! You pass it a function, and you get a function back. That function will call your function once, and thereafter will return `undefined` whenever it is called. Let's try it:

    const askedOnBlindDate = once(
      () => "sure, why not?"
    );

    askedOnBlindDate()
      //=> 'sure, why not?'

    askedOnBlindDate()
      //=> undefined

    askedOnBlindDate()
      //=> undefined

It seems some people will only try blind dating once.

(Note: There are some subtleties with decorators like `once` that involve the intersection of state with methods. We'll look at that again in [stateful method decorators](#stateful-method-decorators).)
## Left-Variadic Functions

A *variadic function* is a function that is designed to accept a variable number of arguments.[^eng] In JavaScript, you can make a variadic function by gathering parameters. For example:

[^eng]: English is about as inconsistent as JavaScript: Functions with a fixed number of arguments can be unary, binary, ternary, and so forth. But can they be "variary?" No! They have to be "variadic."

{:lang="js"}
~~~~~~~~
const abccc = (a, b, ...c) => {
  console.log(a);
  console.log(b);
  console.log(c);
};

abccc(1, 2, 3, 4, 5)
  1
  2
  [3,4,5]
~~~~~~~~

This can be useful when writing certain kinds of destructuring algorithms. For example, we might want to have a function that builds some kind of team record. It accepts a coach, a captain, and an arbitrary number of players. Easy in ECMAScript 2015:

{:lang="js"}
~~~~~~~~
function team(coach, captain, ...players) {
  console.log(`${captain} (captain)`);
  for (let player of players) {
    console.log(player);
  }
  console.log(`squad coached by ${coach}`);
}

team('Luis Enrique', 'Xavi Hernández', 'Marc-André ter Stegen',
     'Martín Montoya', 'Gerard Piqué')
  //=>
    Xavi Hernández (captain)
    Marc-André ter Stegen
    Martín Montoya
    Gerard Piqué
    squad coached by Luis Enrique
~~~~~~~~

But we can't go the other way around:

{:lang="js"}
~~~~~~~~
function team2(...players, captain, coach) {
  console.log(`${captain} (captain)`);
  for (let player of players) {
    console.log(player);
  }
  console.log(`squad coached by ${coach}`);
}
//=> Unexpected token
~~~~~~~~

ECMAScript 2015 only permits gathering parameters from the *end* of the parameter list. Not the beginning. What to do?

### a history lesson

In "Ye Olde Days,"[^ye] JavaScript could not gather parameters, and we had to either do backflips with `arguments` and `.slice`, or we wrote ourselves a `variadic` decorator that could gather arguments into the last declared parameter. Here it is in all of its ECMAScript-5 glory:

[^ye]: Another history lesson. "Ye" in "Ye Olde," was not actually spelled with a "Y" in days of old, it was spelled with a [thorn](https://en.wikipedia.org/wiki/Thorn_(letter)), and is pronounced "the." Another word, "Ye" in "Ye of little programming faith," is pronounced "ye," but it's a different word altogether.

{:lang="js"}
~~~~~~~~
var __slice = Array.prototype.slice;

function rightVariadic (fn) {
  if (fn.length < 1) return fn;

  return function () {
    var ordinaryArgs = (1 <= arguments.length ?
          __slice.call(arguments, 0, fn.length - 1) : []),
        restOfTheArgsList = __slice.call(arguments, fn.length - 1),
        args = (fn.length <= arguments.length ?
          ordinaryArgs.concat([restOfTheArgsList]) : []);

    return fn.apply(this, args);
  }
};

var firstAndButFirst = rightVariadic(function test (first, butFirst) {
  return [first, butFirst]
});

firstAndButFirst('why', 'hello', 'there', 'little', 'droid')
  //=> ["why",["hello","there","little","droid"]]
~~~~~~~~

We don't need `rightVariadic` any more, because instead of:

{:lang="js"}
~~~~~~~~
var firstAndButFirst = rightVariadic(
  function test (first, butFirst) {
    return [first, butFirst]
  });
~~~~~~~~

We now simply write:

{:lang="js"}
~~~~~~~~
const firstAndButFirst = (first, ...butFirst) =>
  [first, butFirst];
~~~~~~~~

This is a *right-variadic function*, meaning that it has one or more fixed arguments, and the rest are gathered into the rightmost argument.

### overcoming limitations

It's nice to have progress. But as noted above, we can't write:

{:lang="js"}
~~~~~~~~
const butLastAndLast = (...butLast, last) =>
  [butLast, last];
~~~~~~~~

That's a *left-variadic function*. All left-variadic functions have one or more fixed arguments, and the rest are gathered into the leftmost argument. JavaScript doesn't do this. But if we wanted to write left-variadic functions, could we make ourselves a `leftVariadic` decorator to turn a function with one or more arguments into a left-variadic function?

We sure can, by using the techniques from `rightVariadic`. Mind you, we can take advantage of modern JavaScript to simplify the code:

{:lang="js"}
~~~~~~~~
const leftVariadic = (fn) => {
  if (fn.length < 1) {
    return fn;
  }
  else {
    return function (...args) {
      const gathered = args.slice(0, args.length - fn.length + 1),
            spread   = args.slice(args.length - fn.length + 1);

      return fn.apply(
        this, [gathered].concat(spread)
      );
    }
  }
};

const butLastAndLast = leftVariadic((butLast, last) => [butLast, last]);

butLastAndLast('why', 'hello', 'there', 'little', 'droid')
  //=> [["why","hello","there","little"],"droid"]
~~~~~~~~

Our `leftVariadic` function is a decorator that turns any function into a function that gathers parameters *from the left*, instead of from the right.

### left-variadic destructuring

Gathering arguments for functions is one of the ways JavaScript can *destructure* arrays. Another way is when assigning variables, like this:

{:lang="js"}
~~~~~~~~
const [first, ...butFirst] = ['why', 'hello', 'there', 'little', 'droid'];

first
  //=> 'why'
butFirst
  //=> ["hello","there","little","droid"]
~~~~~~~~

As with parameters, we can't gather values from the left when destructuring an array:

{:lang="js"}
~~~~~~~~
const [...butLast, last] = ['why', 'hello', 'there', 'little', 'droid'];
  //=> Unexpected token
~~~~~~~~

We could use `leftVariadic` the hard way:

{:lang="js"}
~~~~~~~~
const [butLast, last] = leftVariadic((butLast, last) => [butLast, last])(...['why', 'hello', 'there', 'little', 'droid']);

butLast
  //=> ['why', 'hello', 'there', 'little']

last
  //=> 'droid'
~~~~~~~~

But we can write our own left-gathering function utility using the same principles without all the tedium:

{:lang="js"}
~~~~~~~~
const leftGather = (outputArrayLength) => {
  return function (inputArray) {
    return [inputArray.slice(0, inputArray.length - outputArrayLength + 1)].concat(
      inputArray.slice(inputArray.length - outputArrayLength + 1)
    )
  }
};

const [butLast, last] = leftGather(2)(['why', 'hello', 'there', 'little', 'droid']);

butLast
  //=> ['why', 'hello', 'there', 'little']

last
  //=> 'droid'
~~~~~~~~

With `leftGather`, we have to supply the length of the array we wish to use as the result, and it gathers excess arguments into it from the left, just like `leftVariadic` gathers excess parameters for a function.
## Compose and Pipeline

Here is the B Combinator, or `compose` that we saw in [Combinators and Decorators](#combinators):

    const compose = (a, b) =>
      (c) => a(b(c))

As we saw before, given:

    const addOne = (number) => number + 1;

    const doubleOf = (number) => number * 2;

Instead of:

    const doubleOfAddOne = (number) => doubleOf(addOne(number));

We could write:

    const doubleOfAddOne = compose(doubleOf, addOne);

### variadic compose and recursion

If we wanted to implement a `compose3`, we could write:

    const compose3 = (a, b, c) => (d) =>
      a(b(c(d)))

Or observe that it is really:

    const compose3 = (a, b, c) => compose(a(compose(b, c)))

Once we get to `compose4`, we ask ourselves if there is a better way. For example, if we had a *variadic* compose, we could write `compose(a, b)`, `compose(a, b, c)`, or `compose(a, b, c, d)`.

We can implement a variadic `compose` recursively. The easiest way to reason about writing a recursive `compose` is to start with the smallest or *degenerate* case. If `compose` only took one argument, it would look like this:

    const compose = (a) => a

The next thing is to have a way of breaking a piece off the problem. We can do this with a variadic function:

    const compose = (a, ...rest) =>
      "to be determined"

We can test whether we have the degenerate case:

    const compose = (a, ...rest) =>
      rest.length === 0
        ? a
        : "to be determined"

If it is not the degenerate case, we need to combine what we have with the solution for the rest. In other words, we need to combine `fn` with `compose(...rest)`. How do we do that? Well, consider `compose(a, b)`. We know that `compose(b)` is the degenerate case, it's just `b`. And we know that `compose(a, b)` is `(c) => a(b(c))`.

So let's substitute `compose(b)` for `b`:

    compose(a, compose(b)) === (c) => a(compose(b)(c))

Now substitute `...rest` for `b`:

    compose(a, ...rest) === (c) => a(compose(...rest)(c))

This is our solution:

    const compose = (a, ...rest) =>
      rest.length === 0
        ? a
        : (c) => a(compose(...rest)(c))

There are others, of course. `compose` can be implemented with iteration or with `.reduce`, like this:

    const compose = (...fns) =>
      (value) =>
        fns.reverse().reduce((acc, fn) => fn(acc), value);

But the principle behaviour is the same: To compose a series of functions together, creating a new one. And the value is the same: We can write smaller, single purpose functions and put them together in different ways.

### the semantics of compose

With `compose`, we're usually making a new function. Although it works perfectly well, we don't need to write things like `compose(double, addOne)(3)` inline to get the result `8`. It's easier and clearer to write `double(addOne(3))`.

On the other hand, when working with something like method decorators, it can help to write:

    const setter = compose(fluent, maybe);

    // ...

    SomeClass.prototype.setUser = setter(function (user) {
      this.user = user;
    });

    SomeClass.prototype.setPrivileges = setter(function (privileges) {
      this.privileges = privileges;
    });

This makes it clear that `setter` adds the behaviour of both `fluent` and `maybe` to each method it decorates, and it's sometimes easier to read `const setter = compose(fluent, maybe);` than:

    const setter = (fn) => fluent(maybe(fn));

The take-away is that `compose` is helpful when we are defining a new function that combines the effects of existing functions.

### pipeline {#pipeline}

`compose` is extremely handy, but one thing it doesn't communicate well is the order on operations. `compose` is written that way because it matches the way explicitly composing functions works in JavaScript and most other languages: When you write a(b(...)), `a` happens after `b`.

Sometimes it makes more sense to compose functions in data flow order, as in "The value flows through a and then through b." For this, we can use the `pipeline` function:

    const pipeline = (...fns) =>
      (value) =>
        fns.reduce((acc, fn) => fn(acc), value);

    const setter = pipeline(addOne, double);

Comparing `pipeline` to `compose`, pipeline says "add one to the number and then double it." Compose says, "double the result of adding one to the number." Both do the same job, but communicate their intention in opposite ways.

![Saltspring Island Roasting Facility](images/saltspring/rollers.jpg)
