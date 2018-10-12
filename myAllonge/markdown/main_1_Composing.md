# Composing and Decomposing Data

![Stacked Cups](images/stacked-cups.jpg)

> Recursion is the root of computation since it trades description for time.—Alan Perlis, [Epigrams in Programming](http://www.cs.yale.edu/homes/perlis-alan/quotes.html)
## Arrays and Destructuring Arguments

While we have mentioned arrays briefly, we haven't had a close look at them. **Arrays are JavaScript's "native" representation of lists.** Strings are important because they represent writing. Lists are important because they represent **ordered collections** of things, and **ordered collections** are a fundamental abstraction for making sense of reality.

### array literals

JavaScript has a literal syntax for creating an array: The `[` and `]` characters. We can create an empty array:
```js
    []
      //=> []
```
We can create an array with one or more *elements* by placing them between the brackets and separating the items with commas. Whitespace is optional:
```js
    [1]
      //=> [1]

    [2, 3, 4]
      //=> [2,3,4]
```
Any expression will work:
```js
    [ 2,
      3,
      2 + 2
    ]
      //=> [2,3,4]
```
Including an expression denoting another array:
```js
    [[[[[]]]]]
```
This is an array with one element that is an array with one element that is an array with one element that is an array with one element that is an empty array. Although that seems like something nobody would ever construct, many students have worked with almost the exact same thing when they explored various means of constructing arithmetic from Set Theory.

Any expression will do, including names:
```js
    const wrap = (something) => [something];

    wrap("lunch")
      //=> ["lunch"]
```
Array literals are expressions, and arrays are *reference types*. We can see that each time an array literal is evaluated, we get a new, distinct array, even if it contains the exact same elements:
```js
    [] === []
      //=> false

    [2 + 2] === [2 + 2]
      //=> false

    const array_of_one = () => [1];

    array_of_one() === array_of_one()
      //=> false
```
### element references

Array elements can be extracted using `[` and `]` as postfix operators. We pass an integer as an index of the element to extract:
```js
    const oneTwoThree = ["one", "two", "three"];
    oneTwoThree[0]
      //=> 'one'
    oneTwoThree[1]
      //=> 'two'
    oneTwoThree[2]
      //=> 'three'
```
As we can see, JavaScript Arrays are [zero-based].

[zero-based]: https://en.wikipedia.org/wiki/Zero-based_numbering

We know that every array is its own unique entity, with its own unique reference. What about the contents of an array? Does it store references to the things we give it? Or copies of some kind?
```js
    const x = [],
          a = [x];
    a[0] === x
        //=> true, arrays store references to the things you put in them.
```
### destructuring arrays

There is another way to extract elements from arrays: *Destructuring*, a feature going back to Common Lisp, if not before. We saw how to construct an array literal using `[`, expressions, `,` and `]`. Here's an example of an array literal that uses a name:
```js
    const wrap = (something) => [something];
```
Let's expand it to use a block and an extra name:
```js
    const wrap = (something) => {
      const wrapped = [something];

      return wrapped;
    }
    wrap("package")
      //=> ["package"]
```
The line `const wrapped = [something];` is interesting. On the left hand is a name to be bound, and on the right hand is an array literal, a template for constructing an array, very much like a quasi-literal string.

In JavaScript, we can actually *reverse* the statement and place the template on the left and a value on the right:
```js
    const unwrap = (wrapped) => {
      const [something] = wrapped;

      return something;
    }
    unwrap(["present"])
      //=> "present"
```
The statement `const [something] = wrapped;` *destructures* the array represented by `wrapped`, binding the value of its single element to the name `something`. We can do the same thing with more than one element:
```js
    const surname = (name) => {
      const [first, last] = name;

      return last;
    }
    surname(["Reginald", "Braithwaite"])
      //=> "Braithwaite"
```
We could do the same thing with `(name) => name[1]`, but destructuring is code that resembles the data it consumes, a valuable coding style.

Destructuring can nest:
```js
    const description = (nameAndOccupation) => {
      const [[first, last], occupation] = nameAndOccupation;

      return `${first} is a ${occupation}`;
    }
    description([["Reginald", "Braithwaite"], "programmer"])
      //=> "Reginald is a programmer"
```
### gathering

Sometimes we need to extract arrays from arrays. Here is the most common pattern: Extracting the head and gathering everything but the head from an array:
```js
    const [car, ...cdr] = [1, 2, 3, 4, 5];

    car
      //=> 1
    cdr
      //=> [2, 3, 4, 5]
```
[`car` and `cdr`](https://en.wikipedia.org/wiki/CAR_and_CDR) are archaic terms that go back to an implementation of Lisp running on the IBM 704 computer. Some other languages call them `first` and `butFirst`, or `head` and `tail`. We will use a common convention and call variables we gather `rest`, but refer to the `...` operation as a "gather," following Kyle Simpson's example.`1`

>`1` Kyle Simpson is the author of [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/README.md#you-dont-know-js-book-series), available

Alas, the `...` notation does not provide a universal patten-matching capability. For example, we cannot write
```js
    const [...butLast, last] = [1, 2, 3, 4, 5];
      //=> ERROR

    const [first, ..., last] = [1, 2, 3, 4, 5];
      //=> ERROR
```
Now, when we introduced destructuring, we saw that it is kind-of-sort-of the reverse of array literals. So if
```js
    const wrapped = [something];
```
Then:
```js
    const [unwrapped] = something;
```
What is the reverse of gathering? We know that:
```js
    const [car, ...cdr] = [1, 2, 3, 4, 5];
```
What is the reverse? It would be:
```js
    const cons = [car, ...cdr];
```
Let's try it:
```js
const oneTwoThree = ["one", "two", "three"];

["zero", ...oneTwoThree]
  //=> ["zero","one","two","three"]
```
It works! We can use `...` to place the elements of an array inside another array. We say that using `...` to destructure is gathering, and using it in a literal to insert elements is called "spreading."

### destructuring is not pattern matching

Some other languages have something called *pattern matching*, where you can write something like a destructuring assignment, and the language decides whether the "patterns" matches at all. If it does, assignments are made where appropriate.

In such a language, if you wrote something like:
```js
    const [what] = [];
```
That match would fail because the array doesn't have an element to assign to `what`. But this is not how JavaScript works. JavaScript tries its best to assign things, and if there isn't something that fits, JavaScript binds `undefined` to the name. Therefore:
```js
    const [what] = [];

    what
      //=> undefined

    const [which, what, who] = ["duck feet", "tiger tail"];

    who
      //=> undefined
```
And if there aren't any items to assign with `...`, JavaScript assigns an empty array:
```js
    const [...they] = [];

    they
      //=> []

    const [which, what, ...they] = ["duck feet", "tiger tail"];

    they
      //=> []
```
From its very inception, JavaScript has striven to avoid catastrophic errors. As a result, it often coerces values, passes `undefined` around, or does whatever it can to keep executing without failing. This often means that we must write our own code to detect failure conditions, as we cannot reply on the language to point out when we are doing semantically meaningless things.

### destructuring and return values

Some languages support multiple return values: A function can return several things at once, like a value and an error code. This can easily be emulated in JavaScript with destructuring:
```js
    const description = (nameAndOccupation) => {
      if (nameAndOccupation.length < 2) {
        return ["", "occupation missing"]
      }
      else {
        const [[first, last], occupation] = nameAndOccupation;

        return [`${first} is a ${occupation}`, "ok"];
      }
    }

    const [reg, status] = description([["Reginald", "Braithwaite"], "programmer"]);

    reg
      //=> "Reginald is a programmer"

    status
       //=> "ok"
```
### destructuring parameters

Consider the way we pass arguments to parameters:
```js
    foo()
    bar("smaug")
    baz(1, 2, 3)
```
It is very much like an array literal. And consider how we bind values to parameter names:
```js
    const foo = () => ...
    const bar = (name) => ...
    const baz = (a, b, c) => ...
```
It *looks* like destructuring. It acts like destructuring. There is only one difference: We have not tried gathering. Let's do that:
```js
    const numbers = (...nums) => nums;

    numbers(1, 2, 3, 4, 5)
      //=> [1,2,3,4,5]

    const headAndTail = (head, ...tail) => [head, tail];

    headAndTail(1, 2, 3, 4, 5)
      //=> [1,[2,3,4,5]]
```
Gathering works with parameters! This is very useful indeed, and we'll see more of it in a moment.`1`

>`1` Gathering in parameters has a long history, and the usual terms are to call gathering "pattern matching" and to call a name that is bound to gathered values a "rest parameter." The term "rest" is perfectly compatible with gather: "Rest" is the noun, and "gather" is the verb. We *gather* the *rest* of the parameters.

## Self-Similarity    
> Recursion is the root of computation since it trades description for time.—Alan Perlis, [Epigrams in Programming](http://www.cs.yale.edu/homes/perlis-alan/quotes.html)

In [Arrays and Destructuring Arguments](#arrays-and-destructuring-arguments), we worked with the basic idea that putting an array together with a literal array expression was the reverse or opposite of taking it apart with a destructuring assignment.

We saw that the basic idea that putting an array together with a literal array expression was the reverse or opposite of taking it apart with a destructuring assignment.

Let's be more specific. Some data structures, like lists, can obviously be seen as a collection of items. Some are empty, some have three items, some forty-two, some contain numbers, some contain strings, some a mixture of elements, there are all kinds of lists.

But we can also define a list by describing a rule for building lists. One of the simplest, and longest-standing in computer science, is to say that a list is:

1. Empty, or;
1. Consists of an element concatenated with a list .

Let's convert our rules to array literals. The first rule is simple: `[]` is a list. How about the second rule? We can express that using a spread. Given an element `e` and a list `list`, `[e, ...list]` is a list. We can test this manually by building up a list:
```js
    []
    //=> []

    ["baz", ...[]]
    //=> ["baz"]

    ["bar", ...["baz"]]
    //=> ["bar","baz"]

    ["foo", ...["bar", "baz"]]
    //=> ["foo","bar","baz"]
 ``` 
Thanks to the parallel between array literals + spreads with destructuring + rests, we can also use the same rules to decompose lists:
```js
    const [first, ...rest] = [];
    first
      //=> undefined
    rest
      //=> []:

    const [first, ...rest] = ["foo"];
    first
      //=> "foo"
    rest
      //=> []

    const [first, ...rest] = ["foo", "bar"];
    first
      //=> "foo"
    rest
      //=> ["bar"]

    const [first, ...rest] = ["foo", "bar", "baz"];
    first
      //=> "foo"
    rest
      //=> ["bar","baz"]
 ```
For the purpose of this exploration, we will presume the following:`1`
```js
    const isEmpty = ([first, ...rest]) => first === undefined;

    isEmpty([])
      //=> true

    isEmpty([0])
      //=> false

    isEmpty([[]])
      //=> false
``` 
>`1` Well, actually, this does not work for arrays that contain `undefined` as a value, but we are not going to see that in our examples. A more robust implementation would be `(array) => array.length === 0`, but we are doing backflips to keep this within a very small and contrived playground.
    
Armed with our definition of an empty list and with what we've already learned, we can build a great many functions that operate on arrays. We know that we can get the length of an array using its `.length`. But as an exercise, how would we write a `length` function using just what we have already?

First, we pick what we call a *terminal case*. What is the length of an empty array? `0`. So let's start our function with the observation that if an array is empty, the length is `0`:
```js    
    const length = ([first, ...rest]) =>
      first === undefined
        ? 0
        : // ???
``` 
We need something for when the array isn't empty. If an array is not empty, and we break it into two pieces, `first` and `rest`, the length of our array is going to be `length(first) + length(rest)`. Well, the length of `first` is `1`, there's just one element at the front. But we don't know the length of `rest`. If only there was a function we could call... Like `length`!
```js   
    const length = ([first, ...rest]) =>
      first === undefined
        ? 0
        : 1 + length(rest);
```  
Let's try it!
```js    
    length([])
      //=> 0
  
    length(["foo"])
      //=> 1
  
    length(["foo", "bar", "baz"])
      //=> 3
```
Our `length` function is *recursive*, it calls itself. This makes sense because our definition of a list is recursive, and if a list is self-similar, it is natural to create an algorithm that is also self-similar.

### linear recursion    

"Recursion" sometimes seems like an elaborate party trick. There's even a joke about this:

> When promising students are trying to choose between pure mathematics and applied engineering, they are given a two-part aptitude test. In the first part, they are led to a laboratory bench and told to follow the instructions printed on the card. They find a bunsen burner, a sparker, a tap, an empty beaker, a stand, and a card with the instructions "boil water."

> Of course, all the students know what to do: They fill the beaker with water, place the stand on the burner and the beaker on the stand, then they turn the burner on and use the sparker to ignite the flame. After a bit the water boils, and they turn off the burner and are lead to a second bench.

> Once again, there is a card that reads, "boil water." But this time, the beaker is on the stand over the burner, as left behind by the previous student. The engineers light the burner immediately. Whereas the mathematicians take the beaker off the stand and empty it, thus reducing the situation to a problem they have already solved.

There is more to recursive solutions that simply functions that invoke themselves. Recursive algorithms follow the "divide and conquer" strategy for solving a problem:

1. Divide the problem into smaller problems
1. If a smaller problem is solvable, solve the small problem
1. If a smaller problem is not solvable, divide and conquer that problem
1. When all small problems have been solved, compose the solutions into one big solution

The big elements of divide and conquer are a method for decomposing a problem into smaller problems, a test for the smallest possible problem, and a means of putting the pieces back together. Our solutions are a little simpler in that we don't really break a problem down into multiple pieces, we break a piece off the problem that may or may not be solvable, and solve that before sticking it onto a solution for the rest of the problem.

This simpler form of "divide and conquer" is called *linear recursion*. It's very useful and simple to understand. Let's take another example. Sometimes we want to *flatten* an array, that is, an array of arrays needs to be turned into one array of elements that aren't arrays.`1`

>`1` `flatten` is a very simple [unfold](https://en.wikipedia.org/wiki/Anamorphism), a function that takes a seed value and turns it into an array. Unfolds can be thought of a "path" through a data structure, and flattening a tree is equivalent to a depth-first traverse.

We already know how to divide arrays into smaller pieces. How do we decide whether a smaller problem is solvable? We need a test for the terminal case. Happily, there is something along these lines provided for us:
```js
    Array.isArray("foo")
      //=> false
      
    Array.isArray(["foo"])
      //=> true
```      
The usual "terminal case" will be that flattening an empty array will produce an empty array. The next terminal case is that if an element isn't an array, we don't flatten it, and can put it together with the rest of our solution directly. Whereas if an element is an array, we'll flatten it and put it together with the rest of our solution.

So our first cut at a `flatten` function will look like this:
```js
    const flatten = ([first, ...rest]) => {
      if (first === undefined) {
        return [];
      }
      else if (!Array.isArray(first)) {
        return [first, ...flatten(rest)];
      }
      else {
        return [...flatten(first), ...flatten(rest)];
      }
    }
    
    flatten(["foo", [3, 4, []]])
      //=> ["foo",3,4]
```     
Once again, the solution directly displays the important elements: Dividing a problem into subproblems, detecting terminal cases, solving the terminal cases, and composing a solution from the solved portions.

### mapping

Another common problem is applying a function to every element of an array. JavaScript has a built-in function for this, but let's write our own using linear recursion.

If we want to square each number in a list, we could write:
```js
    const squareAll = ([first, ...rest]) => first === undefined
                                                ? []
                                                : [first * first, ...squareAll(rest)];
    squareAll([1, 2, 3, 4, 5])
       //=> [1,4,9,16,25]
```
And if we wanted to "truthify" each element in a list, we could write:
```js
    const truthyAll = ([first, ...rest]) => first === undefined
                                                ? []
                                                : [!!first, ...truthyAll(rest)];
    truthyAll([null, true, 25, false, "foo"])
      //=> [false,true,true,false,true]
```                                                
This specific case of linear recursion is called "mapping," and it is not necessary to constantly write out the same pattern again and again. Functions can take functions as arguments, so let's "extract" the thing to do to each element and separate it from the business of taking an array apart, doing the thing, and putting the array back together.

Given the signature:

    const mapWith = (fn, array) => // ...
    
We can write it out using a ternary operator. Even in this small function, we can identify the terminal condition, the piece being broken off, and recomposing the solution.
```js
    const mapWith = (fn, [first, ...rest]) =>
      first === undefined
        ? []
        : [fn(first), ...mapWith(fn, rest)];
                                                  
    mapWith((x) => x * x, [1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
      
    mapWith((x) => !!x, [null, true, 25, false, "foo"])
      //=> [false,true,true,false,true]
```
### folding

With the exception of the `length` example at the beginning, our examples so far all involve rebuilding a solution using spreads.  But they needn't. A function to compute the sum of the squares of a list of numbers might look like this:
```js
    const sumSquares = ([first, ...rest]) => first === undefined
                                             ? 0
                                             : first * first + sumSquares(rest);
                                             
    sumSquares([1, 2, 3, 4, 5])
      //=> 55
```
There are two differences between `sumSquares` and our maps above:

1. Given the terminal case of an empty list, we return a `0` instead of an empty list, and;
1. We catenate the square of each element to the result of applying `sumSquares` to the rest of the elements.

Let's rewrite `mapWith` so that we can use it to sum squares.
```js
    const foldWith = (fn, terminalValue, [first, ...rest]) =>
      first === undefined
        ? terminalValue
        : fn(first, foldWith(fn, terminalValue, rest));
```                                                           
And now we supply a function that does slightly more than our mapping functions:
```js
    foldWith((number, rest) => number * number + rest, 0, [1, 2, 3, 4, 5])
      //=> 55
```    
Our `foldWith` function is a generalization of our `mapWith` function. We can represent a map as a fold, we just need to supply the array rebuilding code:
```js
    const squareAll = (array) => foldWith((first, rest) => [first * first, ...rest], [], array);
    
    squareAll([1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
```    
And if we like, we can write `mapWith` using `foldWith`:
```js
    const mapWith = (fn, array) => foldWith((first, rest) => [fn(first), ...rest], [], array),
          squareAll = (array) => mapWith((x) => x * x, array);
    
    squareAll([1, 2, 3, 4, 5])
      //=> [1,4,9,16,25]
```              
And to return to our first example, our version of `length` can be written as a fold:
```js
    const length = (array) => foldWith((first, rest) => 1 + rest, 0, array);
    
    length([1, 2, 3, 4, 5])
      //=> 5
```              
### summary

Linear recursion is a basic building block of algorithms. Its basic form parallels the way linear data structures like lists are constructed: This helps make it understandable. Its specialized cases of mapping and folding are especially useful and can be used to build other functions. And finally, while folding is a special case of linear recursion, mapping is a special case of folding.

## Tail Calls (and Default Arguments)
The `mapWith` and `foldWith` functions we wrote in [Self-Similarity](#self-similarity) are useful for illustrating the basic principles behind using recursion to work with self-similar data structures, but they are not "production-ready" implementations. One of the reasons they are not production-ready is that they consume memory proportional to the size of the array being folded.

Let's look at how. Here's our extremely simple `mapWith` function again:

```js
const mapWith = (fn, [first, ...rest]) =>
  first === undefined
    ? []
    : [fn(first), ...mapWith(fn, rest)];

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
```

Let's step through its execution. First, `mapWith((x) => x * x, [1, 2, 3, 4, 5])` is invoked. `first` is not `undefined`, so it evaluates [fn(first), ...mapWith(fn, rest)]. To do that, it has to evaluate `fn(first)` and `mapWith(fn, rest)`, then evaluate `[fn(first), ...mapWith(fn, rest)]`.

This is roughly equivalent to writing:
```js
const mapWith = function (fn, [first, ...rest]) {
  if (first === undefined) {
    return [];
  }
  else {
    const _temp1 = fn(first),
          _temp2 = mapWith(fn, rest),
          _temp3 = [_temp1, ..._temp2];

    return _temp3;
  }
}
```
Note that while evaluating `mapWith(fn, rest)`, JavaScript must retain the value `first` or `fn(first)`, plus some housekeeping information so it remembers what to do with `mapWith(fn, rest)` when it has a result. JavaScript cannot throw `first` away. So we know that JavaScript is going to hang on to `1`.

Next, JavaScript invokes `mapWith(fn, rest)`, which is semantically equivalent to `mapWith((x) => x * x, [2, 3, 4, 5])`. And the same thing happens: JavaScript has to hang on to `2` (or `4`, or both, depending on the implementation), plus some housekeeping information so it remembers what to do with that value, while it calls the equivalent of `mapWith((x) => x * x, [3, 4, 5])`.

This keeps on happening, so that JavaScript collects the values `1`, `2`, `3`, `4`, and `5` plus housekeeping information by the time it calls `mapWith((x) => x * x, [])`. It can start assembling the resulting array and start discarding the information it is saving.

That information is saved on a *call stack*, and it is quite expensive. Furthermore, doubling the length of an array will double the amount of space we need on the stack, plus double all the work required to set up and tear down the housekeeping data for each call (these are called *call frames*, and they include the place where the function was called, an environment, and so on).

In practice, using a method like this with more than about 50 items in an array may cause some implementations to run very slow, run out of memory and freeze, or cause an error.

```js
mapWith((x) => x * x, [
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
  10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
  20, 21, 22, 23, 24, 25, 26, 27, 28, 29,
  30, 31, 32, 33, 34, 35, 36, 37, 38, 39,
  40, 41, 42, 43, 44, 45, 46, 47, 48, 49,
  50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69,
  70, 71, 72, 73, 74, 75, 76, 77, 78, 79,
  80, 81, 82, 83, 84, 85, 86, 87, 88, 89,
  90, 91, 92, 93, 94, 95, 96, 97, 98, 99,
   0,  1,  2,  3,  4,  5,  6,  7,  8,  9,
  10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
  20, 21, 22, 23, 24, 25, 26, 27, 28, 29,
  30, 31, 32, 33, 34, 35, 36, 37, 38, 39,
  40, 41, 42, 43, 44, 45, 46, 47, 48, 49,
  50, 51, 52, 53, 54, 55, 56, 57, 58, 59,
  60, 61, 62, 63, 64, 65, 66, 67, 68, 69,
  70, 71, 72, 73, 74, 75, 76, 77, 78, 79,
  80, 81, 82, 83, 84, 85, 86, 87, 88, 89,
  90, 91, 92, 93, 94, 95, 96, 97, 98, 99
])
  //=> ???
```
Is there a better way? Yes. In fact, there are several better ways. Making algorithms faster is a very highly studied field of computer science. The one we're going to look at here is called *tail-call optimization*, or "TCO."

### tail-call optimization

A "tail-call" occurs when a function's last act is to invoke another function, and then return whatever the other function returns. For example, consider the `maybe` function decorator:
```js
const maybe = (fn) =>
  function (...args) {
    if (args.length === 0) {
      return;
    }
    else {
      for (let arg of args) {
        if (arg == null) return;
      }
      return fn.apply(this, args);
    }
  }
```
There are three places it returns. The first two don't return anything, they don't matter. But the third is `fn.apply(this, args)`. This is a tail-call, because it invokes another function and returns its result. This is interesting, because after sorting out what to supply as arguments (`this`, `args`), JavaScript can throw away everything in its current stack frame. It isn't going to do any more work, so it can throw its existing stack frame away.

And in fact, it does exactly that: It throws the stack frame away, and does not consume extra memory when making a `maybe`-wrapped call. This is a very important characteristic of JavaScript: **If a function makes a call in tail position, JavaScript optimizes away the function call overhead and stack space.**

That is excellent, but one wrapping is not a big deal. When would we really care? Consider this implementation of `length`:
```js
const length = ([first, ...rest]) =>
  first === undefined
    ? 0
    : 1 + length(rest);
```
The `length` function calls itself, but it is not a tail-call, because it returns `1 + length(rest)`, not `length(rest)`.

The problem can be stated in such a way that the answer is obvious: `length` does not call itself in tail position, because it has to do two pieces of work, and while one of them is in the recursive call to `length`, the other happens after the recursive call.

The obvious solution?

### converting non-tail-calls to tail-calls

The obvious solution is push the `1 +` work into the call to `length`. Here's our first cut:

```js
const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
  first === undefined
    ? 0 + numberToBeAdded
    : lengthDelaysWork(rest, 1 + numberToBeAdded)

lengthDelaysWork(["foo", "bar", "baz"], 0)
  //=> 3
```
This `lengthDelaysWork` function calls itself in tail position. The `1 +` work is done before calling itself, and by the time it reaches the terminal position, it has the answer. Now that we've seen how it works, we can clean up the `0 + numberToBeAdded` business. But while we're doing that, it's annoying to remember to call it with a zero. Let's fix that:
```js
const lengthDelaysWork = ([first, ...rest], numberToBeAdded) =>
  first === undefined
    ? numberToBeAdded
    : lengthDelaysWork(rest, 1 + numberToBeAdded)

const length = (n) =>
  lengthDelaysWork(n, 0);
```
Or we could use partial application:
```js
const callLast = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const length = callLast(lengthDelaysWork, 0);

length(["foo", "bar", "baz"])
  //=> 3
```

This version of `length` calls uses `lengthDelaysWork`, and JavaScript optimizes that not to take up memory proportional to the length of the string. We can use this technique with `mapWith`:
```js
const mapWithDelaysWork = (fn, [first, ...rest], prepend) =>
  first === undefined
    ? prepend
    : mapWithDelaysWork(fn, rest, [...prepend, fn(first)]);

const mapWith = callLast(mapWithDelaysWork, []);

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
```
We can use it with ridiculously large arrays:
```js
mapWith((x) => x * x, [
     0,    1,    2,    3,    4,    5,    6,    7,    8,    9,
    10,   11,   12,   13,   14,   15,   16,   17,   18,   19,
    20,   21,   22,   23,   24,   25,   26,   27,   28,   29,
    30,   31,   32,   33,   34,   35,   36,   37,   38,   39,
    40,   41,   42,   43,   44,   45,   46,   47,   48,   49,
    50,   51,   52,   53,   54,   55,   56,   57,   58,   59,
    60,   61,   62,   63,   64,   65,   66,   67,   68,   69,
    70,   71,   72,   73,   74,   75,   76,   77,   78,   79,
    80,   81,   82,   83,   84,   85,   86,   87,   88,   89,
    90,   91,   92,   93,   94,   95,   96,   97,   98,   99,

  // ...

  2980, 2981, 2982, 2983, 2984, 2985, 2986, 2987, 2988, 2989,
  2990, 2991, 2992, 2993, 2994, 2995, 2996, 2997, 2998, 2999 ])

  //=> [0,1,4,9,16,25,36,49,64,81,100,121,144,169,196, ...
```
Brilliant! We can map over large arrays without incurring all the memory and performance overhead of non-tail-calls. And this basic transformation from a recursive function that does not make a tail call, into a recursive function that calls itself in tail position, is a bread-and-butter pattern for programmers using a language that incorporates tail-call optimization.

### factorials

Introductions to recursion often mention calculating factorials:    
> In mathematics, the factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`. For example:
```js
5! = 5  x  4  x  3  x  2  x  1 = 120.
```
The naïve function for calcuating the factorial of a positive integer follows directly from the definition:
```js
const factorial = (n) =>
  n == 1
  ? n
  : n * factorial(n - 1);

factorial(1)
  //=> 1

factorial(5)
  //=> 120
```

While this is mathematically elegant, it is computational `1`.

>`1` https://en.wikipedia.org/wiki/Filigree

Once again, it is not tail-recursive, it needs to save the stack with each invocation so that it can take the result returned and compute `n * factorial(n - 1)`. We can do the same conversion, pass in the work to be done:
```js
const factorialWithDelayedWork = (n, work) =>
  n === 1
  ? work
  : factorialWithDelayedWork(n - 1, n * work);

const factorial = (n) =>
  factorialWithDelayedWork(n, 1);
```
Or we could use partial application:
```js
const callLast = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const factorial = callLast(factorialWithDelayedWork, 1);

factorial(1)
  //=> 1

factorial(5)
  //=> 120
```
As before, we wrote a `factorialWithDelayedWork` function, then used partial application (`callLast`) to make a `factorial` function that took just the one argument and supplied the initial work value.

### default arguments

Our problem is that we can directly write:
```js
const factorial = (n, work) =>
  n === 1
  ? work
  : factorial(n - 1, n * work);

factorial(1, 1)
  //=> 1

factorial(5, 1)
  //=> 120
```
But it is hideous to have to always add a `1` parameter, we'd be demanding that everyone using the `factorial` function know that we are using a tail-recursive implementation.

What we really want is this: We want to write something like `factorial(6)`, and have JavaScript automatically know that we really mean `factorial(6, 1)`. But when it calls itself, it will call `factorial(5, 6)` and that will not mean `factorial(5, 1)`.

JavaScript provides this exact syntax, it's called a *default argument*, and it looks like this:
```js
const factorial = (n, work = 1) =>
  n === 1
  ? work
  : factorial(n - 1, n * work);

factorial(1)
  //=> 1

factorial(6)
  //=> 720
```
By writing our parameter list as `(n, work = 1) =>`, we're stating that if a second parameter is not provided, `work` is to be bound to `1`. We can do similar things with our other tail-recursive functions:
```js
const length = ([first, ...rest], numberToBeAdded = 0) =>
  first === undefined
    ? numberToBeAdded
    : length(rest, 1 + numberToBeAdded)

length(["foo", "bar", "baz"])
  //=> 3

const mapWith = (fn, [first, ...rest], prepend = []) =>
  first === undefined
    ? prepend
    : mapWith(fn, rest, [...prepend, fn(first)]);

mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
```
Now we don't need to use two functions. A default argument is concise and readable.

### defaults and destructuring

We saw earlier that destructuring parameters works the same way as destructuring assignment. Now we learn that we can create a default parameter argument. Can we create a default destructuring assignment?
```js
const [first, second = "two"] = ["one"];

`${first} . ${second}`
  //=> "one . two"

const [first, second = "two"] = ["primus", "secundus"];

`${first} . ${second}`
  //=> "primus . secundus"
```
How very useful: defaults can be supplied for destructuring assignments, just like defaults for parameters.

## Garbage, Garbage Everywhere
We have now seen how to use [Tail Calls](#tail-calls-and-default-arguments) to execute `mapWith` in constant space:
```js
const mapWith = (fn, [first, ...rest], prepend = []) =>
  first === undefined
    ? prepend
    : mapWith(fn, rest, [...prepend, fn(first)]);
                                                  
mapWith((x) => x * x, [1, 2, 3, 4, 5])
  //=> [1,4,9,16,25]
```
But when we try it on very large arrays, we discover that it is *still* very slow. Much slower than the built-in `.map` method for arrays. The right tool to discover why it's still slow is a memory profiler, but a simple inspection of the program will reveal the following:

Every time we call `mapWith`, we're calling `[...prepend, fn(first)]`. To do that, we take the array in `prepend` and push `fn(first)` onto the end, creating a new array that will be passed to the next invocation of `mapWith`.

Worse, the JavaScript Engine actually copies the elements from `prepend` into the new array one at a time. That is very laborious.`1`

>`1` It needn't always be so: Programmers have developed specialized data structures that make operations like this cheap, often by arranging for structures to share common elements by default, and only making copies when changes are made. But this is not how JavaScript's built-in arrays work.

The array we had in `prepend` is no longer used. In GC environments, it is marked as no longer being used, and eventually the garbage collector recycles the memory it is using. Lather, rinse, repeat: Ever time we call `mapWith`, we're creating a new array, copying all the elements from `prepend` into the new array, and then we no longer use `prepend`.

We may not be creating 3,000 stack frames, but we are creating three thousand new arrays and copying elements into each and every one of them. Although the maximum amount of memory does not grow, the thrashing as we create short-lived arrays is very bad, and we do a lot of work copying elements from one array to another.

> **Key Point**: Our `[first, ...rest]` approach to recursion is slow because that it creates a lot of temporary arrays, and it spends an enormous amount of time copying elements into arrays that end up being discarded. 

So here's a question: If this is such a slow approach, why do some examples of "functional" algorithms work this exact way?

![The IBM 704](images/IBM704b.jpg)

### some history

Once upon a time, there was a programming language called [Lisp], an acronym for LISt Processing.[^lisp] Lisp was one of the very first high-level languages, the very first implementation was written for the [IBM 704] computer. (The very first FORTRAN implementation was also written for the 704).

[Lisp]: https://en.wikipedia.org/wiki/Lisp_(programming_language)
[IBM 704]: https://en.wikipedia.org/wiki/IBM_704
[^lisp]: Lisp is still very much alive, and one of the most interesting and exciting programming languages in use today is [Clojure](http://clojure.org/), a Lisp dialect that runs on the JVM, along with its sibling [ClojureScript](https://github.com/clojure/clojurescript), Clojure that transpiles to JavaScript.

The 704 had a 36-bit word, meaning that it was very fast to store and retrieve 36-bit values. The CPU's instruction set featured two important macros: `CAR` would fetch 15 bits representing the Contents of the Address part of the Register, while `CDR` would fetch the Contents of the Decrement part of the Register.

In broad terms, this means that a single 36-bit word could store two separate 15-bit values and it was very fast to save and retrieve pairs of values. If you had two 15-bit values and wished to write them to the register, the `CONS` macro would take the values and write them to a 36-bit word.

Thus, `CONS` put two values together, `CAR` extracted one, and `CDR` extracted the other. Lisp's basic data type is often said to be the list, but in actuality it was the "cons cell," the term used to describe two 15-bit values stored in one word. The 15-bit values were used as pointers that could refer to a location in memory, so in effect, a cons cell was a little data structure with two pointers to other cons cells.

Lists were represented as linked lists of cons cells, with each cell's head pointing to an element and the tail pointing to another cons cell.

> Having these instructions be very fast was important to those early designers: They were working on one of the first high-level languages (COBOL and FORTRAN being the others), and computers in the late 1950s were extremely small and slow by today's standards. Although the 704 used core memory, it still used vacuum tubes for its logic. Thus, the design of programming languages and algorithms was driven by what could be accomplished with limited memory and performance.

Here's the scheme in JavaScript, using two-element arrays to represent cons cells:
```js
const cons = (a, d) => [a, d],
      car  = ([a, d]) => a,
      cdr  = ([a, d]) => d;
``` 
We can make a list by calling `cons` repeatedly, and terminating it with `null`:

```js
const oneToFive = cons(1, cons(2, cons(3, cons(4, cons(5, null)))));

oneToFive
  //=> [1,[2,[3,[4,[5,null]]]]]
```
Notice that though JavaScript displays our list as if it is composed of arrays nested within each other like Russian Dolls, in reality the arrays refer to each other with references, so `[1,[2,[3,[4,[5,null]]]]]` is actually more like:

```js
const node5 = [5,null],
      node4 = [4, node5],
      node3 = [3, node4],
      node2 = [2, node3],
      node1 = [1, node2];
    
```
This is a [Linked List](https://en.wikipedia.org/wiki/Linked_list), it's just that those early Lispers used the names `car` and `cdr` after the hardware instructions, whereas today we use words like `data` and `reference`. But it works the same way: If we want the head of a list, we call `car` on it:
```js
car(oneToFive)
  //=> 1
```
`car` is very fast, it simply extracts the first element of the cons cell.

But what about the rest of the list? `cdr` does the trick:
```js
cdr(oneToFive)
  //=> [2,[3,[4,[5,null]]]]
```
Again, it's just extracting a reference from a cons cell, it's very fast. In Lisp, it's blazingly fast because it happens in hardware. There's no making copies of arrays, the time to `cdr` a list with five elements is the same as the time to `cdr` a list with 5,000 elements, and no temporary arrays are needed. In JavaScript, it's still much, much, much faster to get all the elements except the head from a linked list than from an array. Getting one reference to a structure that already exists is faster than copying a bunch of elements.

So now we understand that in Lisp, a lot of things use linked lists, and they do that in part because it was what the hardware made possible.

Getting back to JavaScript now, when we write `[first, ...rest]` to gather or spread arrays, we're emulating the semantics of `car` and `cdr`, but not the implementation. We're doing something laborious and memory-inefficient compared to using a linked list as Lisp did and as we can still do if we choose.

That being said, it is easy to understand and helps us grasp how literals and destructuring works, and how recursive algorithms ought to mirror the self-similarity of the data structures they manipulate. And so it is today that languages like JavaScript have arrays that are slow to split into the equivalent of a `car`/`cdr` pair, but instructional examples of recursive programs still have echoes of their Lisp origins.

We'll look at linked lists again when we look at [Plain Old JavaScript Objects](#plain-old-javascript-objects).

### so why arrays

If `[first, ...rest]` is so slow, why does JavaScript use arrays instead of making everything a linked list?

Well, linked lists are fast for a few things, like taking the front element off a list, and taking the remainder of a list. But not for iterating over a list: Pointer chasing through memory is quite a bit slower than incrementing an index. In addition to the extra fetches to dereference pointers, pointer chasing suffers from cache misses. And if you want an arbitrary item from a list, you have to iterate through the list element by element, whereas with the indexed array you just fetch it.

We have avoided discussing rebinding and mutating values, but if we want to change elements of our lists, the naïve linked list implementation suffers as well: When we take the `cdr` of a linked list, we are sharing the elements. If we make any change other than cons-ing a new element to the front, we are changing both the new list and the old list.

Arrays avoid this problem by pessimistically copying all the references whenever we extract an element or sequence of elements from them (We'll see this explained later in [Mutation](#mutation)).

For these and other reasons, almost all languages today make it possible to use a fast array or vector type that is optimized for iteration, and even Lisp now has a variety of data structures that are optimized for specific use cases.

### summary

Although we showed how to use tail calls to map and fold over arrays with `[first, ...rest]`, in reality this is not how it ought to be done. But it is an extremely simple illustration of how recursion works when you have a self-similar means of constructing a data structure.

## Plain Old JavaScript Objects

Lists are not the only way to represent collections of things, but they are the "oldest" data structure in the history of high level languages, because they map very closely to the way the hardware is organized in a computer. Lists are obviously very handy for homogeneous collections of things, like a shopping list:
```js
const remember = ["the milk", "the coffee beans", "the biscotti"];
```
And they can be used to store heterogeneous things in various levels of structure:
```js
const user = [["Reginald", "Braithwaite"],[ "author"
    ,["JavaScript Allongé", "JavaScript Spessore", "CoffeeScript Ristretto"]]];
```
Remembering that the name is the first item is error-prone, and being expected to look at `user[0][1]` and know that we are talking about a surname is unreasonable. So back when lists were the only things available, programmers would introduce constants to make things easier on themselves:
```js
const NAME = 0,
      FIRST = 0,
      LAST = 1,
      OCCUPATION = 1,
      TITLE = 0,
      RESPONSIBILITIES = 1;

const user = [["Reginald", "Braithwaite"]
        ,[ "author", ["JavaScript Allongé", "JavaScript Spessore", "CoffeeScript Ristretto"]]];
```
Now they could write `user[NAME][LAST]` or `user[OCCUPATION][TITLE]` instead of `user[0][1]` or `user[1][0]`. Over time, this need to build heterogeneous data structures with access to members by name evolved into the [Dictionary] data type, a mapping from a unique set of objects to another set of objects.

[Dictionary]: https://en.wikipedia.org/wiki/Associative_array

Dictionaries store key-value pairs, so instead of binding `NAME` to `0` and then storing a name in an array at index `0`, we can bind a name directly to `name` in a dictionary, and we let JavaScript sort out whether the implementation is a list of key-value pairs, a hashed collection, a tree of some sort, or anything else.

JavaScript has dictionaries, and it calls them "objects." The word "object" is loaded in programming circles, due to the widespread use of the term "object-oriented programming" that was coined by Alan Kay but has since come to mean many, many things to many different people.

In JavaScript, an object is a map from string keys to values.

### literal object syntax

JavaScript has a literal syntax for creating objects. This object maps values to the keys `year`, `month`, and `day`:
```js
    { year: 2012, month: 6, day: 14 }
```
Two objects created with separate evaluations have differing identities, just like arrays:
```js
    { year: 2012, month: 6, day: 14 } === { year: 2012, month: 6, day: 14 }
      //=> false
```
Objects use `[]` to access the values by name, using a string:
```js
    { year: 2012, month: 6, day: 14 }['day']
      //=> 14
```
Values contained within an object work just like values contained within an array, we access them by reference to the original:
```js
    const unique = () => [],
          x = unique(),
          y = unique(),
          z = unique(),
          o = { a: x, b: y, c: z };

    o['a'] === x && o['b'] === y && o['c'] === z
      //=> true
```
Names needn't be alphanumeric strings. For anything else, enclose the label in quotes:
```js
    { 'first name': 'reginald', 'last name': 'lewis' }['first name']
      //=> 'reginald'
```
If the name is an alphanumeric string conforming to the same rules as names of variables, there's a simplified syntax for accessing the values:
```js
    const date = { year: 2012, month: 6, day: 14 };

    date['day'] === date.day
      //=> true
```
Expressions can be used for keys as well. The syntax is to enclose the key's expression in `[` and `]`:
```js
    {
      ["p" + "i"]: 3.14159265
    }
      //=> {"pi":3.14159265}
```
All containers can contain any value, including functions or other containers, like a fat arrow function:
```js
    const Mathematics = {
      abs: (a) => a < 0 ? -a : a
    };

    Mathematics.abs(-5)
      //=> 5
```
Or proper functions:
```js
    const SecretDecoderRing = {
      encode: function (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode: function (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }
```
Or named function expressions:
```js
    const SecretDecoderRing = {
      encode: function encode (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode: function decode (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }
```
It is very common to associate named function expressions with keys in objects, and there is a "compact method syntax" for binding named function expressions to keywords:
```js
    const SecretDecoderRing = {
      encode (plaintext) {
        return plaintext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code + 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      },
      decode (cyphertext) {
        return cyphertext
          .split('')
          .map( char => char.charCodeAt() )
          .map( code => code - 1 )
          .map( code => String.fromCharCode(code) )
          .join('');
      }
    }
```
(There are some other technical differences between binding a named function expression and using compact method syntax, but they are not relevant here. We will generally prefer compact method syntax whenever we can.)

### destructuring objects

Just as we saw with arrays, we can write destructuring assignments with literal object syntax. So, we can write:
```js
const user = {
  name: { first: "Reginald",
          last: "Braithwaite"
        },
  occupation: { title: "Author",
                responsibilities: [ "JavaScript Allongé",
                                    "JavaScript Spessore",
                                    "CoffeeScript Ristretto"
                                  ]
              }
};
user.name.last
  //=> "Braithwaite"
user.occupation.title
  //=> "Author"
```
And we can also write:
```js
const {name: { first: given, last: surname}, occupation: { title: title } } = user;
surname
  //=> "Braithwaite"
title
  //=> "Author"
```
And of course, we destructure parameters:
```js
const description = ({name: { first: given }, occupation: { title: title } }) =>
  `${given} is a ${title}`;

description(user)
  //=> "Reginald is a Author"
```
Terrible grammar and capitalization, but let's move on. It is very common to write things like `title: title` when destructuring objects. When the label is a valid variable name, it's often the most obvious variable name as well. So JavaScript supports a further syntactic optimization:
```js
const description = ({name: { first }, occupation: { title } }) =>
  `${first} is a ${title}`;

description(user)
  //=> "Reginald is a Author"
```
And that same syntax works for literals:
```js
const abbrev = ({name: { first, last }, occupation: { title } }) => {
  return { first, last, title};
}

abbrev(user)
  //=> {"first":"Reginald","last":"Braithwaite","title":"Author"}
```
### revisiting linked lists

Earlier, we used two-element arrays as nodes in a linked list:
```js
const cons = (a, d) => [a, d],
      car  = ([a, d]) => a,
      cdr  = ([a, d]) => d;
```
In essence, this simple implementation used functions to create an abstraction with named elements. But now that we've looked at objects, we can use an object instead of a two-element array. While we're at it, let's use contemporary names. So our linked list nodes will be formed from `{ first, rest }`

In that case, a linked list of the numbers `1`, `2`, and `3` will look like this: `{ first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY } } }`.

We can then perform the equivalent of `[first, ...rest]` with direct property accessors:
```js
const EMPTY = {};
const OneTwoThree = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY } } };

OneTwoThree.first
  //=> 1

OneTwoThree.rest
  //=> {"first":2,"rest":{"first":3,"rest":{}}}

OneTwoThree.rest.rest.first
  //=> 3
```
Taking the length of a linked list is easy:
```js
const length = (node, delayed = 0) =>
  node === EMPTY
    ? delayed
    : length(node.rest, delayed + 1);

length(OneTwoThree)
  //=> 3
```
What about mapping? Well, let's start with the simplest possible thing, making a *copy* of a list. As we saw above, and discussed in [Garbage, Garbage Everywhere](#garbage-garbage-everywhere), it is fast to iterate forward through a linked list. What isn't fast is naïvely copying a list:
```js
const slowcopy = (node) =>
  node === EMPTY
    ? EMPTY
    : { first: node.first, rest: slowcopy(node.rest)};

slowcopy(OneTwoThree)
  //=> {"first":1,"rest":{"first":2,"rest":{"first":3,"rest":{}}}}
```
The problem here is that linked lists are constructed back-to-front, but we iterate over them front-to-back. So to copy a list, we have to save all the bits on the call stack and then construct the list from back-to-front as all the recursive calls return.

We could follow the strategy of delaying the work. Let's write that naively:
```js
const copy2 = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : copy2(node.rest, { first: node.first, rest: delayed });

copy2(OneTwoThree)
  //=> {"first":3,"rest":{"first":2,"rest":{"first":1,"rest":{}}}}
```
Well, well, well. We have unwittingly *reversed* the list. This makes sense, if lists are constructed from back to front, and we make a linked list out of items as we iterate through it, we're going to get a backwards copy of the list. This isn't a bad thing by any stretch of the imagination. Let's call it what it is:
```js
const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(node.rest, { first: node.first, rest: delayed });
```
And now, we can make a reversing map:
```js
const reverseMapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverseMapWith(fn, node.rest, { first: fn(node.first), rest: delayed });

reverseMapWith((x) => x * x, OneTwoThree)
  //=> {"first":9,"rest":{"first":4,"rest":{"first":1,"rest":{}}}}
```
And a regular `mapWith` follows:
```js
const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(node.rest, { first: node.first, rest: delayed });

const mapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? reverse(delayed)
    : mapWith(fn, node.rest, { first: fn(node.first), rest: delayed });

mapWith((x) => x * x, OneTwoThree)
  //=> {"first":1,"rest":{"first":4,"rest":{"first":9,"rest":{}}}}
```
Our `mapWith` function takes twice as long as a straight iteration, because it iterates over the entire list twice, once to map, and once to reverse the list. Likewise, it takes twice as much memory, because it constructs a reverse of the desired result before throwing it away.

Mind you, this is still much, much faster than making partial copies of arrays. For a list of length *n*, we created *n* superfluous nodes and copied *n* superfluous values. Whereas our naïve array algorithm created 2*n* superfluous arrays and copied *n*^2^ superfluous values.

## Mutation
In JavaScript, almost every type of value can *mutate*. Their identities stay the same, but not their structure. Specifically, arrays and objects can mutate. Recall that you can access a value from within an array or an object using `[]`. You can reassign a value using `[] =`:
```js
    const oneTwoThree = [1, 2, 3];
    oneTwoThree[0] = 'one';
    oneTwoThree
      //=> [ 'one', 2, 3 ]
```
You can even add a value:
```js
    const oneTwoThree = [1, 2, 3];
    oneTwoThree[3] = 'four';
    oneTwoThree
      //=> [ 1, 2, 3, 'four' ]
```
You can do the same thing with both syntaxes for accessing objects:
```js
    const name = {firstName: 'Leonard', lastName: 'Braithwaite'};
    name.middleName = 'Austin'
    name
      //=> { firstName: 'Leonard',
      #     lastName: 'Braithwaite',
      #     middleName: 'Austin' }
```
We have established that JavaScript's semantics allow for two different bindings to refer to the same value. For example:
```js
    const allHallowsEve = [2012, 10, 31]
    const halloween = allHallowsEve;  
```      
Both `halloween` and `allHallowsEve` are bound to the same array value within the local environment. And also:
```js
    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      // ...
    })(allHallowsEve);
```
There are two nested environments, and each one binds a name to the exact same array value. In each of these examples, we have created two *aliases* for the same value. Before we could reassign things, the most important point about this is that the identities were the same, because they were the same value.

This is vital. Consider what we already know about shadowing:
```js
    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween = [2013, 10, 31];
    })(allHallowsEve);
    allHallowsEve
      //=> [2012, 10, 31]
```      
The outer value of `allHallowsEve` was not changed because all we did was rebind the name `halloween` within the inner environment. However, what happens if we *mutate* the value in the inner environment?
```js
    const allHallowsEve = [2012, 10, 31];
    (function (halloween) {
      halloween[0] = 2013;
    })(allHallowsEve);
    allHallowsEve
      //=> [2013, 10, 31]
```      
This is different. We haven't rebound the inner name to a different variable, we've mutated the value that both bindings share. Now that we've finished with mutation and aliases, let's have a look at it.
      
> JavaScript permits the reassignment of new values to existing bindings, as well as the reassignment and assignment of new values to elements of containers such as arrays and objects. Mutating existing objects has special implications when two bindings are aliases of the same value.

> Note well: Declaring a variable `const` does not prevent us from mutating its value, only from rebinding its name. This is an important distinction.

### mutation and data structures

Mutation is a surprisingly complex subject. It is possible to compute anything without ever mutating an existing entity. Languages like [Haskell] don't permit mutation at all. In general, mutation makes some algorithms shorter to write and possibly faster, but harder to reason about.

[Haskell]: https://en.wikipedia.org/wiki/Haskell_(programming_language)

One pattern many people follow is to be liberal with mutation when constructing data, but conservative with mutation when consuming data. Let's recall linked lists from [Plain Old JavaScript Objects](#plain-old-javascript-objects). While we're executing the `mapWith` function, we're constructing a new linked list. By this pattern, we would be happy to use mutation to construct the list while running `mapWith`.

But after returning the new list, we then become conservative about mutation. This also makes sense: Linked lists often use structure sharing. For example:
```js
const EMPTY = {};
const OneToFive = { first: 1, 
                    rest: {
                      first: 2,
                      rest: {
                        first: 3,
                        rest: {
                          first: 4,
                          rest: {
                            first: 5,
                            rest: EMPTY } } } } };

OneToFive
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":"four","rest":{"first":"five","rest":{}}}}}}
                            
const ThreeToFive = OneToFive.rest.rest;

ThreeToFive
  //=> {"first":3,"rest":{"first":4,"rest":{"first":5,"rest":{}}}}
  
ThreeToFive.first = "three";
ThreeToFive.rest.first = "four";
ThreeToFive.rest.rest.first = "five";

ThreeToFive
  //=> {"first":"three","rest":{"first":"four","rest":{"first":"five","rest":{}}}}

OneToFive
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":"four","rest":{"first":"five","rest":{}}}}}}
```
Changes made to `ThreeToFive` affect `OneToFive`, because they share the same structure. When we wrote `ThreeToFive = OneToFive.rest.rest;`, we weren't making a brand new copy of `{"first":3,"rest":{"first":4,"rest":{"first":5,"rest":{}}}}`, we were getting a reference to the same chain of nodes.

Structure sharing like this is what makes linked lists so fast for taking everything but the first item of a list: We aren't making a new list, we're using some of the old list. Whereas destructuring an array with `[first, ...rest]` does make a copy, so:
```js
const OneToFive = [1, 2, 3, 4, 5];

OneToFive
  //=> [1,2,3,4,5]

const [a, b, ...ThreeToFive] = OneToFive;

ThreeToFive
  //=> [3, 4, 5]

ThreeToFive[0] = "three";
ThreeToFive[1] = "four";
ThreeToFive[2] = "five";

ThreeToFive
  //=> ["three","four","five"]
  
OneToFive
  //=> [1,2,3,4,5]
```
The gathering operation `[a, b, ...ThreeToFive]` is slower, but "safer."

So back to avoiding mutation. In general, it's easier to reason about data that doesn't change. We don't have to remember to use copying operations when we pass it as a value to a function, or extract some data from it. We just use the data, and the less we mutate it, the fewer the times we have to think about whether making changes will be "safe."

### building with mutation

As noted, one pattern is to be more liberal about mutation when building a data structure. Consider our `copy` algorithm. Without mutation, a copy of a linked list can be made in constant space by reversing a reverse of the list:
```js
const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(node.rest, { first: node.first, rest: delayed });

const copy = (node) => reverse(reverse(node));
```
If we want to make a copy of a linked list without iterating over it twice and making a copy we discard later, we can use mutation:
```js
const copy = (node, head = null, tail = null) => {
  if (node === EMPTY) {
    return head;
  }
  else if (tail === null) {
    const { first, rest } = node;
    const newNode = { first, rest };
    return copy(rest, newNode, newNode);
  }
  else {
    const { first, rest } = node;
    const newNode = { first, rest };
    tail.rest = newNode;
    return copy(node.rest, head, newNode);
  }
}
```
This algorithm makes copies of nodes as it goes, and mutates the last node in the list so that it can splice the next one on. Adding a node to an existing list is risky, as we saw when considering the fact that `OneToFive` and `ThreeToFive` share the same nodes. But when we're in the midst of creating a brand new list, we aren't sharing any nodes with any other lists, and we can afford to be more liberal about using mutation to save space and/or time.

Armed with this basic copy implementation, we can write `mapWith`:
```js
const mapWith = (fn, node, head = null, tail = null) => {
  if (node === EMPTY) {
    return head;
  }
  else if (tail === null) {
    const { first, rest } = node;
    const newNode = { first: fn(first), rest };
    return mapWith(fn, rest, newNode, newNode);
  }
  else {
    const { first, rest } = node;
    const newNode = { first: fn(first), rest };
    tail.rest = newNode;
    return mapWith(fn, node.rest, head, newNode);
  }
}

mapWith((x) => 1.0 / x, OneToFive)
  //=> {"first":1,"rest":{"first":0.5
  //    ,"rest":{"first":0.3333333333333333,"rest":{"first":0.25,"rest":{"first":0.2,"rest":{}}}}}}
```
## Reassignment

Like some imperative programming languages, JavaScript allows you to re-assign the value bound to parameters. We saw this earlier in [rebinding](main_0_functions.md#rebinding):

By default, JavaScript permits us to *rebind* new values to names bound with a parameter. For example, we can write:
```js
    const evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        n = n - 2;
        return evenStevens(n);
      }
    }

    evenStevens(42)
      //=> true
```
The line `n = n - 2;` *rebinds* a new value to the name `n`. We will discuss this at much greater length in [Reassignment](#reassignment), but long before we do, let's try a similar thing with a name bound using `const`. We've already bound `evenStevens` using `const`, let's try rebinding it:
```js
    evenStevens = (n) => {
      if (n === 0) {
        return true;
      }
      else if (n == 1) {
        return false;
      }
      else {
        return evenStevens(n - 2);
      }
    }
      //=> ERROR, evenStevens is read-only
```
JavaScript does not permit us to rebind a name that has been bound with `const`. We can *shadow* it by using `const` to declare a new binding with a new function or block scope, but we cannot rebind a name that was bound with `const` in an existing scope.

Rebinding parameters is usually avoided, but what about rebinding names we declare within a function? What we want is a statement that works like `const`, but permits us to rebind variables. JavaScript has such a thing, it's called `let`:
```js
let age = 52;

age = 53;
age
  //=> 53
```
We took the time to carefully examine what happens with bindings in environments. Let's take the time to explore what happens with reassigning values to variables. The key is to understand that we are rebinding a different value to the same name in the same environment.

So let's consider what happens with a shadowed variable:
```js
    (() => {
      let age = 49;

      if (true) {
        let age = 50;
      }
      return age;
    })()
      //=> 49
```
Using `let` to bind `50` to age within the block does not change the binding of `age` in the outer environment because the binding of `age` in the block shadows the binding of `age` in the outer environment, just like `const`. We go from:
```js
    {age: 49, '..': global-environment}
```
To:
```js
    {age: 50, '..': {age: 49, '..': global-environment}}
```
Then back to:
```js
    {age: 49, '..': global-environment}
```
However, if we don't shadow `age` with `let`, reassigning within the block changes the original:
```js
    (() => {
      let age = 49;

      if (true) {
        age = 50;
      }
      return age;
    })()
      //=> 50
```
Like evaluating variable labels, when a binding is rebound, JavaScript searches for the binding in the current environment and then each ancestor in turn until it finds one. It then rebinds the name in that environment.

### mixing `let` and `const`

Some programmers dislike deliberately shadowing variables. The suggestion is that shadowing a variable is confusing code. If you buy that argument, the way that shadowing works in JavaScript exists to protect us from accidentally shadowing a variable when we move code around.

If you dislike deliberately shadowing variables, you'll probably take an even more opprobrious view of mixing `const` and `let` semantics with a shadowed variable:
```js
    (() => {
      let age = 49;

      if (true) {
        const age = 50;
      }
      age = 51;
      return age;
    })()
      //=> 51
```
Shadowing a `let` with a `const` does not change our ability to rebind the variable in its original scope. And:
```js
    (() => {
      const age = 49;

      if (true) {
        let age = 50;
      }
      age = 52;
      return age;
    })()
      //=> ERROR: age is read-only
```
Shadowing a `const` with a `let` does not permit it to be rebound in its original scope.

### `var`

JavaScript has one *more* way to bind a name to a value, `var`.`1`

>`1` How many have we seen so far? Well, parameters bind names. Function declarations bind names. Named function expressions bind names. `const` and `let` bind names. So that's five different ways so far. And there are more!

`var` looks a lot like `let`:
```js
const factorial = (n) => {
  let x = n;
  if (x === 1) {
    return 1;
  }
  else {
    --x;
    return n * factorial(x);
  }
}

factorial(5)
  //=> 120

const factorial2 = (n) => {
  var x = n;
  if (x === 1) {
    return 1;
  }
  else {
    --x;
    return n * factorial2(x);
  }
}

factorial2(5)
  //=> 120
```
But of course, it's not exactly like `let`. It's just different enough to present a source of confusion. First, `var` is not block scoped, it's function scoped, just like [function declarations]():
```js
(() => {
  var age = 49;

  if (true) {
    var age = 50;
  }
  return age;
})()
  //=> 50
```
Declaring `age` twice does not cause an error(!), and the inner declaration does not shadow the outer declaration. All `var` declarations behave as if they were hoisted to the top of the function, a little like function declarations.

But, again, it is unwise to expect consistency. A function declaration can appear anywhere within a function, but the declaration *and* the definition are hoisted. Note this example of a function that uses a helper:
```js
const factorial = (n) => {
  return innerFactorial(n, 1);

  function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> 24
```
JavaScript interprets this code as if we had written:
```js
const factorial = (n) => {
  let innerFactorial = function innerFactorial (x, y) {
      if (x == 1) {
        return y;
      }
      else {
        return innerFactorial(x-1, x * y);
      }
    }

  return innerFactorial(n, 1);
}
```
JavaScript hoists the `let` and the assignment. But not so with `var`:
```js
const factorial = (n) => {

  return innerFactorial(n, 1);

  var innerFactorial = function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> undefined is not a function (evaluating 'innerFactorial(n, 1)')
```
JavaScript hoists the declaration, but not the assignment. It is as if we'd written:
```js
const factorial = (n) => {

  let innerFactorial = undefined;

  return innerFactorial(n, 1);

  innerFactorial = function innerFactorial (x, y) {
    if (x == 1) {
      return y;
    }
    else {
      return innerFactorial(x-1, x * y);
    }
  }
}

factorial(4)
  //=> undefined is not a function (evaluating 'innerFactorial(n, 1)')
```
In that way, `var` is a little like `const` and `let`, we should always declare and bind names before using them. But it's not like `const` and `let` in that it's function scoped, not block scoped.

### why `const` and `let` were invented

`const` and `let` are recent additions to JavaScript. For nearly twenty years, variables were declared with `var` (not counting parameters and function declarations, of course). However, its functional scope was a problem.

We haven't looked at it yet, but JavaScript provides a `for` loop for your iterating pleasure and convenience. It looks a lot like the `for` loop in C. Here it is with `var`:
```js
    var sum = 0;
    for (var i = 1; i <= 100; i++) {
      sum = sum + i
    }
    sum
      #=> 5050
```
Hopefully, you can think of a faster way to calculate this sum.`1` And perhaps you have noticed that `var i = 1` is tucked away instead of being at the top as we prefer. But is this ever a problem?

>`1` There is a well known story about Karl Friedrich Gauss when he was in elementary school. His teacher got mad at the class and told them to add the numbers 1 to 100 and give him the answer by the end of the class. About 30 seconds later Gauss gave him the answer. The other kids were adding the numbers like this: `1 + 2 + 3 + . . . . + 99 + 100 = ?` But Gauss rearranged the numbers to add them like this: `(1 + 100) + (2 + 99) + (3 + 98) + . . . . + (50 + 51) = ?` If you notice every pair of numbers adds up to 101. There are 50 pairs of numbers, so the answer is 50*101 = 5050. Of course Gauss came up with the answer about 20 times faster than the other kids.

Yes. Consider this variation:
```js
    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      introductions[i] = "Hello, my name is " + names[i]
    }
    introductions
      //=> [ 'Hello, my name is Karl',
      //     'Hello, my name is Friedrich',
      //     'Hello, my name is Gauss' ]
```
So far, so good. Hey, remember that functions in JavaScript are values? Let's get fancy!
```js
    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      introductions[i] = (soAndSo) =>
        `Hello, ${soAndSo}, my name is ${names[i]}`
    }
    introductions
      //=> [ [Function],
      //     [Function],
      //     [Function] ]
```
Again, so far, so good. Let's try one of our functions:
```js
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is undefined'
```
What went wrong? Why didn't it give us 'Hello, Raganwald, my name is Friedrich'? The answer is that pesky `var i`. Remember that `i` is bound in the surrounding environment, so it's as if we wrote:
```js
    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'],
        i = undefined;

    for (i = 0; i < 3; i++) {
      introductions[i] = function (soAndSo) {
        return "Hello, " + soAndSo + ", my name is " + names[i]
      }
    }
    introductions
```
Now, at the time we created each function, `i` had a sensible value, like `0`, `1`, or `2`. But at the time we *call* one of the functions, `i` has the value `3`, which is why the loop terminated. So when the function is called, JavaScript looks `i` up in its enclosing environment (its  closure, obviously), and gets the value `3`. That's not what we want at all.

The error wouldn't exist at all if we'd used `let` in the first place
```js
    let introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (let i = 0; i < 3; i++) {
      introductions[i] = (soAndSo) =>
        `Hello, ${soAndSo}, my name is ${names[i]}`
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'
```
This small error was a frequent cause of confusion, and in the days when there was no block-scoped `let`, programmers would need to know how to fake it, usually with an IIFE:
```js
    var introductions = [],
        names = ['Karl', 'Friedrich', 'Gauss'];

    for (var i = 0; i < 3; i++) {
      ((i) => {
        introductions[i] = (soAndSo) =>
          `Hello, ${soAndSo}, my name is ${names[i]}`
        }
      })(i)
    }
    introductions[1]('Raganwald')
      //=> 'Hello, Raganwald, my name is Friedrich'
```
Now we're creating a new inner parameter, `i` and binding it to the value of the outer `i`. This works, but `let` is so much simpler and cleaner that it was added to the language in the ECMAScript 2015 specification.

In this book, we will use function declarations sparingly, and not use `var` at all. That does not mean that you should follow the exact same practice in your own code: The purpose of this book is to illustrate certain principles of programming. The purpose of your own code is to get things done. The two goals are often, but not always, aligned.

## Copy on Write
We've seen how to build lists with arrays and with linked lists. We've touched on an important difference between them:

* When you take the rest of an array with destructuring (`[first, ...rest]`), you are given a *copy* of the elements of the array.
* When you take the rest of a linked list with its reference, you are given the exact same nodes of the elements of the original list.

The consequence of this is that if you have an array, and you take it's "rest," your "child" array is a copy of the elements of the parent array. And therefore, modifications to the parent do not affect the child, and modifications to the child do not affect the parent.

Whereas if you have a linked list, and you take it's "rest," your "child" list shares its nodes with the "parent" list. And therefore, modifications to the parent also modify the child, and modifications to the child also modify the parent.

Let's confirm our understanding:
```js
const parentArray = [1, 2, 3];
const [aFirst, ...childArray] = parentArray;

parentArray[2] = "three";
childArray[0] = "two";

parentArray
  //=> [1,2,"three"]
childArray
  //=> ["two",3]

const EMPTY = { first: {}, rest: {} };
const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = parentList.rest;

parentList.rest.rest.first = "three";
childList.first = "two";

parentList
  //=> {"first":1,"rest":{"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}
```
This is remarkably unsafe. If we *know* that a list doesn't share any elements with another list, we can safely modify it. But how do we keep track of that? Add a bunch of bookkeeping to track references? We'll end up reinventing reference counting and garbage collection.

### a few utilities

before we go any further, let's write a few naïve list utilities so that we can work at a slightly higher level of abstraction:
```js
const copy = (node, head = null, tail = null) => {
  if (node === EMPTY) {
    return head;
  }
  else if (tail === null) {
    const { first, rest } = node;
    const newNode = { first, rest };
    return copy(rest, newNode, newNode);
  }
  else {
    const { first, rest } = node;
    const newNode = { first, rest };
    tail.rest = newNode;
    return copy(node.rest, head, newNode);
  }
}

const first = ({first, rest}) => first;
const rest = ({first, rest}) => rest;

const reverse = (node, delayed = EMPTY) =>
  node === EMPTY
    ? delayed
    : reverse(rest(node), { first: first(node), rest: delayed });

const mapWith = (fn, node, delayed = EMPTY) =>
  node === EMPTY
    ? reverse(delayed)
    : mapWith(fn, rest(node), { first: fn(first(node)), rest: delayed });

const at = (index, list) =>
  index === 0
    ? first(list)
    : at(index - 1, rest(list));
    
const set = (index, value, list, originalList = list) =>
  index === 0
    ? (list.first = value, originalList)
    : set(index - 1, value, rest(list), originalList)
    
const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

set(2, "three", parentList);
set(0, "two", childList);

parentList
  //=> {"first":1,"rest":{"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":"three","rest":{"first":{},"rest":{}}}}
```
Our new `at` and `set` functions behave similarly to `array[index]` and `array[index] = value`. The main difference is that `array[index] = value` evaluates to `value`, while `set(index, value, list)` evaluates to the modified `list`.
  
### copy-on-read

So back to the problem of structure sharing. One strategy for avoiding problems is to be *pessimistic*. Whenever we take the rest of a list, make a copy.
```js
const rest = ({first, rest}) => copy(rest);

const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

const newParentList = set(2, "three", parentList);
set(0, "two", childList);

parentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":"two","rest":{"first":3,"rest":{"first":{},"rest":{}}}}
```
This strategy is called "copy-on-read", because when we attempt the parent  to "read" the value of a child of the list, we make a copy and read the copy of the child. Thereafter, we can write to the parent or the copy of the child freely.

As we expected, making a copy lets us modify the copy without interfering with the original. This is, however, expensive. Sometimes we don't need to make a copy because we won't be modifying the list. Our `mapWith` function would be very expensive if we make a copy every time we call `rest(node)`.

There's also a bug: What happens when we modify the first element of a list? But before we fix that, let's try being lazy about copying.

### copy-on-write

Why are we copying? In case we modify a child list. Ok, what if we do this: Make the copy when we know we are modifying the list. When do we know that? When we call `set`. We'll restore our original definition for `rest`, but change `set`:
```js
const rest = ({first, rest}) => rest;

const set = (index, value, list) =>
  index === 0
    ? { first: value, rest: list.rest }
    : { first: list.first, rest: set(index - 1, value, list.rest) };

const parentList = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY }}};
const childList = rest(parentList);

const newParentList = set(2, "three", parentList);
const newChildList = set(0, "two", childList);
```
Our original parent and child lists remain unmodified:
```js
parentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":3,"rest":{"first":{},"rest":{}}}}}
childList
  //=> {"first":2,"rest":{"first":3,"rest":{"first":{},"rest":{}}}}
```
But our new parent and child lists are copies that contain the desired modifications, without interfering with each other:
```js
newParentList
  //=> {"first":1,"rest":{"first":2,"rest":{"first":"three","rest":{"first":{},"rest":{}}}}}
newChildList
  //=> {"first":"two","rest":{"first":3,"rest":{"first":{},"rest":{}}}}
```
And now functions like `mapWith` that make copies without modifying anything, work at full speed.

This strategy of waiting to copy until you are writing is called copy-on-write, or "COW:"

> Copy-on-write is the name given to the policy that whenever a task attempts to make a change to the shared information, it should first create a separate (private) copy of that information to prevent its changes from becoming visible to all the other tasks.—[Wikipedia][Copy-on-write]

[Copy-on-write]: https://en.wikipedia.org/wiki/Copy-on-write

Like all strategies, it makes a tradeoff: It's much cheaper than pessimistically copying structures when you make an infrequent number of small changes, but if you tend to make a lot of changes to some that you aren't sharing, it's more expensive.

Looking at the code again, you see that the `copy` function doesn't copy on write: It follows the pattern that while constructing something, we own it and can be liberal with mutation. Once we're done with it and give it to someone else, we need to be conservative and use a strategy like copy-on-read or copy-on-write.
## Tortoises, Hares, and Teleporting Turtles

A good long while ago (The First Age of Internet Startups), someone asked me one of those pet algorithm questions. It was, "Write an algorithm to detect a loop in a linked list, in constant space."

I'm not particularly surprised that I couldn't think up an answer in a few minutes at the time. And to the interviewer's credit, he didn't terminate the interview on the spot, he asked me to describe the kinds of things going through my head.

I think I told him that I was trying to figure out if I could adapt a hashing algorithm such as XORing everything together. This is the "trick answer" to a question about finding a missing integer from a list, so I was trying the old, "Transform this into [a problem you've already solved](http://www-users.cs.york.ac.uk/susan/joke/3.htm#boil)" meta-algorithm. We moved on from there, and he didn't reveal the "solution."

I went home and pondered the problem. I wanted to solve it. Eventually, I came up with something and tried it (In Java!) on my home PC. I sent him an email sharing my result, to demonstrate my ability to follow through. I then forgot about it for a while. Some time later, I was told that the correct solution was:
```js
const EMPTY = null;
const isEmpty = (node) => node === EMPTY;
const pair = (first, rest = EMPTY) => ({first, rest});
const list = (...elements) => {
  const [first, ...rest] = elements;
  
  return elements.length === 0
    ? EMPTY
    : pair(first, list(...rest))
}

const forceAppend = (list1, list2) => {
  if (isEmpty(list1)) {
    return "FAIL!"
  }
  if (isEmpty(list1.rest)) {
    list1.rest = list2;
  }
  else {
    forceAppend(list1.rest, list2);
  }
}

const tortoiseAndHare = (aPair) => {
  let tortoisePair = aPair,
      harePair = aPair.rest;
  
  while (true) {
    if (isEmpty(tortoisePair) || isEmpty(harePair)) {
      return false;
    }
    if (tortoisePair.first === harePair.first) {
      return true;
    }
    
    harePair = harePair.rest;
    
    if (isEmpty(harePair)) {
      return false;
    }
    if (tortoisePair.first === harePair.first) {
      return true;
    }
    
    tortoisePair = tortoisePair.rest;
    harePair = harePair.rest;
  }
};

const aList = list(1, 2, 3, 4, 5);

tortoiseAndHare(aList)
  //=> false

forceAppend(aList, aList.rest.rest);

tortoiseAndHare(aList);
  //=> true
```
This algorithm is called "The Tortoise and the Hare," and was discovered by Robert Floyd in the 1960s. You have two node references, and one traverses the list at twice the speed of the other. No matter how large it is, you will eventually have the fast reference equal to the slow reference, and thus you'll detect the loop.

At the time, I couldn't think of any way to use hashing to solve the problem, so I gave up and tried to fit this into a powers-of-two algorithm. My first pass at it was clumsy, but it was roughly equivalent to this:
```js
const teleportingTurtle = (list) => {
  let speed = 1,
      rabbit = list,
      turtle = rabbit;
  
  while (true) {
    for (let i = 0; i <= speed; i += 1) {
      rabbit = rabbit.rest;
      if (rabbit == null) {
        return false;
      }
      if (rabbit === turtle) {
        return true;
      }
    }
    turtle = rabbit;
    speed *= 2;
  }
  return false;
};

const aList = list(1, 2, 3, 4, 5);

teleportingTurtle(aList)
  //=> false

forceAppend(aList, aList.rest.rest);

teleportingTurtle(aList);
  //=> true
```
Years later, I came across a discussion of this algorithm, [The Tale of the Teleporting Turtle](http://www.penzba.co.uk/Writings/TheTeleportingTurtle.html). It seems to be faster under certain circumstances, depending on the size of the loop and the relative costs of certain operations.

What's interesting about these two algorithms is that they both *tangle* two separate concerns: How to traverse a data structure, and what to do with the elements that you encounter. In [Functional Iterators](#functional-iterators), we'll investigate one pattern for separating these concerns.

## Functional Iterators

Let's consider a remarkably simple problem: Finding the sum of the elements of an array. In tail-recursive style, it looks like this:
```js
const arraySum = ([first, ...rest], accumulator = 0) =>
  first === undefined
    ? accumulator
    : arraySum(rest, first + accumulator)

arraySum([1, 4, 9, 16, 25])
  //=> 55
```
As we saw earlier, this entangles the mechanism of traversing the array with the business of summing the bits. So we can separate them using `fold`:
```js
const callLeft = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...args, ...remainingArgs);

const foldArrayWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : fn(first, foldArrayWith(fn, terminalValue, rest));

const arraySum = callLeft(foldArrayWith, (a, b) => a + b, 0);

arraySum([1, 4, 9, 16, 25])
  //=> 55
```
The nice thing about this is that the definition for `arraySum` mostly concerns itself with summing, and not with traversing over a collection of data. But it still relies on `foldArrayWith`, so it can only sum arrays.

What happens when we want to sum a tree of numbers? Or a linked list of numbers?

Well, we call `arraySum` with an array, and it has baked into it a method for traversing the array. Perhaps we could extract both of those things. Let's rearrange our code a bit:
```js
const callRight = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const foldArrayWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : fn(first, foldArrayWith(fn, terminalValue, rest));

const foldArray = (array) => callRight(foldArrayWith, array);

const sumFoldable = (folder) => folder((a, b) => a + b, 0);

sumFoldable(foldArray([1, 4, 9, 16, 25]))
  //=> 55
```
What we've done is turn an array into a function that folds an array with    
`const foldArray = (array) => callRight(foldArrayWith, array);`.     
The `sumFoldable` function doesn't care what kind of data structure we have, as long as it's foldable.

Here it is summing a tree of numbers:
```js
const callRight = (fn, ...args) =>
    (...remainingArgs) =>
      fn(...remainingArgs, ...args);

const foldTreeWith = (fn, terminalValue, [first, ...rest]) =>
  first === undefined
    ? terminalValue
    : Array.isArray(first)
      ? fn(foldTreeWith(fn, terminalValue, first), foldTreeWith(fn, terminalValue, rest))
      : fn(first, foldTreeWith(fn, terminalValue, rest));

const foldTree = (tree) => callRight(foldTreeWith, tree);
const sumFoldable = (folder) => folder((a, b) => a + b, 0);

sumFoldable(foldTree([1, [4, [9, 16]], 25]))
  //=> 55
```
We've found another way to express *the principle of separating* **traversing a data structure** from **the operation we want to perform on that data structure,** [we've completely separated **the knowledge of how to sum** from **the knowledge of how to fold an array or tree** (or anything else, really).]()

### iterating

Folding is a universal operation, and with care we can accomplish any task with folds that could be accomplished with that stalwart of structured programming, the `for` loop. Nevertheless, there is some value in being able to express some algorithms as iteration.

JavaScript has a particularly low-level version of `for` loop that mimics the semantics of the `C` language. Summing the elements of an array can be accomplished with:
```js
const arraySum = (array) => {
  let sum = 0;

  for (let i = 0; i < array.length; ++i) {
    sum += array[i];
  }
  return sum
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
```
Once again, we're mixing the code for iterating over an array with the code for calculating a sum. And worst of all, we're getting really low-level with details like knowing that the elements of an array are indexed with consecutive integers that begin with `0`.

We can write this a slightly different way, using a `while` loop:
```js
const arraySum = (array) => {
  let done,
      sum = 0,
      i = 0;

  while ((done = i == array.length, !done)) {
    const value = array[i++];
    sum += value;
  }
  return sum
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
```
Notice that buried inside our loop, we have bound the names `done` and `value`. We can put those into a POJO (a [Plain Old JavaScript Object](#plain-old-javascript-objects)). It'll be a little awkward, but we'll be patient:
```js
const arraySum = (array) => {
  let iter,
      sum = 0,
      index = 0;

  while (
    (eachIteration = {
        done: index === array.length,
        value: index < array.length ? array[index] : undefined
      },
      ++index,
      !eachIteration.done)
    ) {
    sum += eachIteration.value;
  }
  return sum;
}

arraySum([1, 4, 9, 16, 25])
  //=> 55
```
With this code, we make a POJO that has `done` and `value` keys. All the summing code needs to know is to add `eachIteration.value`. Now we can extract the ickiness into a separate function:
```js
const arrayIterator = (array) => {
  let i = 0;

  return () => {
    const done = i === array.length;

    return {
      done,
      value: done ? undefined : array[i++]
    }
  }
}

const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;

  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum;
}

iteratorSum(arrayIterator([1, 4, 9, 16, 25]))
  //=> 55
```
Now this is something else. The `arrayIterator` function takes an array and returns a function we can call repeatedly to obtain the elements of the array. The `iteratorSum` function iterates over the elements by calling the `iterator` function repeatedly until it returns `{ done: true }`.

We can write a different iterator for a different data structure. Here's one for linked lists:
```js
const EMPTY = null;

const isEmpty = (node) => node === EMPTY;

const pair = (first, rest = EMPTY) => ({first, rest});

const list = (...elements) => {
  const [first, ...rest] = elements;

  return elements.length === 0
    ? EMPTY
    : pair(first, list(...rest))
}

const print = (aPair) =>
  isEmpty(aPair)
    ? ""
    : `${aPair.first} ${print(aPair.rest)}`

const listIterator = (aPair) =>
  () => {
    const done = isEmpty(aPair);
    if (done) {
      return {done};
    }
    else {
      const {first, rest} = aPair;

      aPair = aPair.rest;
      return { done, value: first }
    }
  }

const iteratorSum = (iterator) => {
  let eachIteration,
      sum = 0;;

  while ((eachIteration = iterator(), !eachIteration.done)) {
    sum += eachIteration.value;
  }
  return sum
}

const aListIterator = listIterator(list(1, 4, 9, 16, 25));

iteratorSum(aListIterator)
  //=> 55
```
### unfolding and laziness

Iterators are functions. When they iterate over an array or linked list, they are traversing something that is already there. But they could just as easily manufacture the data as they go. Let's consider the simplest example:
```js
const NumberIterator = (number = 0) =>
  () => ({ done: false, value: number++ })

fromOne = NumberIterator(1);

fromOne().value;
  //=> 1
fromOne().value;
  //=> 2
fromOne().value;
  //=> 3
fromOne().value;
  //=> 4
fromOne().value;
  //=> 5
```
And here's another one:
```js
const FibonacciIterator  = () => {
  let previous = 0,
      current = 1;

  return () => {
    const value = current;

    [previous, current] = [current, current + previous];
    return {done: false, value};
  };
};

const fib = FibonacciIterator()

fib().value
  //=> 1
fib().value
  //=> 1
fib().value
  //=> 2
fib().value
  //=> 3
fib().value
  //=> 5
```
**A function that starts with a seed and expands it into a data structure is called an *unfold*. It's the opposite of a fold.** It's possible to write a generic unfold mechanism, but let's pass on to what we can do with unfolded iterators.

For starters, we can `map` an iterator, just like we map a collection:
```js
const mapIteratorWith = (fn, iterator) =>
  () => {
    const {done, value} = iterator();

    return ({done, value: done ? undefined : fn(value)});
  }

const squares = mapIteratorWith((x) => x * x, NumberIterator(1));

squares().value
  //=> 1
squares().value
  //=> 4
squares().value
  //=> 9
```
This business of going on forever has some drawbacks. Let's introduce an idea: A function that takes an iterator and returns another iterator. We can start with `take`, an easy function that returns an iterator that only returns a fixed number of elements:
```js
const take = (iterator, numberToTake) => {
  let count = 0;

  return () => {
    if (++count <= numberToTake) {
      return iterator();
    } else {
      return {done: true};
    }
  };
};

const toArray = (iterator) => {
  let eachIteration,
      array = [];

  while ((eachIteration = iterator(), !eachIteration.done)) {
    array.push(eachIteration.value);
  }
  return array;
}

toArray(take(FibonacciIterator(), 5))
  //=> [1, 1, 2, 3, 5]

toArray(take(squares, 5))
  //=> [1, 4, 9, 16, 25]
```
How about the squares of the first five odd numbers? We'll need an iterator that produces odd numbers. We can write that directly:
```js
const odds = () => {
  let number = 1;

  return () => {
    const value = number;

    number += 2;
    return {done: false, value};
  }
}

const squareOf = callLeft(mapIteratorWith, (x) => x * x)

toArray(take(squareOf(odds()), 5))
  //=> [1, 9, 25, 49, 81]
```
We could also write a filter for iterators to accompany our mapping function:
```js
const filterIteratorWith = (fn, iterator) =>
  () => {
    do {
      const {done, value} = iterator();
    } while (!done && !fn(value));
    return {done, value};
  }

const oddsOf = callLeft(filterIteratorWith, (n) => n % 2 === 1);

toArray(take(squareOf(oddsOf(NumberIterator(1))), 5))
  //=> [1, 9, 25, 49, 81]
```
Mapping and filtering iterators allows us to compose the parts we already have, rather than writing a tricky bit of code with ifs and whiles and boundary conditions.

### bonus

Many programmers coming to JavaScript from other languages are familiar with three "canonical" operations on collections: folding, filtering, and finding. In Smalltalk, for example, they are known as `collect`, `select`, and `detect`.

We haven't written anything that finds the first element of an iteration that meets a certain criteria. Or have we?
```js
const firstInIteration = (fn, iterator) =>
  take(filterIteratorWith(fn, iterator), 1);
```
This is interesting, because it is lazy: It doesn't apply `fn` to every element in an iteration, just enough to find the first that passes the test. Whereas if we wrote something like:
```js
const firstInArray = (fn, array) =>
  array.filter(fn)[0];
```
JavaScript would apply `fn` to every element. If `array` was very large, and `fn` very slow, this would consume a lot of unnecessary time. And if `fn` had some sort of side-effect, the program could be buggy.

### caveat

Please note that unlike most of the other functions discussed in this book, iterators are *stateful*. There are some important implications of stateful functions. One is that while functions like `take(...)` appear to create an entirely new iterator, in reality they return a decorated reference to the original iterator. So as you traverse the new decorator, you're changing the state of the original!

For all intents and purposes, once you pass an iterator to a function, you can expect that you no longer "own" that iterator, and that its state either has changed or will change.

## Making Data Out Of Functions

In our code so far, we have used arrays and objects to represent the structure of data, and we have extensively used the ternary operator to write algorithms that terminate when we reach a base case.

For example, this `length` function uses a functions to bind values to names, POJOs to structure nodes, and the ternary function to detect the base case, the empty list.
```js
const EMPTY = {};
const OneTwoThree = { first: 1, rest: { first: 2, rest: { first: 3, rest: EMPTY } } };

OneTwoThree.first
  //=> 1
  
OneTwoThree.rest.first
  //=> 2
  
OneTwoThree.rest.rest.first
  //=> 3
  
const length = (node, delayed = 0) =>
  node === EMPTY
    ? delayed
    : length(node.rest, delayed + 1);

length(OneTwoThree)
  //=> 3
```
A very long time ago, mathematicians like Alonzo Church, Moses Schönfinkel, Alan Turning, and Haskell Curry and asked themselves if we really needed all these features to perform computations. They searched for a radically simpler set of tools that could accomplish all of the same things.

They established that arbitrary computations could be represented a small set of axiomatic components. For example, we don't need arrays to represent lists, or even POJOs to represent nodes in a linked list. We can model lists just using functions.

> [To Mock a Mockingbird](http://www.amazon.com/gp/product/0192801422/ref=as_li_ss_tl?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0192801422) established the metaphor of songbirds for the combinators, and ever since then logicians have called the K combinator a "kestrel," the B combinator a "bluebird," and so forth. 

> The [oscin.es] library contains code for all of the standard combinators and for experimenting using the standard notation.

[To Mock a Mockingbird]: http://www.amazon.com/gp/product/0192801422/ref=as_li_ss_tl?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0192801422
[oscin.es]: http://oscin.es

Let's start with some of the building blocks of combinatory logic, the K, I, and V combinators, nicknamed the "Kestrel", the "Idiot Bird", and the "Vireo:"
```js
const K = (x) => (y) => x;
const I = (x) => (x);
const V = (x) => (y) => (z) => z(x)(y);
```
### the kestrel and the idiot

A *constant function* is a function that always returns the same thing, no matter what you give it. For example, `(x) => 42` is a constant function that always evaluates to 42. The kestrel, or `K`, is a function that makes constant functions. You give it a value, and it returns a constant function that gives that value.

For example:
```js
const K = (x) => (y) => x;

const fortyTwo = K(42);

fortyTwo(6)
  //=> 42

fortyTwo("Hello")
  //=> 42
```
The *identity function* is a function that evaluates to whatever parameter you pass it. So `I(42) => 42`. Very simple, but useful. Now we'll take it one more step forward: Passing a value to `K` gets a function back, and passing a value to that function gets us a value.

Like so:
```js
K(6)(7)
  //=> 6
  
K(12)(24)
  //=> 12
```
This is very interesting. Given two values, we can say that `K` always returns the *first* value: `K(x)(y) => x` (that's not valid JavaScript, but it's essentially how it works).

Now, an interesting thing happens when we pass functions to each other. Consider `K(I)`. From what we just wrote, `K(x)(y) => x` So `K(I)(x) => I`. Makes sense. Now let's tack one more invocation on: What is `K(I)(x)(y)`? If `K(I)(x) => I`, then `K(I)(x)(y) === I(y)` which is `y`.

Therefore, `K(I)(x)(y) => y`:
```js
K(I)(6)(7)
  //=> 7
  
K(I)(12)(24)
  //=> 24
```
Aha! Given two values, `K(I)` always returns the *second* value.
```js
K("primus")("secundus")
  //=> "primus"
  
K(I)("primus")("secundus")
  //=> "secundus"
```
If we are not feeling particularly academic, we can name our functions:
```js
const first = K,
      second = K(I);
      
first("primus")("secundus")
  //=> "primus"
  
second("primus")("secundus")
  //=> "secundus"
```
> This is very interesting. Given two values, we can say that `K` always returns the *first* value, and given two values, `K(I)` always returns the *second* value.

### backwardness

Our `first` and `second` functions are a little different than what most people are used to when we talk about functions that access data. If we represented a pair of values as an array, we'd write them like this:
```js
const first = ([first, second]) => first,
      second = ([first, second]) => second;
      
const latin = ["primus", "secundus"];
      
first(latin)
  //=> "primus"
  
second(latin)
  //=> "secundus"
```
Or if we were using a POJO, we'd write them like this:
```js
const first = ({first, second}) => first,
      second = ({first, second}) => second;
      
const latin = {first: "primus", second: "secundus"};
      
first(latin)
  //=> "primus"
  
second(latin)
  //=> "secundus"
```
In both cases, the functions `first` and `second` know how the data is represented, whether it be an array or an object. You pass the data to these functions, and they extract it.

But the `first` and `second` we built out of `K` and `I` don't work that way. You call them and pass them the bits, and they choose what to return. So if we wanted to use them with a two-element array, we'd need to have a piece of code that calls some code.

Here's the first cut:
```js
const first = K,
      second = K(I);
      
const latin = (selector) => selector("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
```
Our `latin` data structure is no longer a dumb data structure, it's a function. And instead of passing `latin` to `first` or `second`, we pass `first` or `second` to `latin`. It's *exactly backwards* of the way we write functions that operate on data.

### the vireo

Given that our `latin` data is represented as the function `(selector) => selector("primus")("secundus")`, our obvious next step is to make a function that makes data. For arrays, we'd write `cons = (first, second) => [first, second]`. For objects we'd write: `cons = (first, second) => {first, second}`. In both cases, we take two parameters, and return the form of the data.

For "data" we access with `K` and `K(I)`, our "structure" is the function `(selector) => selector("primus")("secundus")`. Let's extract those into parameters:
```js
(first, second) => (selector) => selector(first)(second)
```
For consistency with the way combinators are written as functions taking just one parameter, we'll [curry] the function:
```js
(first) => (second) => (selector) => selector(first)(second)
```
[curry]: https://en.wikipedia.org/wiki/Currying

Let's try it, we'll use the word `pair` for the function that makes data (When we need to refer to a specific pair, we'll use the name `aPair` by default):
```js
const first = K,
      second = K(I),
      pair = (first) => (second) => (selector) => selector(first)(second);

const latin = pair("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
```
It works! Now what is this `pair` function? If we change the names to `x`, `y`, and `z`, we get: `(x) => (y) => (z) => z(x)(y)`. That's the V combinator, the Vireo! So we can write:
```js
const first = K,
      second = K(I),
      pair = V;

const latin = pair("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
```
> As an aside, the Vireo is a little like JavaScript's `.apply` function. It says, "take these two values and apply them to this function." There are other, similar combinators that apply values to functions. One notable example is the "thrush" or T combinator: It takes one value and applies it to a function. It is known to most programmers as `.tap`.

Armed with nothing more than `K`, `I`, and `V`, we can make a little data structure that holds two values, the `cons` cell of Lisp and the node of a linked list. Without arrays, and without objects, just with functions. We'd better try it out to check.

### lists with functions as data

Here's another look at linked lists using POJOs. We use the term `rest` instead of `second`, but it's otherwise identical to what we have above:
```js
const first = ({first, rest}) => first,
      rest  = ({first, rest}) => rest,
      pair = (first, rest) => ({first, rest}),
      EMPTY = ({});
      
const l123 = pair(1, pair(2, pair(3, EMPTY)));

first(l123)
  //=> 1

first(rest(l123))
  //=> 2

first(rest(rest(l123)))
  //=3
```
We can write `length` and `mapWith` functions over it:
```js
const length = (aPair) =>
  aPair === EMPTY
    ? 0
    : 1 + length(rest(aPair));

length(l123)
  //=> 3

const reverse = (aPair, delayed = EMPTY) =>
  aPair === EMPTY
    ? delayed
    : reverse(rest(aPair), pair(first(aPair), delayed));

const mapWith = (fn, aPair, delayed = EMPTY) =>
  aPair === EMPTY
    ? reverse(delayed)
    : mapWith(fn, rest(aPair), pair(fn(first(aPair)), delayed));
    
const doubled = mapWith((x) => x * 2, l123);

first(doubled)
  //=> 2

first(rest(doubled))
  //=> 4

first(rest(rest(doubled)))
  //=> 6
```
Can we do the same with the linked lists we build out of functions? Yes:
```js
const first = K,
      rest  = K(I),
      pair = V,
      EMPTY = (() => {});
      
const l123 = pair(1)(pair(2)(pair(3)(EMPTY)));

l123(first)
  //=> 1

l123(rest)(first)
  //=> 2

return l123(rest)(rest)(first)
  //=> 3
```
We write them in a backwards way, but they seem to work. How about `length`?
```js
const length = (aPair) =>
  aPair === EMPTY
    ? 0
    : 1 + length(aPair(rest));
    
length(l123)
  //=> 3
```
And `mapWith`?
```js
const reverse = (aPair, delayed = EMPTY) =>
  aPair === EMPTY
    ? delayed
    : reverse(aPair(rest), pair(aPair(first))(delayed));

const mapWith = (fn, aPair, delayed = EMPTY) =>
  aPair === EMPTY
    ? reverse(delayed)
    : mapWith(fn, aPair(rest), pair(fn(aPair(first)))(delayed));
    
const doubled = mapWith((x) => x * 2, l123)

doubled(first)
  //=> 2

doubled(rest)(first)
  //=> 4

doubled(rest)(rest)(first)
  //=> 6
```
Presto, **we can use pure functions to represent a linked list**. And with care, we can do amazing things like use functions to represent numbers, build more complex data structures like trees, and in fact, anything that can be computed can be computed using just functions and nothing else.

But without building our way up to something insane like writing a JavaScript interpreter using JavaScript functions and no other data structures, let's take things another step in a slightly different direction.

We used functions to replace arrays and POJOs, but we still use JavaScript's built-in operators to test for equality (`===`) and to branch `?:`.

### say "please"

We keep using the same pattern in our functions: `aPair === EMPTY ? doSomething : doSomethingElse`. This follows the philosophy we used with data structures: The function doing the work inspects the data structure.

We can reverse this: Instead of asking a pair if it is empty and then deciding what to do, we can ask the pair to do it for us. Here's `length` again:
```js
const length = (aPair) =>
  aPair === EMPTY
    ? 0
    : 1 + length(aPair(rest));
```
Let's presume we are working with a slightly higher abstraction, we'll call it a `list`. Instead of writing `length(list)` and examining a list, we'll write something like:
```js
const length = (list) => list(
  () => 0,
  (aPair) => 1 + length(aPair(rest)))
);
```
Now we'll need to write `first` and `rest` functions for a list, and those names will collide with the `first` and `rest` we wrote for pairs. So let's disambiguate our names:
```js
const pairFirst = K,
      pairRest  = K(I),
      pair = V;
      
const first = (list) => list(
    () => "ERROR: Can't take first of an empty list",
    (aPair) => aPair(pairFirst)
  );
      
const rest = (list) => list(
    () => "ERROR: Can't take first of an empty list",
    (aPair) => aPair(pairRest)
  );

const length = (list) => list(
    () => 0,
    (aPair) => 1 + length(aPair(pairRest)))
  );
```
We'll also write a handy list printer:
```js
const print = (list) => list(
    () => "",
    (aPair) => `${aPair(pairFirst)} ${print(aPair(pairRest))}`
  );
```
How would all this work? Let's start with the obvious. What is an empty list?
```js
const EMPTYLIST = (whenEmpty, unlessEmpty) => whenEmpty()
```
And what is a node of a list?
```js
const node = (x) => (y) =>
  (whenEmpty, unlessEmpty) => unlessEmpty(pair(x)(y));
```
Let's try it:
```js
const l123 = node(1)(node(2)(node(3)(EMPTYLIST)));

print(l123)
  //=> 1 2 3
```
We can write `reverse` and `mapWith` as well. We aren't being super-strict about emulating combinatory logic, we'll use default parameters:
```js
const reverse = (list, delayed = EMPTYLIST) => list(
  () => delayed,
  (aPair) => reverse(aPair(pairRest), node(aPair(pairFirst))(delayed))
);

print(reverse(l123));
  //=> 3 2 1
  
const mapWith = (fn, list, delayed = EMPTYLIST) =>
  list(
    () => reverse(delayed),
    (aPair) => mapWith(fn, aPair(pairRest), node(fn(aPair(pairFirst)))(delayed))
  );
  
print(mapWith(x => x * x, reverse(l123)))
  //=> 941
```
We have managed to provide the exact same functionality that `===` and `?:` provided, but using functions and nothing else.

### functions are not the real point

There are lots of similar texts explaining how to construct complex semantics out of functions. You can establish that `K` and `K(I)` can represent `true` and `false`, model magnitudes with [Church Numerals] or [Surreal Numbers], and build your way up to printing FizzBuzz.

The superficial conclusion reads something like this:

[Church Numerals]: https://en.wikipedia.org/wiki/Church_encoding
[Surreal Numbers]: https://en.wikipedia.org/wiki/Surreal_number

> Functions are a fundamental building block of computation. They are "axioms" of combinatory logic, and can be used to compute anything that JavaScript can compute.

However, that is not the interesting thing to note here. Practically speaking, languages like JavaScript already provide arrays with mapping and folding methods, choice operations, and other rich constructs. Knowing how to make a linked list out of functions is not really necessary for the working programmer. (Knowing that it can be done, on the other hand, is very important to understanding computer science.)

Knowing how to make a list out of just functions is a little like knowing that photons are the [Gauge Bosons] of the electromagnetic force. It's the QED of physics that underpins the Maxwell's Equations of programming. Deeply important, but not practical when you're building a bridge.

[Gauge Bosons]: https://en.wikipedia.org/wiki/Gauge_boson

So what *is* interesting about this? What nags at our brain as we're falling asleep after working our way through this?

### a return to backward thinking

To make pairs work, we did things *backwards*, we passed the `first` and `rest` functions to the pair, and the pair called our function. As it happened, the pair was composed by the vireo (or V combinator): `(x) => (y) => (z) => z(x)(y)`.

But we could have done something completely different. We could have written a pair that stored its elements in an array, or a pair that stored its elements in a POJO. All we know is that we can pass the pair function a function of our own, at it will be called with the elements of the pair.

The exact implementation of a pair is hidden from the code that uses a pair. Here, we'll prove it:
```js
const first = K,
      second = K(I),
      pair = (first) => (second) => {
        const pojo = {first, second};
        
        return (selector) => selector(pojo.first)(pojo.second);
      };

const latin = pair("primus")("secundus");

latin(first)
  //=> "primus"
  
latin(second)
  //=> "secundus"
```
This is a little gratuitous, but it makes the point: The code that uses the data doesn't reach in and touch it: The code that uses the data provides some code and asks the data to do something with it.

The same thing happens with our lists. Here's `length` for lists:
```js
const length = (list) => list(
    () => 0,
    (aPair) => 1 + length(aPair(pairRest)))
  );
```
We're passing `list` what we want done with an empty list, and what we want done with a list that has at least one element. We then ask `list` to do it, and provide a way for `list` to call the code we pass in.

We won't bother here, but it's easy to see how to swap our functions out and replace them with an array. Or a column in a database. This is fundamentally *not* the same thing as this code for the length of a linked list:
```js
const length = (node, delayed = 0) =>
  node === EMPTY
    ? delayed
    : length(node.rest, delayed + 1);
```
The line `node === EMPTY` presumes a lot of things. It presumes there is one canonical empty list value. It presumes you can compare these things with the `===` operator. We can fix this with an `isEmpty` function, but now we're pushing even more knowledge about the structure of lists into the code that uses them.

Having a list know itself whether it is empty hides implementation information from the code that uses lists. This is a fundamental principle of good design. It is a tenet of Object-Oriented Programming, but it is **not** exclusive to OOP: We can and should design data structures to hide implementation information from the code that use them, whether we are working with functions, objects, or both.

There are many tools for hiding implementation information, and we have now seen two particularly powerful patterns:

* Instead of directly manipulating part of an entity, pass it a function and have it call our function with the part we want.
* And instead of testing some property of an entity and making a choice of our own with `?:` (or `if`), pass the entity the work we want done for each case and let it test itself.
