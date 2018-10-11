# Recipes with Objects, Mutations, and State

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.

## Memoize

Consider that age-old interview quiz, writing a recursive fibonacci function (there are other ways to derive a fibonacci number, of course). Here's an implementation that doesn't use a [named function expression](#named-function-expressions). The reason for that omission will be explained later:
```js
      const fibonacci = (n) =>
        n < 2
          ? n
          : fibonacci(n-2) + fibonacci(n-1);
          
      [0,1,2,3,4,5,6,7,8].map(fibonacci)
        //=> [0,1,1,2,3,5,8,13,21]
```
We'll time it:
```js
    s = (new Date()).getTime()
    fibonacci(45)
    ( (new Date()).getTime() - s ) / 1000
      //=> 15.194
```   
Why is it so slow? Well, it has a nasty habit of recalculating the same results over and over and over again. We could rearrange the computation to avoid this, but let's be lazy and trade space for time. What we want to do is use a lookup table. Whenever we want a result, we look it up. If we don't have it, we calculate it and write the result in the table to use in the future. If we do have it, we return the result without recalculating it.

Here's our recipe:
```js
    const memoized = (fn) => {
      const lookupTable = {};
        
      return function (...args) {
        const key = JSON.stringify(this, args);
      
        return lookupTable[key] || (lookupTable[key] = fn.apply(this, args));
      }
    }
``` 
We can apply `memoized` to a function and we will get back a new function that "memoizes" its results so that it never has to recalculate the same value twice. It only works for functions that are "idempotent," meaning functions that always return the same result given the same argument(s). Like `fibonacci`:

Let's try it:
```js
    const fastFibonacci = memoized(
      (n) =>
        n < 2
          ? n
          : fastFibonacci(n-2) + fastFibonacci(n-1)
    );

    fastFibonacci(45)
      //=> 1134903170
``` 
We get the result back instantly. It works! You can use memoize with all sorts of "idempotent" pure functions. by default, it works with any function that takes arguments which can be transformed into JSON using JavaScript's standard library function for this purpose.

If you have another strategy for turning the arguments into a string key, we'll need to make a version that allows you to supply an optional `keymaker` function:
```js
    const memoized = (fn, keymaker = JSON.stringify) => {
      const lookupTable = {};
        
      return function (...args) {
        const key = keymaker.apply(this, args);
      
        return lookupTable[key] || (lookupTable[key] = fn.apply(this, args));
      }
    }
```       
### memoizing recursive functions

We deliberately picked a recursive function to memoize, because it demonstrates a pitfall when combining decorators with named functional expressions. Consider this implementation that uses a named functional expression:
```js
    var fibonacci = function fibonacci (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    }
```     
If we try to memoize it, we don't get the expected speedup:
```js
    var fibonacci = memoized( function fibonacci (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    });
``` 
That's because the function bound to the name `fibonacci` in the outer environment has been memoized, but the named functional expression binds the name `fibonacci` inside the unmemoized function, so none of the recursive calls to fibonacci are *ever* memoized. Therefore we must write:
```js
    var fibonacci = memoized( function (n) {
      if (n < 2) {
        return n
      }
      else {
        return fibonacci(n-2) + fibonacci(n-1)
      }
    });
``` 
If we need to prevent a rebinding from breaking the function, we'll need to use the [module](#modules) pattern.

## getWith

`getWith` is a very simple function. It takes the name of an attribute and returns a function that extracts the value of that attribute from an object:
```js
    const getWith = (attr) => (object) => object[attr]
``` 
You can use it like this:
```js
    const inventory = {
      apples: 0,
      oranges: 144,
      eggs: 36
    };

    getWith('oranges')(inventory)
      //=> 144
``` 
This isn't much of a recipe yet. But let's combine it with [mapWith](#mapping):
```js
    const inventories = [
      { apples: 0, oranges: 144, eggs: 36 },
      { apples: 240, oranges: 54, eggs: 12 },
      { apples: 24, oranges: 12, eggs: 42 }
    ];

    mapWith(getWith('oranges'))(inventories)
      //=> [ 144, 54, 12 ]
``` 
That's nicer than writing things out "longhand:"
```js
    mapWith((inventory) => inventory.oranges)(inventories)
      //=> [ 144, 54, 12 ]
``` 
`getWith` plays nicely with [maybe](#maybe) as well. Consider a sparse array. You can use:
```js
    mapWith(maybe(getWith('oranges')))
``` 
To get the orange count from all the non-null inventories in a list.

### what's in a name?

Why is this called `getWith`? Consider this function that is common in languages that have functions and dictionaries but not methods:
```js
    const get = (object, attr) => object[attr];
``` 
You might ask, "Why use a function instead of just using `[]`?" The answer is, we can manipulate functions in ways that we can't manipulate syntax. For example, do you remember from [flip](#flip) that we can define `mapWith` from `map`?
```js
    var mapWith = flip(map);
``` 
We can do the same thing with `getWith`, and that's why it's named in this fashion:
```js
    var getWith = flip(get)
```     
## pluckWith

This pattern of combining [mapWith](#mapping) and [getWith](#getWith) is very frequent in JavaScript code. So much so, that we can take it up another level:
```js
    const pluckWith = (attr) => mapWith(getWith(attr));
```     
Or even better:
```js
    const pluckWith = compose(mapWith, getWith);
```     
And now we can write:
```js
    const inventories = [
      { apples: 0, oranges: 144, eggs: 36 },
      { apples: 240, oranges: 54, eggs: 12 },
      { apples: 24, oranges: 12, eggs: 42 }
    ];
    
    pluckWith('eggs')(inventories)
      //=> [ 36, 12, 42 ]
```       
Libraries like [Underscore] provide `pluck`, the flipped version of `pluckWith`:
```js
    _.pluck(inventories, 'eggs')
      //=> [ 36, 12, 42 ]
``` 
Our recipe is terser when you want to name a function:
```js
    const eggsByStore = pluckWith('eggs');
```     
vs.
```js
    const eggsByStore = (inventories) =>
      _.pluck(inventories, 'eggs');
```     
And of course, if we have `pluck` we can use [flip](#flip) to derive `pluckWith`:
```js
    const pluckWith = flip(_.pluck);
``` 
[Underscore]: http://underscorejs.org

## Deep Mapping

[mapWith](#mapWith) is an excellent tool, but from time to time you will find yourself working with arrays that represent trees rather than lists. For example, here is a partial list of sales extracted from a report of some kind. It's grouped in some mysterious way, and we need to operate on each item in the report.
```js
    const report = 
      [ [ { price: 1.99, id: 1 },
        { price: 4.99, id: 2 },
        { price: 7.99, id: 3 },
        { price: 1.99, id: 4 },
        { price: 2.99, id: 5 },
        { price: 6.99, id: 6 } ],
      [ { price: 5.99, id: 21 },
        { price: 1.99, id: 22 },
        { price: 1.99, id: 23 },
        { price: 1.99, id: 24 },
        { price: 5.99, id: 25 } ],

      // ...

      [ { price: 7.99, id: 221 },
        { price: 4.99, id: 222 },
        { price: 7.99, id: 223 },
        { price: 10.99, id: 224 },
        { price: 9.99, id: 225 },
        { price: 9.99, id: 226 } ] ];
``` 
We could nest some `mapWith`s, but we humans are tool users. If we can use a stick to extract tasty ants from a hole to eat, we can automate working with arrays:
```js
    const deepMapWith = (fn) =>
      function innerdeepMapWith (tree) {
        return Array.prototype.map.call(tree, (element) =>
          Array.isArray(element)
            ? innerdeepMapWith(element)
            : fn(element)
        );
      };
```       
And now we can use `deepMapWith` on a tree the way we use `mapWith` on a flat array:
```js
    deepMapWith(getWith('price'))(report)
      //=>  [ [ 1.99,
                4.99,
                7.99,
                1.99,
                2.99,
                6.99 ],
              [ 5.99,
                1.99,
                1.99,
                1.99,
                5.99 ],
                
              // ...
              
              [ 7.99,
                4.99,
                7.99,
                10.99,
                9.99,
                9.99 ] ]
``` 
We'll have another look at trees of data when we look at TreeIterators for [Collections](#collections).
