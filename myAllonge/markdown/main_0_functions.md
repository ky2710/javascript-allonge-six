# The first sip: Basic Functions
## As Little As Possible About Functions, But No Less

In JavaScript, functions are values, but they are also much more than simple numbers, strings, or even complex data structures like trees or maps. Functions represent computations to be performed. Like numbers, strings, and arrays, they have a representation. Let's start with the second simplest possible function.`[1]` In JavaScript, it looks like this:
```js
() => 0
```    
This is a function that is applied to no values and returns `0`. Let's verify that our function is a value like all others:
```js
(() => 0)
  //=> [Function]
```       
What!? Why didn't it type back `() => 0` for us? This *seems* to break our rule that if an expression is also a value, JavaScript will give the same value back to us. What's going on? The simplest and easiest answer is that although the JavaScript interpreter does indeed return that value, displaying it on the screen is a slightly different matter. `[Function]` is a choice made by the people who wrote Node.js, the JavaScript environment that hosts the JavaScript REPL. If you try the same thing in a browser, you may see something else.

>`[1]` The simplest possible function is `() => {}`, we'll see that later.

> I'd prefer something else, but I must accept that what gets typed back to us on the screen is arbitrary, and all that really counts is that it is somewhat useful for a human to read. But we must understand that whether we see `[Function]` or `() => 0`, internally JavaScript has a full and proper function.

### functions and identities

You recall that we have two types of values with respect to identity: Value types and reference types. Value types share the same identity if they have the same contents. Reference types do not.

Which kind are functions? Let's try them out and see. For reasons of appeasing the JavaScript parser, we'll enclose our functions in parentheses:
```javascript
    (() => 0) === (() => 0)
      //=> false
```      
Like arrays, every time you evaluate an expression to produce a function, you get a new function that is not identical to any other function, even if you use the same expression to generate it. "Function" is a reference type.

### applying functions

Let's put functions to work. The way we use functions is to *apply* them to zero or more values called *arguments*. Just as `2 + 2` produces a value (in this case `4`), **applying a function to zero or more arguments produces a value** as well.

Here's how we apply a function to some values in JavaScript: Let's say that *fn_expr* is an expression that when evaluated, produces a function. Let's call the arguments *args*. Here's how to apply a function to some arguments:

  *fn_expr*`(`*args*`)`
    
Right now, we only know about one such expression: `() => 0`, so let's use it. We'll put it in parentheses`[1]` to keep the parser happy, like we did above: `(() => 0)`. Since we aren't giving it any arguments, we'll simply write `()` after the expression. So we write:
```javascript
    (() => 0)()
      //=> 0
```

>`[1]` If you're used to other programming languages, you've probably internalized the idea that sometimes parentheses are used to group operations in an expression like math, and sometimes to apply a function to arguments. If not... Welcome to the [ALGOL] family of programming languages!

[ALGOL]: https://en.wikipedia.org/wiki/ALGOL

### functions that return values and evaluate expressions

We've seen `() => 0`. We know that `(() => 0)()` returns `0`, and this is unsurprising. Likewise, the following all ought to be obvious:

```javascript
  (() => 1)()
    //=> 1
  (() => "Hello, JavaScript")()
    //=> "Hello, JavaScript"
  (() => Infinity)()
    //=> Infinity
```

Well, the last one's a doozy, but still, the general idea is this: We can make a function that returns a value by putting the value to the right of the arrow.

In the prelude, we looked at expressions. Values like `0` are expressions, as are things like `40 + 2`. Can we put an expression to the right of the arrow?
```javascript
    (() => 1 + 1)()
      //=> 2
    (() => "Hello, " + "JavaScript")()
      //=> "Hello, JavaScript"
    (() => Infinity * Infinity)()
      //=> Infinity
```
Yes we can. We can put any expression to the right of the arrow. For example, `(() => 0)()` is an expression. Can we put it to the right of an arrow, like this: `() => (() => 0)()`?

Let's try it:
```javascript
    (() => (() => 0)())()
      //=> 0
```
Yes we can! Functions can return the value of evaluating another function.

When dealing with expressions that have a lot of the same characters (like parentheses), you may find it helpful to format the code to make things stand out. So we can also write:
```javascript
    (() =>
        (() => 0
          )()
      )()
      //=> 0
```
It evaluates to the same thing, `0`.

### commas

The comma operator in JavaScript is interesting. It takes two arguments, evaluates them both, and itself evaluates to the value of the right-hand argument. In other words:
```javascript
    (1, 2)
      //=> 2

    (1 + 1, 2 + 2)
      //=> 4
```
We can use commas with functions to create functions that evaluate multiple expressions:
```javascript
    (() => (1 + 1, 2 + 2))()
      //=> 4
```
This is useful when trying to do things that might involve *side-effects*, but we'll get to that later. In most cases, JavaScript does not care whether things are separated by spaces, tabs, or line breaks. So we can also write:
```javascript
    () =>
      (1 + 1, 2 + 2)
```
Or even:
```javascript
    () => (
        1 + 1,
        2 + 2
      )
```
### the simplest possible block

There's another thing we can put to the right of an arrow, a *block*. A block has zero or more *statements*, separated by semicolons.`[1]`

>`[1]` Sometimes, you will find JavaScript that has statements that are separated by newlines without semi-colons. This works because JavaScript has a feature that can infer where the semi-colons should be most of the time. We will not take advantage of this feature, but it's helpful to know it exists.

So, this is a valid function:
```javascript
    () => {}
```    
It returns the result of evaluating a block that has no statements. What would that be? Let's try it:
```javascript
    (() => {})()
      //=> undefined
```
What is this `undefined`?

### `undefined`

In JavaScript, the absence of a value is written `undefined`, and it means there is no value. It will crop up again. `undefined` is its own type of value, and it acts like a value type:
```javascript
    undefined
      //=> undefined
```
Like numbers, booleans and strings, JavaScript can print out the value `undefined`.
```javascript
    undefined === undefined
      //=> true
    (() => {})() === (() => {})()
      //=> true
    (() => {})() === undefined
      //=> true
```      
No matter how you evaluate `undefined`, you get an identical value back. **`undefined` is a value that means "I don't have a value." But it's still a value**:smile:
      
> You might think that `undefined` in JavaScript is equivalent to `NULL` in SQL. No. In SQL, two things that are `NULL` are not equal to nor share the same identity, because two unknowns can't be equal. In JavaScript, every `undefined` is identical to every other `undefined`.

### void

We've seen that JavaScript represents an undefined value by typing `undefined`, and we've generated undefined values in two ways:

1. By evaluating a function that doesn't return a value `(() => {})()`, and;
2. By writing `undefined` ourselves.

There's a **third way**, with JavaScript's `void` operator. Behold:
```javascript
    void 0
      //=> undefined
    void 1
      //=> undefined
    void (2 + 2)
      //=> undefined
```     
`void` is an operator that takes any value and evaluates to `undefined`, always. So, when we deliberately want an undefined value, should we use the first, second, or third form?`[1]` The answer is, use `void`. By convention, use `void 0`.

>`[1]` Experienced JavaScript programmers are aware that there's a **fourth way**, using a function argument. This was actually the preferred mechanism until `void` became commonplace.

The first form works but it's cumbersome. The second form works most of the time, but it is possible to break it by reassigning `undefined` to a different value, something we'll discuss in [Reassignment and Mutation](main_1_Composing.md#reassignment). The third form is guaranteed to always work, so that's what we will use.`[1]`

>`[1]` As an exercise for the reader, we suggest you ask your friendly neighbourhood programming language designer or human factors subject-matter expert to explain why a keyword called `void` is used to generate an `undefined` value, instead of calling them both `void` or both `undefined`. We have no idea.

### back on the block

Back to our function. We evaluated this:
```javascript
    (() => {})()
      //=> undefined
```
We said that the function returns the result of evaluating a *block*, and we said that a block is a (possibly empty) list of JavaScript *statements* separated by semicolons.`[1]`

>`[1]` You can also separate statements with line breaks. Readers who follow internet flame-fests may be aware of something called [automatic semi-colon insertion](http://lucumr.pocoo.org/2011/2/6/automatic-semicolon-insertion/). Basically, there's a step where JavaScript looks at your code and follows some rules to guess where you meant to put semicolons in should you leave them out. This feature was originally created as a kind of helpful error-correction. Some programmers argue that since it's part of the language's definition, it's fair game to write code that exploits it, so they deliberately omit any semicolon that JavaScript will insert for them.

Something like: { statement^1; statement^2; statement^3; ... ; statement^n }

We haven't discussed these *statements*. What's a statement?

There are many kinds of JavaScript statements, but the first kind is one we've already met. An expression is a JavaScript statement. Although they aren't very practical, these are valid JavaScript functions, and they return `undefined` when applied:
```javascript
    () => { 2 + 2 }
    () => { 1 + 1; 2 + 2 }
```    
As we saw with commas above, we can rearrange these functions onto multiple lines when we feel its more readable that way:
```javascript
    () => {
        1 + 1;
        2 + 2
      }
```
But no matter how we arrange them, a block with one or more expressions still evaluates to `undefined`:
```javascript
    (() => { 2 + 2 })()
      //=> undefined
      
    (() => { 1 + 1; 2 + 2 })()
      //=> undefined
      
    (() => {
        1 + 1;
        2 + 2
      })()
      //=> undefined
```
As you can see, a block with one expression does not behave like an expression, and a block with more than one expression does not behave like an expression constructed with the comma operator:
```javascript
    (() => 2 + 2)()
      //=> 4
    (() => { 2 + 2 })()
      //=> undefined
      
    (() => (1 + 1, 2 + 2))()
      //=> 4
    (() => { 1 + 1; 2 + 2 })()
      //=> undefined
```
So how do we get a function that evaluates a block to return a value when applied? With the `return` keyword and any expression:
```javascript
    (() => { return 0 })()
      //=> 0
      
    (() => { return 1 })()
      //=> 1
      
    (() => { return 'Hello ' + 'World' })()
      // 'Hello World'
```      
The `return` keyword creates a *return statement* that immediately terminates the function application and returns the result of evaluating its expression. For example:
```javascript      
    (() => {
        1 + 1;
        return 2 + 2
      })()
      //=> 4
```
And also:
```javascript      
    (() => {
        return 1 + 1;
        2 + 2
      })()
      //=> 2
```
The return statement is the first statement we've seen, and it behaves differently than an expression. For example, you can't use one as the expression in a simple function, because it isn't an expression:
```javascript
    (() => return 0)()
      //=> ERROR
```
Statements belong inside blocks and only inside blocks. Some languages simplify this by making everything an expression, but JavaScript maintains this distinction, so when learning JavaScript we also learn about statements like function declarations, for loops, if statements, and so forth. We'll see a few more of these later.

### functions that evaluate to functions

If an expression that evaluates to a function is, well, an expression, and if a return statement can have any expression on its right side... *Can we put an expression that evaluates to a function on the right side of a function expression?*

Yes:
```javascript
    () => () => 0
```    
That's a function! It's a function that when applied, evaluates to a function that when applied, evaluates to `0`.  So we have *a function, that returns a function, that returns zero*. Likewise:
```javascript
    () => () => true
```    
That's a function, that returns a function, that returns `true`:
```javascript
    (() => () => true)()()
      //=> true
```      
We could, of course, do the same thing with a block if we wanted:
```javascript
    () => () => { return true; }
```
But we generally don't.

---

Well. We've been very clever, but so far this all seems very abstract. Diffraction of a crystal is beautiful and interesting in its own right, but you can't blame us for wanting to be shown a practical use for it, like being able to determine the composition of a star millions of light years away. So... In the next chapter, "[I'd Like to Have an Argument, Please](#fargs)," we'll see how to make functions practical.

## Ah. I'd Like to Have an Argument, Please.

 Up to now, we've looked at functions without arguments. We haven't even said what an argument *is*, only that our functions don't have any.

> Most programmers are perfectly familiar with arguments (often called "parameters"). Secondary school mathematics discusses this. So you know what they are, and I know that you know what they are, but please be patient with the explanation!

Let's make a function with an argument:
```javascript
    (room) => {}
```
This function has one argument, `room`, and an empty body. Here's a function with two arguments and an empty body:
```javascript
    (room, board) => {}
```
I'm sure you are perfectly comfortable with the idea that this function has two arguments, `room`, and `board`. What does one do with the arguments? Use them in the body, of course. What do you think this is?
```javascript
    (diameter) => diameter * 3.14159265
```
It's a function for calculating the circumference of a circle given the diameter. I read that aloud as "When applied to a value representing the diameter, this function *returns* the diameter times 3.14159265."

Remember that to **apply a function with no arguments**, we wrote `(() => {})()`. To **apply a function with an argument (or arguments)**, we put the argument (or arguments) within the parentheses, like this:
```javascript
    ((diameter) => diameter * 3.14159265)(2)
      //=> 6.2831853
```
You won't be surprised to see how to write and apply a function to two arguments:
```javascript
    ((room, board) => room + board)(800, 150)
      //=> 950
```
> ### a quick summary of functions and bodies
>
> How arguments are used in a body's expression is probably perfectly obvious to you from the examples, especially if you've used any programming language (except for the dialect of BASIC--which I recall from my secondary school--that didn't allow parameters when you called a procedure).
>
> Expressions consist either of representations of values (like `3.14159265`, `true`, and `undefined`), operators that combine expressions (like `3 + 2`), some special forms like `[1, 2, 3]` for creating arrays out of expressions, or `function (`*arguments*`) {`*body-statements*`}` for creating functions.
>
> One of the important possible statements is a return statement. A return statement accepts any valid JavaScript expression.
>
> This loose definition is recursive, so we can intuit (or use our experience with other languages) that since a function can contain a return statement with an expression, we can write a function that returns a function, or an array that contains another array expression. Or a function that returns an array, an array of functions, a function that returns an array of functions, and so forth:
>
>````js
>  () => () => {};
>  () => [ 1, 2, 3];
>  [1, [2, 3], 4];
>  () => [
>      () => 1,
>      () => 2,
>      () => 3
>    ];
>````
### call by value

Like most contemporary programming languages, JavaScript uses the "call by value" [evaluation strategy]. That means that when you write some code that appears to apply a function to an expression or expressions, JavaScript evaluates all of those expressions and applies the functions to the resulting value(s).

[evaluation strategy]: http://en.wikipedia.org/wiki/Evaluation_strategy

So when you write:
```javascript
    ((diameter) => diameter * 3.14159265)(1 + 1)
      //=> 6.2831853
```
What happened internally is that the expression `1 + 1` was evaluated first, resulting in `2`. Then our circumference function was applied to `2`.`[1]`

>`[1]` We said that you can't apply a function to an expression. You *can* apply a function to one or more functions. Functions are values! This has interesting applications, and they will be explored much more thoroughly in [Functions That Are Applied to Functions](#consumers).

We'll see [below](#call-by-sharing) that while JavaScript always calls by value, the notion of a "value" has additional subtlety. But before we do, let's look at variables.

### variables and bindings

Right now everything looks simple and straightforward, and we can move on to talk about arguments in more detail. And we're going to work our way up from `(diameter) => diameter * 3.14159265` to functions like:
```javascript
    (x) => (y) => x
```
> `(x) => (y) => x` just looks crazy, as if we are learning English as a second language and the teacher promises us that soon we will be using words like *antidisestablishmentarianism*. Besides a desire to use long words to sound impressive, this is not going to seem attractive until we find ourselves wanting to discuss the role of the Church of England in 19th century British politics.
>
> But there's another reason for learning the word *antidisestablishmentarianism*: We might learn how prefixes and postfixes work in English grammar. It's the same thing with `(x) => (y) => x`. It has a certain important meaning in its own right, and it's also an excellent excuse to learn about functions that make functions, environments, variables, and more.

In order to talk about how this works, we should agree on a few terms (you may already know them, but let's check-in together and "synchronize our dictionaries"). The first `x`, the one in `(x) => ...`, is an *argument*. The `y` in `function (y) ...` is another argument. The second `x`, the one in `=> x`, is not an argument, *it's an expression referring to a variable*. Arguments and variables work the same way whether we're talking about `(x) => (y) => x`  or just plain `(x) => x`.

Every time a function is invoked ("invoked" means "applied to zero or more arguments"), a new *environment* is created. An environment is a (possibly empty) dictionary that maps variables to values by name. The `x` in the expression that we call a "variable" is itself an expression that is evaluated by looking up the value in the environment.

How does the value get put in the environment? Well for arguments, that is very simple. When you apply the function to the arguments, an entry is placed in the dictionary for each argument. So when we write:
```js
    ((x) => x)(2)
      //=> 2
```
What happens is this:

1. JavaScript parses this whole thing as an expression made up of several sub-expressions.
1. It then starts evaluating the expression, including evaluating sub-expressions
1. One sub-expression, `(x) => x` evaluates to a function.
1. Another, `2`, evaluates to the number 2.
1. JavaScript now evaluates applying the function to the argument `2`. Here's where it gets interesting...
1. An environment is created.
1. The value '2' is bound to the name 'x' in the environment.
1. The expression 'x' (the right side of the function) is evaluated within the environment we just created.
1. The value of a variable when evaluated in an environment is the value bound to the variable's name in that environment, which is '2'
1. And that's our result.

When we talk about environments, we'll use an [unsurprising syntax][json] for showing their bindings: `{x: 2, ...}`. meaning, that the environment is a dictionary, and that the value `2` is bound to the name `x`, and that there might be other stuff in that dictionary we aren't discussing right now.

[json]: http://json.org/

### call by sharing

Earlier, we distinguished JavaScript's *value types* from its *reference types*. At that time, we looked at how JavaScript distinguishes objects that are identical from objects that are not. Now it is time to take another look at the distinction between value and reference types.

There is a property that JavaScript strictly maintains: When a value--any value--is passed as an argument to a function, the value bound in the function's environment must be identical to the original.

We said that JavaScript binds names to values, but we didn't say what it means to bind a name to a value. Now we can elaborate: When JavaScript binds a value-type to a name, it makes a copy of the value and places the copy in the environment. As you recall, value types like strings and numbers are identical to each other if they have the same content. So JavaScript can make as many copies of strings, numbers, or booleans as it wishes.

What about reference types? JavaScript does not place copies of reference values in any environment. JavaScript places *references* to reference types in environments, and when the value needs to be used, JavaScript uses the reference to obtain the original.

Because many references can share the same value, and because JavaScript passes references as arguments, JavaScript can be said to implement "call by sharing" semantics. Call by sharing is generally understood to be a specialization of call by value, and it explains why some values are known as value types and other values are known as reference types.

And with that, we're ready to look at *closures*. When we combine our knowledge of value types, reference types, arguments, and closures, we'll understand why this function always evaluates to `true` no matter what argument `[1]` you apply it to:
```javascript
    (value) =>
      ((ref1, ref2) => ref1 === ref2)(value, value)
```
>`[1]` Unless the argument is `NaN`, which isn't equal to anything, *including itself*. `NaN` in JavaScript behaves a lot like `NULL` in SQL.
## Closures and Scope

It's time to see how a function within a function works:
```javascript
    ((x) => (y) => x)(1)(2)
      //=> 1
```
First off, let's use what we learned above. Given `(`*some function*`)(`*some argument*`)`, we know that we apply the function to the argument, create an environment, bind the value of the argument to the name, and evaluate the function's expression. So we do that first with this code:
```javascript
    ((x) => (y) => x)(1)
      //=> [Function]
```      
The environment belonging to the function with signature `(x) => ...` becomes `{x: 1, ...}`, and the result of applying the function is another function value. It makes sense that the result value is a function, because the expression for `(x) => ...`'s body is:
```javascript
    (y) => x
```
So now we have a value representing that function. Then we're going to take the value of that function and apply it to the argument `2`, something like this:
```javascript
      ((y) => x)(2)
```
So we seem to get a new environment `{y: 2, ...}`. How is the expression `x` going to be evaluated in that function's environment? There is no `x` in its environment, it must come from somewhere else.

A> This, by the way, is one of the great defining characteristics of JavaScript and languages in the same family: Whether they allow things like functions to nest inside each other, and if so, how they handle variables from "outside" of a function that are referenced inside a function. For example, here's the equivalent code in Ruby:
A>
A> <<(code/k.rb)
A>
A> Now let's enjoy a relaxed Allongé before we continue!

### if functions without free variables are pure, are closures impure?

The function `(y) => x` is interesting. It contains a *free variable*, `x`.[^nonlocal] A free variable is one that is not bound within the function. Up to now, we've only seen one way to "bind" a variable, namely by passing in an argument with the same name. Since the function `(y) => x` doesn't have an argument named `x`, the variable `x` isn't bound in this function, which makes it "free."

[^nonlocal]: You may also hear the term "non-local variable." [Both are correct.](https://en.wikipedia.org/wiki/Free_variables_and_bound_variables) 

Now that we know that variables used in a function are either bound or free, we can bifurcate functions into those with free variables and those without:

  * Functions containing no free variables are called *pure functions*.
  * Functions containing one or more free variables are called *closures*.
  
Pure functions are easiest to understand. They always mean the same thing wherever you use them. Here are some pure functions we've already seen:
```javascript
    () => {}
    
    (x) => x
      
    (x) => (y) => x
```
The first function doesn't have any variables, therefore doesn't have any free variables. The second doesn't have any free variables, because its only variable is bound. The third one is actually two functions, one inside the other. `(y) => ...` has a free variable, but the entire expression refers to `(x) => ...`, and it doesn't have a free variable: The only variable anywhere in its body is `x`, which is certainly bound within `(x) => ...`.

From this, we learn something: A pure function can contain a closure.

X> If pure functions can contain closures, can a closure contain a pure function? Using only what we've learned so far, attempt to compose a closure that contains a pure function. If you can't, give your reasoning for why it's impossible.

Pure functions always mean the same thing because all of their "inputs" are fully defined by their arguments. Not so with a closure. If I present to you this pure function `(x, y) => x + y`, we know exactly what it does with `(2, 2)`. But what about this closure: `(y) => x + y`? We can't say what it will do with argument `(2)` without understanding the magic for evaluating the free variable `x`.

### it's always the environment

To understand how closures are evaluated, we need to revisit environments. As we've said before, all functions are associated with an environment. We also hand-waved something when describing our environment. Remember that we said the environment for `((x) => (y) => x)(1)` is `{x: 1, ...}` and that the environment for `((y) => x)(2)` is `{y: 2, ...}`? Let's fill in the blanks!

The environment for `((y) => x)(2)` is *actually* `{y: 2, '..': {x: 1, ...}}`. `'..'` means something like "parent" or "enclosure" or "super-environment." It's `(x) => ...`'s environment, because the function `(y) => x` is within `(x) => ...`'s body. So whenever a function is applied to arguments, its environment always has a reference to its parent environment.

And now you can guess how we evaluate `((y) => x)(2)` in the environment `{y: 2, '..': {x: 1, ...}}`. The variable `x` isn't in `(y) => ...`'s immediate environment, but it is in its parent's environment, so it evaluates to `1` and that's what `((y) => x)(2)` returns even though it ended up ignoring its own argument.

A> `(x) => x` is called the I Combinator, or the *Identity Function*. `(x) => (y) => x` is called the K Combinator, or *Kestrel*. Some people get so excited by this that they write entire books about them, some are [great][mock], some--how shall I put this--are [interesting][interesting] if you use Ruby.

[mock]: http://www.amzn.com/0192801422?tag=raganwald001-20
[interesting]: https://leanpub.com/combinators "Kestrels, Quirky Birds, and Hopeless Egocentricity"

Functions can have grandparents too:
```javascript
    (x) =>
       (y) =>
         (z) => x + y + z
```
This function does much the same thing as:
```javascript
    (x, y, z) => x + y + z
```    
Only you call it with `(1)(2)(3)` instead of `(1, 2, 3)`. The other big difference is that you can call it with `(1)` and get a function back that you can later call with `(2)(3)`.


A> The first function is the result of [currying] the second function. Calling a curried function with only some of its arguments is sometimes called [partial application]. Some programming languages automatically curry and partially evaluate functions without the need to manually nest them.

[currying]: https://en.wikipedia.org/wiki/Currying
[partial application]: https://en.wikipedia.org/wiki/Partial_application

### shadowy variables from a shadowy planet

An interesting thing happens when a variable has the same name as an ancestor environment's variable. Consider:
```javascript
    (x) =>
      (x, y) => x + y
```
The function `(x, y) => x + y` is a pure function, because its `x` is defined within its own environment. Although its parent also defines an `x`, it is ignored when evaluating `x + y`. JavaScript always searches for a binding starting with the functions own environment and then each parent in turn until it finds one. The same is true of:
```javascript
    (x) =>
      (x, y) =>
        (w, z) =>
          (w) =>
            x + y + z
```
When evaluating `x + y + z`, JavaScript will find `x` and `y` in the great-grandparent scope and `z` in the parent scope. The `x` in the great-great-grandparent scope is ignored, as are both `w`s. When a variable has the same name as an ancestor environment's binding, it is said to *shadow* the ancestor.

This is often a good thing.

### which came first, the chicken or the egg?

This behaviour of pure functions and closures has many, many consequences that can be exploited to write software. We are going to explore them in some detail as well as look at some of the other mechanisms JavaScript provides for working with variables and mutable state.

But before we do so, there's one final question: Where does the ancestry start? If there's no other code in a file, what is `(x) => x`'s parent environment?

JavaScript always has the notion of at least one environment we do not control: A global environment in which many useful things are bound such as libraries full of standard functions. So when you invoke `((x) => x)(1)` in the REPL, its full environment is going to look like this: `{x: 1, '..': `*global environment*`}`.

Sometimes, programmers wish to avoid this. If you don't want your code to operate directly within the global environment, what can you do? Create an environment for them, of course. Many programmers choose to write every JavaScript file like this:
```javascript
    // top of the file
    (() => {
      
      // ... lots of JavaScript ...
      
    })();
    // bottom of the file
```
The effect is to insert a new, empty environment in between the global environment and your own functions: `{x: 1, '..': {'..': `*global environment*`}}`. As we'll see when we discuss mutable state, this helps to prevent programmers from accidentally changing the global state that is shared by all code in the program.
## That Constant Coffee Craving {#const}

Up to now, all we've really seen are *anonymous functions*, functions that don't have a name. This feels very different from programming in most other languages, where the focus is on naming functions, methods, and procedures. Naming things is a critical part of programming, but all we've seen so far is how to name arguments.

There are other ways to name things in JavaScript, but before we learn some of those, let's see how to use what we already have to name things. Let's revisit a very simple example:
```javascript
    (diameter) => diameter * 3.14159265
```
What is this "3.14159265" number? [PI], obviously. We'd like to name it so that we can write something like:
```javascript
    (diameter) => diameter * PI
```
In order to bind `3.14159265` to the name `PI`, we'll need a function with a parameter of `PI` applied to an argument of `3.14159265`. If we put our function expression in parentheses, we can apply it to the argument of `3.14159265`:
```javascript
    ((PI) => 
      // ????
    )(3.14159265)
```
What do we put inside our new function that binds `3.14159265` to the name `PI` when evaluated? Our circumference function, of course:

[PI]: https://en.wikipedia.org/wiki/Pi
```javascript
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)
```
This expression, when evaluated, returns a function that calculates circumferences. That sounds bad, but when we think about it, `(diameter) => diameter * 3.14159265` is also an expression, that when evaluated, returns a function that calculates circumferences. All of our "functions" are expressions. This one has a few more moving parts, that's all. But we can use it just like `(diameter) => diameter * 3.14159265`.

Let's test it:
```javascript
    ((diameter) => diameter * 3.14159265)(2)
      //=> 6.2831853
      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)(2)
      //=> 6.2831853
```
That works! We can bind anything we want in an expression by wrapping it in a function that is immediately invoked with the value we want to bind.[^explain-iife]

[^explain-iife]: JavaScript programmers regularly use the idea of writing an expression that denotes a function and then immediately applying it to arguments. Explaining the pattern, Ben Alman coined the term [Immediately Invoked Function Expression][iife] for it, often abbreviated "IIFE."

### inside-out

There's another way we can make a function that binds `3.14159265` to the name `PI` and then uses that in its expression. We can turn things inside-out by putting the binding inside our diameter calculating function, like this:
```javascript
    (diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265)
```
It produces the same result as our previous expressions for a diameter-calculating function:
```javascript
    ((diameter) => diameter * 3.14159265)(2)
      //=> 6.2831853
      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)(2)
      //=> 6.2831853

    ((diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265))(2)
      //=> 6.2831853
```
Which one is better? Well, the first one seems simplest, but a half-century of experience has taught us that names matter. A "magic literal" like `3.14159265` is anathema to sustainable software development.

The third one is easiest for most people to read. It separates concerns nicely: The "outer" function describes its parameters:
```javascript
    (diameter) =>
      // ...
```
Everything else is encapsulated in its body. That's how it should be, naming `PI` is its concern, not ours. The other formulation:
```javascript
    ((PI) =>
      // ...
    )(3.14159265)
```
"Exposes" naming `PI` first, and we have to look inside to find out why we care. So, should we should always write this?
```javascript
    (diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265)
```     
Well, the wrinkle with this is that typically, invoking functions is considerably more expensive than evaluating expressions. Every time we invoke the outer function, we'll invoke the inner function. We could get around this by writing 
```javascript      
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)
```    
But then we've obfuscated our code, and we don't want to do that unless we absolutely have to.

What would be very nice is if the language gave us a way to bind names inside of blocks without incurring the cost of a function invocation. And JavaScript does.

### const

Another way to write our "circumference" function would be to pass `PI` along with the diameter argument, something like this:
```javascript
    (diameter, PI) => diameter * PI
```
And we could use it like this:
```javascript
    ((diameter, PI) => diameter * PI)(2, 3.14159265)
      //=> 6.2831853
```
This differs from our example above in that there is only one environment, rather than two. We have one binding in the environment representing our regular argument, and another our "constant." That's more efficient, and it's *almost* what we wanted all along: A way to bind `3.14159265` to a readable name.

JavaScript gives us a way to do that, the `const` keyword. We'll learn a lot more about `const` in future chapters, but here's the most important thing we can do with `const`:
```javascript
    (diameter) => {
      const PI = 3.14159265;

      return diameter * PI
    }
```
The `const` keyword introduces one or more bindings in the block that encloses it. It doesn't incur the cost of a function invocation. That's great. Even better, it puts the symbol (like `PI`) close to the value (`3.14159265`). That's much better than what we were writing.

We use the `const` keyword in a *const statement*. `const` statements occur inside blocks, we can't use them when we write a fat arrow that has an expression as its body.

It works just as we want. Instead of:
```javascript
    ((diameter) =>
      ((PI) =>
        diameter * PI)(3.14159265))(2)
```        
Or:
```javascript
    ((diameter, PI) => diameter * PI)(2, 3.14159265)
      //=> 6.2831853
```      
We write:
```javascript
    ((diameter) => {
      const PI = 3.14159265;

      return diameter * PI
    })(2)
      //=> 6.2831853
```
We can bind any expression. Functions are expressions, so we can bind helper functions:
```javascript
    (d) => {
      const calc = (diameter) => {
        const PI = 3.14159265;

        return diameter * PI
      };

      return "The circumference is " + calc(d)
    }
```
Notice `calc(d)`? This underscores what we've said: if we have an expression that evaluates to a function, we apply it with `()`. A name that's bound to a function is a valid expression evaluating to a function.[^namedfn]

[^namedfn]: We're into the second chapter and we've finally named a function. Sheesh.

A> Amazing how such an important idea--naming functions--can be explained *en passant* in just a few words. That emphasizes one of the things JavaScript gets really, really right: Functions as "first class entities." Functions are values that can be bound to names like any other value, passed as arguments, returned from other functions, and so forth.

We can bind more than one name-value pair by separating them with commas. For readability, most people put one binding per line:
```javascript
    (d) => {
      const PI   = 3.14159265,
          calc = (diameter) => diameter * PI;

      return "The circumference is " + calc(d)
    }
```
### nested blocks

Up to now, we've only ever seen blocks we use as the body of functions. But there are other kinds of blocks. One of the places you can find blocks is in an `if` statement. In JavaScript, an `if` statement looks like this:
```javascript
    (n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else
          return !even(x - 1);
      }
      return even(n)
    }
```    
And it works for fairly small numbers:
```javascript
    ((n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else
          return !even(x - 1);
      }
      return even(n)
    })(13)
      //=> false
```
The `if` statement is a statement, not an expression (an unfortunate design choice), and its clauses are statements or blocks. So we could also write something like:
```javascript
    (n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else {
          const odd = (y) => !even(y);
          
          return odd(x - 1);
        }
      }
      return even(n)
    }
```
And this also works:
```javascript
    ((n) => {
      const even = (x) => {
        if (x === 0)
          return true;
        else {
          const odd = (y) => !even(y);
          
          return odd(x - 1);
        }
      }
      return even(n)
    })(42)
      //=> true
```    
We've used a block as the `else` clause, and since it's a block, we've placed a `const` statement inside it.

### const and lexical scope

This seems very straightforward, but alas, there are some semantics of binding names that we need to understand if we're to place `const` anywhere we like. The first thing to ask ourselves is, what happens if we use `const` to bind two different values to the "same" name?

Let's back up and reconsider how closures work. What happens if we use parameters to bind two different values to the same name?

Here's the second formulation of our diameter function, bound to a name using an IIFE:
```javascript
    ((diameter_fn) =>
      // ...
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )
```
It's more than a bit convoluted, but it binds `((PI) => (diameter) => diameter * PI)(3.14159265)` to `diameter_fn` and evaluates the expression that we've elided. We can use any expression in there, and that expression can invoke `diameter_fn`. For example:
```javascript
    ((diameter_fn) =>
      diameter_fn(2)
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )
      //=> 6.2831853
```      
We know this from the chapter on [closures](#closures), but even though `PI` is not bound when we invoke `diameter_fn` by evaluating `diameter_fn(2)`, `PI` *is* bound when we evaluated `(diameter) => diameter * PI`, and thus the expression `diameter * PI` is able to access values for `PI` and `diameter` when we evaluate `diameter_fn`.

This is called [lexical scoping], because we can discover where a name is bound by looking at the source code for the program. We can see that `PI` is bound in an environment surrounding `(diameter) => diameter * PI`, we don't need to know where `diameter_fn` is invoked.

We can test this by deliberately creating a "conflict:"
```javascript
    ((diameter_fn) =>
      ((PI) =>
        diameter_fn(2)
      )(3)
    )(
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )
      //=> 6.2831853
```
Although we have bound `3` to `PI` in the environment surrounding `diameter_fn(2)`, the value that counts is `3.14159265`, the value we bound to `PI` in the environment surrounding (diameter) => diameter * PI.

That much we can carefully work out from the way closures work. Does `const` work the same way? Let's find out:
```javascript
    ((diameter_fn) => {
      const PI = 3;
      
      return diameter_fn(2)
    })(
      (() => {
        const PI = 3.14159265;
        
        return (diameter) => diameter * PI
      })()
    )
      //=> 6.2831853
```
Yes. Binding values to names with `const` works just like binding values to names with parameter invocations, it uses lexical scope.

[lexical scoping]: https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scope_vs._dynamic_scope

### are consts also from a shadowy planet?

We just saw that values bound with `const` use lexical scope, just like values bound with parameters. They are looked up in the environment where they are declared. And we know that functions create environments. Parameters are declared when we create functions, so it makes sense that parameters are bound to environments created when we invoke functions.

But `const` statements can appear inside blocks, and we saw that blocks can appear inside of other blocks, including function bodies. So where are `const` variables bound? In the function environment? Or in an environment corresponding to the block?

We can test this by creating another conflict. But instead of binding two different variables to the same name in two different places, we'll bind two different values to the same name, but one environment will be completely enclosed by the other.

Let's start, as above, by doing this with parameters. We'll start with:
```javascript
    ((PI) =>
      (diameter) => diameter * PI
    )(3.14159265)
```
And gratuitously wrap it in another IIFE so that we can bind `PI` to something else:
```javascript
    ((PI) =>
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )(3)
```   
This still evaluates to a function that calculates diameters:
```javascript
    ((PI) =>
      ((PI) =>
        (diameter) => diameter * PI
      )(3.14159265)
    )(3)(2)
      //=> 6.2831853
```      
And we can see that our `diameter * PI` expression uses the binding for `PI` in the closest parent environment.  but one question: Did binding `3.14159265` to `PI` somehow change the binding in the "outer" environment? Let's rewrite things slightly differently:
```javascript
    ((PI) => {
      ((PI) => {})(3);
      
      return (diameter) => diameter * PI;
    })(3.14159265)
```    
Now we bind `3` to `PI` in an otherwise empty IIFE inside of our IIFE that binds `3.14159265` to `PI`. Does that binding "overwrite" the outer one? Will our function return `6` or `6.2831853`? This is a book, you've already scanned ahead, so you know that the answer is **no**, the inner binding does not overwrite the outer binding:
```javascript
    ((PI) => {
      ((PI) => {})(3);
      
      return (diameter) => diameter * PI;
    })(3.14159265)(2)
      //=> 6.2831853
```      
We say that when we bind a variable using a parameter inside another binding, the inner binding *shadows* the outer binding. It has effect inside its own scope, but does not affect the binding in the enclosing scope.

So what about `const`. Does it work the same way?
```javascript
    ((diameter) => {
      const PI = 3.14159265;
      
      (() => {
        const PI = 3;
      })();
      
      return diameter * PI;
    })(2)
      //=> 6.2831853
```
Yes, names bound with `const` shadow enclosing bindings just like parameters. But wait! There's more!!!

Parameters are only bound when we invoke a function. That's why we made all these IIFEs. But `const` statements can appear inside blocks. What happens when we use a `const` inside of a block?

We'll need a gratuitous block. We've seen `if` statements, what could be more gratuitous than:
```javascript
    if (true) {
      // an immediately invoked block statement (IIBS)
    }
```    
Let's try it:
```javascript
    ((diameter) => {
      const PI = 3;
      
      if (true) {
        const PI = 3.14159265;
      
        return diameter * PI;
      }
    })(2)
      //=> 6.2831853

    ((diameter) => {
      const PI = 3.14159265;
      
      if (true) {
        const PI = 3;
      }
      return diameter * PI;
    })(2)
      //=> 6.2831853
```      
Ah! `const` statements don't just shadow values bound within the environments created by functions, they shadow values bound within environments created by blocks!

This is enormously important. Consider the alternative: What if `const` could be declared inside of a block, but it always bound the name in the function's scope. In that case, we'd see things like this:
```javascript
    ((diameter) => {
      const PI = 3.14159265;
      
      if (true) {
        const PI = 3;
      }
      return diameter * PI;
    })(2)
      //=> would return 6 if const had function scope
```      
If `const` always bound its value to the name defined in the function's environment, placing a `const` statement inside of a block would merely rebind the existing name, overwriting its old contents. That would be super-confusing. And this code would "work:"
```javascript
    ((diameter) => {
      if (true) {
        const PI = 3.14159265;
      }
      return diameter * PI;
    })(2)
      //=> would return 6.2831853 if const had function scope
```
Again, confusing. Typically, we want to bind our names as close to where we need them as possible. This design rule is called the [Principle of Least Privilege][plp], and it has both quality and security implications. Being able to bind a name inside of a block means that if the name is only needed in the block, we are not "leaking" its binding to other parts of the code that do not need to interact with it.

[plp]: https://en.wikipedia.org/wiki/Principle_of_least_privilege
    
### rebinding {#rebinding-peek}

By default, JavaScript permits us to *rebind* new values to names bound with a parameter. For example, we can write:
```javascript
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
```javascript
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

This is valuable, as it greatly simplifies the analysis of programs to see at a glance that when something is bound with `const`, we need never worry that its value may change.
## Naming Functions {#named-function-expressions}

Let's get right to it. This code does *not* name a function:
```javascript
    const repeat = (str) => str + str
```
It doesn't name the function "repeat" for the same reason that `const answer = 42` doesn't name the number `42`. This syntax binds an anonymous function to a name in an environment, but the function itself remains anonymous.

### the `function` keyword

JavaScript *does* have a syntax for naming a function, we use the `function` keyword. Until ECMAScript 2015 was created, `function` was the usual syntax for writing functions.

Here's our `repeat` function written using a "fat arrow"
```javascript
    (str) => str + str
```
And here's (almost) the exact same function written using the `function` keyword:
```javascript
    function (str) { return str + str }
```
Let's look at the obvious differences:

1. We introduce a function with the `function` keyword.
1. Something else we're about to discuss is optional.
1. We have arguments in parentheses, just like fat arrow functions.
1. We do not have a fat arrow, we go directly to the body.
1. We always use a block, we cannot write `function (str) str + str`. This means that if we want our functions to return a value, we always need to use the `return` keyword

If we leave out the "something optional" that comes after the `function` keyword, we can translate all of the fat arrow functions that we've seen into `function` keyword functions, e.g.
```javascript
    (n) => (1.618**n - -1.618**-n) / 2.236
```
Can be written as:
```javascript
    function (n) {
      return (1.618**n - -1.618**-n) / 2.236;
    }
```
This still does not *name* a function, but as we noted above, functions written with the `function` keyword have an optional "something else." Could that "something else" name a function? Yes, of course.[^ofcourse]

[^ofcourse]: "Yes of course?" Well, in chapter of a book dedicated to naming functions, it is not surprising that feature we mention has something to do with naming functions.

Here are our example functions written with names:
```javascript
    const repeat = function repeat (str) {
      return str + str;
    };

    const fib = function fib (n) {
      return (1.618**n - -1.618**-n) / 2.236;
    };
```
Placing a name between the `function` keyword and the argument list names the function. Confusingly, the name of the function is not exactly the same thing as the name we may choose to bind to the value of the function. For example, we can write:
```javascript
    const double = function repeat (str) {
      return str + str;
    }

In this expression, `double` is the name in the environment, but `repeat` is the function's actual name. This is a *named function expression*. That may seem confusing, but think of the binding names as properties of the environment, not of the function. While the name of the function is a property of the function, not of the environment.

And indeed the name *is* a property:
```javascript
    double.name
      //=> 'repeat'
```
In this book we are not examining JavaScript's tooling such as debuggers baked into browsers, but we will note that when you are navigating call stacks in all modern tools, the function's binding name is ignored but its actual name is displayed, so naming functions is very useful even if they don't get a formal binding, e.g.
```javascript
    someBackboneView.on('click', function clickHandler () {
      //...
    });
```
Now, the function's actual name has no effect on the environment in which it is used. To whit:
```javascript
    const bindingName = function actualName () {
      //...
    };

    bindingName
      //=> [Function: actualName]

    actualName
      //=> ReferenceError: actualName is not defined
```
So "actualName" isn't bound in the environment where we use the named function expression. Is it bound anywhere else? Yes it is. Here's a function that determines whether a positive integer is even or not. We'll use it in an IIFE so that we don't have to bind it to a name with `const`:
```javascript
    (function even (n) {
      if (n === 0) {
        return true
      }
      else return !even(n - 1)
    })(5)
      //=> false

    (function even (n) {
      if (n === 0) {
        return true
      }
      else return !even(n - 1)
    })(2)
      //=> true
```
Clearly, the name `even` is bound to the function *within the function's body*. Is it bound to the function outside of the function's body?
```javascript
    even
      //=> Can't find variable: even
```
`even` is bound within the function itself, but not outside it. This is useful for making recursive functions as we see above, and it speaks to the principle of least privilege: If you don't *need* to name it anywhere else, you needn't.

### function declarations {#function-declarations}

There is another syntax for naming and/or defining a function. It's called a *function declaration statement*, and it looks a lot like a named function expression, only we use it as a statement:
```javascript
    function someName () {
      // ...
    }
```
This behaves a *little* like:
```javascript
    const someName = function someName () {
      // ...
    }
```
In that it binds a name in the environment to a named function. However, there are two important differences. First, function declarations are *hoisted* to the top of the function in which they occur.

Consider this example where we try to use the variable `fizzbuzz` as a function before we bind a function to it with `const`:
```javascript
    (function () {
      return fizzbuzz();

      const fizzbuzz = function fizzbuzz () {
        return "Fizz" + "Buzz";
      }
    })()
      //=> undefined is not a function (evaluating 'fizzbuzz()')
```
We haven't actually bound a function to the name `fizzbuzz` before we try to use it, so we get an error. But a function *declaration* works differently:
```javascript
    (function () {
      return fizzbuzz();

      function fizzbuzz () {
        return "Fizz" + "Buzz";
      }
    })()
      //=> 'FizzBuzz'
```
Although `fizzbuzz` is declared later in the function, JavaScript behaves as if we'd written:
```javascript
    (function () {
      const fizzbuzz = function fizzbuzz () {
        return "Fizz" + "Buzz";
      }

      return fizzbuzz();
    })()
```
The definition of the `fizzbuzz` is "hoisted" to the top of its enclosing scope (an IIFE in this case). This behaviour is intentional on the part of JavaScript's design to facilitate a certain style of programming where you put the main logic up front, and the "helper functions" at the bottom. It is not necessary to declare functions in this way in JavaScript, but understanding the syntax and its behaviour (especially the way it differs from `const`) is essential for working with production code.

### function declaration caveats[^caveats]

Function declarations are formally only supposed to be made at what we might call the "top level" of a function. Although some JavaScript environments permit the following code, this example is technically illegal and definitely a bad idea:
```javascript
    (function (camelCase) {
      return fizzbuzz();

      if (camelCase) {
        function fizzbuzz () {
          return "Fizz" + "Buzz";
        }
      }
      else {
        function fizzbuzz () {
          return "Fizz" + "Buzz";
        }
      }
    })(true)
      //=> 'FizzBuzz'? Or ERROR: Can't find variable: fizzbuzz?
```
Function declarations are not supposed to occur inside of blocks. The big trouble with expressions like this is that they may work just fine in your test environment but work a different way in production. Or it may work one way today and a different way when the JavaScript engine is updated, say with a new optimization.

Another caveat is that a function declaration cannot exist inside of *any* expression, otherwise it's a function expression. So this is a function declaration:
```javascript
    function trueDat () { return true }
```
But this is not:
```javascript
    (function trueDat () { return true })
```
The parentheses make this an expression, not a function declaration.

[^caveats]: A number of the caveats discussed here were described in Jyrly Zaytsev's excellent article [Named function expressions demystified](http://kangax.github.com/nfe/).
## Combinators and Function Decorators {#combinators}

### higher-order functions

As we've seen, JavaScript functions take values as arguments and return values. JavaScript functions are values, so JavaScript functions can take functions as arguments, return functions, or both. Generally speaking, a function that either takes functions as arguments, or returns a function, or both, is referred to as a "higher-order" function.

Here's a very simple higher-order function that takes a function as an argument:
```javascript
    const repeat = (num, fn) =>
      (num > 0)
        ? (repeat(num - 1, fn), fn(num))
        : undefined

    repeat(3, function (n) {
      console.log(`Hello ${n}`)
    })
      //=>
        'Hello 1'
        'Hello 2'
        'Hello 3'
        undefined
```
Higher-order functions dominate *JavaScript Allongé*. But before we go on, we'll talk about some specific types of higher-order functions.

### combinators

The word "combinator" has a precise technical meaning in mathematics:

> "A combinator is a higher-order function that uses only function application and earlier defined combinators to define a result from its arguments."--[Wikipedia][combinators]

[combinators]: https://en.wikipedia.org/wiki/Combinatory_logic "Combinatory Logic"

If we were learning Combinatorial Logic, we'd start with the most basic combinators like `S`, `K`, and `I`, and work up from there to practical combinators. We'd learn that the fundamental combinators are named after birds following the example of Raymond Smullyan's famous book [To Mock a Mockingbird][mock].

[mock]: http://www.amazon.com/gp/product/B00A1P096Y/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00A1P096Y&linkCode=as2&tag=raganwald001-20

In this book, we will be using a looser definition of "combinator:" Higher-order pure functions that take only functions as arguments and return a function. We won't be strict about using only previously defined combinators in their construction.

Let's start with a useful combinator: Most programmers call it *Compose*, although the logicians call it the B combinator or "Bluebird." Here is the typical[^bluebird] programming implementation:
```javascript
    const compose = (a, b) =>
      (c) => a(b(c))
```
Let's say we have:
```javascript
    const addOne = (number) => number + 1;

    const doubleOf = (number) => number * 2;
```
With `compose`, anywhere you would write
```javascript
    const doubleOfAddOne = (number) => doubleOf(addOne(number));
```
You could also write:
```javascript
    const doubleOfAddOne = compose(doubleOf, addOne);
```
This is, of course, just one example of many. You'll find lots more perusing the recipes in this book. While some programmers believe "There Should Only Be One Way To Do It," having combinators available as well as explicitly writing things out with lots of symbols and keywords has some advantages when used judiciously.

### a balanced statement about combinators

Code that uses a lot of combinators tends to name the verbs and adverbs (like `doubleOf`, `addOne`, and `compose`) while avoiding language keywords and the names of nouns (like `number`). So one perspective is that combinators are useful when you want to emphasize what you're doing and how it fits together, and more explicit code is useful when you want to emphasize what you're working with.

### function decorators {#decorators}

A *function decorator* is a higher-order function that takes one function as an argument, returns another function, and the returned function is a variation of the argument function. Here's a ridiculously simple  decorator:[^variadic]
```javascript
    const not = (fn) => (x) => !fn(x)
```
[^variadic]: We'll see later why an even more useful version would be written `(fn) => (...args) => !fn(...args)`

So instead of writing `!someFunction(42)`, we can write `not(someFunction)(42)`. Hardly progress. But like `compose`, we could write either:
```javascript
    const something = (x) => x != null;
```
And elsewhere, write:
```javascript
    const nothing = (x) => !something(x);
```
Or we could write:
```javascript
    const nothing = not(something);
```
`not` is a function decorator because it modifies a function while remaining strongly related to the original function's semantics. You'll see other function decorators in the recipes, like [once](#once) and [maybe](#maybe). Function decorators aren't strict about being pure functions, so there's more latitude for making decorators than combinators.

[^bluebird]: As we'll discuss later, this implementation of the B Combinator is correct in languages like Scheme, but for truly general-purpose use in JavaScript, it needs to correctly manage the [function context](#context).
## Building Blocks {#buildingblocks}

When you look at functions within functions in JavaScript, there's a bit of a "spaghetti code" look to it. The strength of JavaScript is that you can do anything. The weakness is that you will. There are ifs, fors, returns, everything thrown higgledy piggledy together. Although you needn't restrict yourself to a small number of simple patterns, it can be helpful to understand the patterns so that you can structure your code around some basic building blocks.

### composition

One of the most basic of these building blocks is *composition*:
```javascript
    const cookAndEat = (food) => eat(cook(food));
```    
It's really that simple: Whenever you are chaining two or more functions together, you're composing them. You can compose them with explicit JavaScript code as we've just done. You can also generalize composition with the B Combinator or "compose" that we saw in [Combinators and Decorators](#combinators):
```javascript
    const compose = (a, b) => (c) => a(b(c));

    const cookAndEat = compose(eat, cook);
```    
If that was all there was to it, composition wouldn't matter much. But like many patterns, using it when it applies is only 20% of the benefit. The other 80% comes from organizing your code such that you can use it: Writing functions that can be composed in various ways.

In the recipes, we'll look at a decorator called  [once](#once): It ensures that a function can only be executed once. Thereafter, it does nothing. Once is useful for ensuring that certain side effects are not repeated. We'll also look at [maybe](#maybe): It ensures that a function does nothing if it is given nothing (like `null` or `undefined`) as an argument.

Of course, you needn't use combinators to implement either of these ideas, you can use if statements. But `once` and `maybe` compose, so you can chain them together as you see fit:
```javascript
    const actuallyTransfer= (from, to, amount) =>
      // do something
    
    const invokeTransfer = once(maybe(actuallyTransfer(...)));
```    
### partial application

Another basic building block is *partial application*. When a function takes multiple arguments, we "apply" the function to the arguments by evaluating it with all of the arguments, producing a value. But what if we only supply some of the arguments? In that case, we can't get the final value, but we can get a function that represents *part* of our application.

Code is easier than words for this. The [Underscore] library provides a higher-order function called *map*.[^headache] It applies another function to each element of an array, like this:
```javascript
    _.map([1, 2, 3], (n) => n * n)
      //=> [1, 4, 9]
```
We don't want to fool around writing `_.`, so we can use it by writing:[^_map]

[^_map]: If we don't want to sort out [Underscore], we can also write the following: `const map = (a, fn) => a.map(fn);`, and trust that it works even though we haven't discussed methods yet.
```javascript
    const map = _.map;
```      
This code implements a partial application of the map function by applying the function `(n) => n * n` as its second argument:
```javascript
    const squareAll = (array) => map(array,  (n) => n * n);
```
The resulting function--`squareAll`--is still the map function, it's just that we've applied one of its two arguments already. `squareAll` is nice, but why write one function every time we want to partially apply a function to a map? We can abstract this one level higher. `mapWith` takes any function as an argument and returns a partially applied map function.
```javascript
    const mapWith = (fn) =>
      (array) => map(array, fn);
    
    const squareAll = mapWith((n) => n * n);
    
    squareAll([1, 2, 3])
      //=> [1, 4, 9]
```
We'll discuss `mapWith` again. The important thing to see is that partial application is orthogonal to composition, and that they both work together nicely:
```javascript
    const safeSquareAll = mapWith(maybe((n) => n * n));
    
    safeSquareAll([1, null, 2, 3])
      //=> [1, null, 4, 9]
```
We generalized composition with the `compose` combinator. Partial application also has a combinator, which we'll see in the [partial](#partial) recipe.

[^bind]: Modern JavaScript provides a limited form of partial application through the `Function.prototype.bind` method. This will be discussed in greater length when we look at function contexts.

[^headache]: Modern JavaScript implementations provide a map method for arrays, but Underscore's implementation also works with older browsers if you are working with that headache.

[Underscore]: http://underscorejs.org
## Magic Names {#magic-names}

When a function is applied to arguments (or "called"), JavaScript binds the values of arguments to the function's argument names in an environment created for the function's execution. What we haven't discussed so far is that JavaScript also binds values to some "magic" names in addition to any you put in the argument list.[^read-only]

[^read-only]: You should never attempt to define your own bindings against "magic" names that JavaScript binds for you. It is wise to treat them as read-only at all times.

### the function keyword

There are two separate rules for these "magic" names, one for when you invoke a function using the `function` keyword, and another for functions defined with "fat arrows." We'll begin with how things work for functions defined with the `function` keyword.

The first magic name is  `this`, and it is bound to something called the function's [context](#context). We will explore `this` in more detail when we start discussing objects and classes. The second magic name is very interesting, it's called `arguments`, and the most interesting thing about it is that it contains a list of arguments passed to a function:

{:lang="js"}
~~~~~~~~
const plus = function (a, b) {
  return arguments[0] + arguments[1];
}

plus(2,3)
  //=> 5
~~~~~~~~

Although `arguments` looks like an array, it isn't an array: It's more like an object[^pojo] that happens to bind some values to properties with names that look like integers starting with zero:

{:lang="js"}
~~~~~~~~
const args = function (a, b) {
  return arguments;
}

args(2,3)
  //=> { '0': 2, '1': 3 }
~~~~~~~~

`arguments` always contains all of the arguments passed to a function, regardless of how many are declared. Therefore, we can write `plus` like this:

{:lang="js"}
~~~~~~~~
const plus = function () {
  return arguments[0] + arguments[1];
}

plus(2,3)
  //=> 5
~~~~~~~~

When discussing objects, we'll discuss properties in more depth. Here's something interesting about `arguments`:

{:lang="js"}
~~~~~~~~
const howMany = function () {
  return arguments['length'];
}

howMany()
  //=> 0

howMany('hello')
  //=> 1

howMany('sharks', 'are', 'apex', 'predators')
  //=> 4
~~~~~~~~

The most common use of the `arguments` binding is to build functions that can take a variable number of arguments. We'll see it used in many of the recipes, starting off with [partial application](#simple-partial) and [ellipses](#ellipses).

[^pojo]: We'll look at [arrays](#arrays) and [plain old javascript objects](#pojos) in depth later.

### magic names and fat arrows

The magic names `this` and `arguments` have a different behaviour when you invoke a function that was defined with a fat arrow: Instead of being bound when the function is invoked, the fat arrow function always acquires the bindings for `this` and `arguments` from its enclosing scope, just like any other binding.

For example, when this expression's inner function is defined with `function`, `arguments[0]` refers to its only argument, `"inner"`:

```js
(function () {
  return (function () { return arguments[0]; })('inner');
})('outer')
  //=> "inner"
```

But if we use a fat arrow, `arguments` will be defined in the outer environment, the one defined with `function`. And thus `arguments[0]` will refer to `"outer"`, not to `"inner"`:

```js
(function () {
  return (() => arguments[0])('inner');
})('outer')
  //=> "outer"
```

Although it seems quixotic for the two syntaxes to have different semantics, it makes sense when you consider the design goal: Fat arrow functions are designed to be very lightweight and are often used with constructs like mapping or callbacks to emulate syntax.

To give a contrived example, this function takes a number and returns an array representing a row in a hypothetical multiplication table. It uses `mapWith`, which we discussed in [Building Blocks](#buildingblocks).[^mapWith] We'll use `arguments` just to show the difference between using a fat arrow and the function keyword:

[^mapWith]: We can also write the following: `const mapWith = (fn) => array => array.map(fn);`, and trust that it works even though we haven't discussed methods yet.

```js
const row = function () {
  return mapWith((column) => column * arguments[0])(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  )
}

row(3)
  //=> [3,6,9,12,15,18,21,24,27,30,33,36
```

This works just fine, because `arguments[0]` refers to the `3` we passed to the function `row`. Our "fat arrow" function `(column) => column * arguments[0]` doesn't bind `arguments` when it's invoked. But if we rewrite `row` to use the `function` keyword, it stops working:

```js
const row = function () {
  return mapWith(function (column) { return column * arguments[0] })(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
  )
}

row(3)
  //=> [1,4,9,16,25,36,49,64,81,100,121,144]
```

Now our inner function binds `arguments[0]` every time it is invoked, so we get the same result as if we'd written
`function (column) { return column * column }`.

Although this example is clearly unrealistic, there is a general design principle that deserves attention. Sometimes, a function is meant to be used as a Big-F function. It has a name, it is called by different pieces of code, it's a first-class entity in the code.

But sometimes, a function is a small-f function. It's a simple representation of an expression to be computed. In our example above, `row` is a Big-F function, but `(column) => column * arguments[0]` is a small-f function, it exists just to give `mapWith` something to apply.

Having magic variables apply to Big-F functions but not to small-G functions makes it much easier to use small-F functions as syntax, treating them as expressions or blocks that can be passed to functions like `mapWith`.
## Summary

> ### Functions
>
> * Functions are values that can be part of expressions, returned from other functions, and so forth.
> * Functions are *reference values*.
> * Functions are applied to arguments.
> * The arguments are passed by sharing, which is also called "pass by value."
> * Fat arrow functions have expressions or blocks as their bodies.
> * `function` keyword functions always have blocks as their bodies.
> * Function bodies have zero or more statements.
> * Expression bodies evaluate to the value of the expression.
> * Block bodies evaluate to whatever is returned with the `return` keyword, or to `undefined`.
> * JavaScript uses `const` to bind values to names within block scope.
> * JavaScript uses function declarations to bind functions to names within function scope. Function declarations are "hoisted."
> * Function application creates a scope.
> * Blocks also create scopes if `const` statements are within them.
> * Scopes are nested and free variable references closed over.
> * Variables can shadow variables in an enclosing scope.
