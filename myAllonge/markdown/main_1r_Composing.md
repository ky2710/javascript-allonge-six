# Recipes with Data

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## mapWith

In JavaScript, arrays have a `.map` method. Map takes a function as an argument, and applies it to each of the elements of the array, then returns the results in another array. For example:

```js
[1, 2, 3, 4, 5].map(x => x * x)
  //=> [1, 4, 9, 16, 25]
```
We could write a function that behaves like the `.map` method if we wanted:
```js
const map = (list, fn) =>
  list.map(fn);
```
This recipe isn't for `map`: It's for `mapWith`, a function that wraps around `map` and turns any other function into a mapper. `mapWith` is very simple:`1`

>`1` If we were always `mapWith`-ing arrays, we could write `list.map(fn)`. However, there are some objects that have a `.length` property and `[]` accessors that can be mapWithted but do not have a `.map` method. `mapWith` works with those objects. This points to a larger issue around the question of whether containers really ought to implement methods like `.map`. In a language like JavaScript, we are free to define objects that know about their own implementations, such as exactly how `[]` and `.length` works and then to define standalone functions that do the rest.
```js
const mapWith = (fn) => (list) => list.map(fn);
```
`mapWith` differs from `map` in two ways. It reverses the arguments, taking the function first and the list second. It also "curries" the function: Instead of taking two arguments, it takes one argument and returns a function that takes another argument.

That means that you can pass a function to `mapWith` and get back a function that applies that mapping to any array. For example, we might need a function to return the squares of an array. Instead of writing a a wrapper around `.map`:
```js
    const squaresOf = (list) =>
      list.map(x => x * x);

    squaresOf([1, 2, 3, 4, 5])
      //=> [1, 4, 9, 16, 25]
```
We can call `mapWith` in one step:
```js
    const squaresOf = mapWith(n => n * n);

    squaresOf([1, 2, 3, 4, 5])
      //=> [1, 4, 9, 16, 25]
```
If we didn't use `mapWith`, we'd could have also used `callRight` with `map` to accomplish the same result:
```js
    const squaresOf = callRight(map, (n => n * n);

    squaresOf([1, 2, 3, 4, 5])
      //=> [1, 4, 9, 16, 25]
```
Both patterns take us to the same destination: Composing functions out of common pieces, rather than building them entirely from scratch. `mapWith` is a very convenient abstraction for a very common pattern.

*`mapWith` was suggested by [ludicast](http://github.com/ludicast)*

## Flip

We wrote [mapWith](#mapWith) like this:
```js
const mapWith = (fn) => (list) => list.map(fn);
```
Let's consider the case whether we have a `map` function of our own, perhaps from the [allong.es](https://github.com/raganwald/allong.es) library, perhaps from [Underscore](http://underscorejs.org). We could write our function something like this:
```js
    const mapWith = (fn) => (list) => map(list, fn);
```
Looking at this, we see we're conflating two separate transformations. First, we're reversing the order of arguments. You can see that if we simplify it:
```js
    const mapWith = (fn, list) => map(list, fn);
```
Second, we're "currying" the function so that instead of defining a function that takes two arguments, it returns a function that takes the first argument and returns a function that takes the second argument and applies them both, like this:
```js
    const mapper = (list) => (fn) => map(list, fn);
```
Let's return to the implementation of `mapWith` that relies on a `map` function rather than a method:
```js
    const mapWith = (fn) => (list) => map(list, fn);
```    
We're going to extract these two operations by refactoring our function to paramaterize `map`. The first step is to give our parameters generic names:
```js
    const mapWith = (first) => (second) => map(second, first);
```
Then we wrap the entire thing in a function and extract `map`
```js
    const wrapper = (fn) =>
      (first) => (second) => fn(second, first);
```
What we have now is a function that takes a function and "flips" the order of arguments around, then curries it. So let's call it `flipAndCurry`:
```js
    const flipAndCurry = (fn) =>
      (first) => (second) => fn(second, first);
```      
Sometimes you want to flip, but not curry:
```js
    const flip = (fn) =>
      (first, second) => fn(second, first);
```
This is gold. Consider how we define [mapWith](#mapWith) now:
```js
    var mapWith = flipAndCurry(map);
```
Much nicer!

### self-currying flip

Sometimes we'll want to flip a function, but retain the flexibility to call it in its curried form (pass one parameter) or non-curried form (pass both). We *could* make that into `flip`:
```js
    const flip = (fn) =>
      function (first, second) {
        if (arguments.length === 2) {
          return fn(second, first);
        }
        else {
          return function (second) {
            return fn(second, first);
          };
        };
      };
```
Now if we write `mapWith = flip(map)`, we can call `mapWith(fn, list)` or `mapWith(fn)(list)`, our choice.

### flipping methods

When we learn about context and methods, we'll see that `flip` throws the current context away, so it can't be used to flip methods. A small alteration gets the job done:
```js
    const flipAndCurry = (fn) =>
      (first) =>
        function (second) {
          return fn.call(this, second, first);
        }

    const flip = (fn) =>
      function (first, second) {
        return fn.call(this, second, first);
      }

    const flip = (fn) =>
      function (first, second) {
        if (arguments.length === 2) {
          return fn.call(this, second, first);
        }
        else {
          return function (second) {
            return fn.call(this, second, first);
          };
        };
      };
```
## Object.assign

It's very common to want to "extend" an object by assigning properties to it:
```js
    const inventory = {
      apples: 12,
      oranges: 12
    };
    
    inventory.bananas = 54;
    inventory.pears = 24;
```
It's also common to want to assign the properties of one object to another:

[shallow copy]: https://en.wikipedia.org/wiki/Object_copy#Shallow_copy
```js
      for (let fruit in shipment) {
        inventory[fruit] = shipment[fruit]
      }
```
Both needs can be met with `Object.assign`, a standard function. You can copy an object by extending an empty object:
```js
    Object.assign({}, {
      apples: 12,
      oranges: 12
    })
      //=> { apples: 12, oranges: 12 }
```
You can extend one object with another:
```js
    const inventory = {
      apples: 12,
      oranges: 12
    };
    
    const shipment = {
      bananas: 54,
      pears: 24
    }
    
    Object.assign(inventory, shipment)
      //=> { apples: 12,
      //     oranges: 12,
      //     bananas: 54,
      //     pears: 24 }
```      
And when we discuss prototypes, we will use `Object.assign` to turn this:
```js
    const Queue = function () {
      this.array = [];
      this.head = 0;
      this.tail = -1
    };
      
    Queue.prototype.pushTail = function (value) {
      // ...
    };
    Queue.prototype.pullHead = function () {
      // ...
    };
    Queue.prototype.isEmpty = function () {
      // ...
    }
```
Into this:
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
        // ...
      },
      pullHead () {
        // ...
      },
      isEmpty () {
        // ...
      }      
    });
```    
Assigning properties from one object to another (also called "cloning" or "shallow copying") is a basic building block that we will later use to implement more advanced paradigms like mixins.

## Why? `1`

>`1` https://en.wikipedia.org/wiki/Fixed-point_combinator#Example_in_JavaScript "Call-by-value fixed-point combinator in JavaScript"

This is the [canonical Y Combinator][y]:
```js
    const Y = (f) =>
      ( x => f(v => x(x)(v)) )(
        x => f(v => x(x)(v))
      );
```
You use it like this:
```js
    const factorial = Y(function (fac) {
      return function (n) {
        return (n == 0 ? 1 : n * fac(n - 1));
      }
    });
 
    factorial(5)
      //=> 120
```
Why? It enables you to make recursive functions without needing to bind a function to a name in an environment. This has little practical utility in JavaScript, but in combinatory logic it's essential: With fixed-point combinators it's possible to compute everything computable without binding names.

So again, why include the recipe? Well, besides all of the practical applications that combinators provide, there is this little thing called *The joy of working things out.*

There are many explanations of the Y Combinator's mechanism on the internet, but resist the temptation to read any of them: Work it out for yourself. Use it as an excuse to get familiar with your environment's debugging facility.

One tip is to use JavaScript to name things. For example, you could start by writing:
```js
    const Y = (f) => {
      const something = x => f(v => x(x)(v));
      
      return something(something);
    };
```
What is this `something` and how does it work? Another friendly tip: Change some of the fat arrow functions inside of it into named function expressions to help you decipher stack traces.

Work things out for yourself!
