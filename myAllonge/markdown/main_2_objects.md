# Stir the Allongé: Objects and State

So far, we have discussed what many call "pure functional" programming, where every expression is necessarily [idempotent], because we have no way of changing state within a program using the tools we have examined.

We've also explored functions that rebind names within themselves as part of performing their calculations. And we briefly touched upon the notion of mutating an object as part of building it. But we have avoided objects that are meant to be changed, objects that model *state*.

[idempotent]: https://en.wikipedia.org/wiki/Idempotence

It's time to change *everything*.
## Encapsulating State with Closures

> OOP to me means only messaging, local retention and protection and hiding of state-process, and extreme late-binding of all things.--[Alan Kay][oop]

[oop]: http://userpage.fu-berlin.de/~ram/pub/pub_jf47ht81Ht/doc_kay_oop_en

We're going to look at encapsulation using JavaScript's functions and objects. We're not going to call it object-oriented programming, mind you, because that would start a long debate. This is just plain encapsulation,`1` with a dash of information-hiding.

>`1` "A language construct that facilitates the bundling of data with the methods (or other functions) operating on that data."--[Wikipedia]

[Wikipedia]: https://en.wikipedia.org/wiki/Encapsulation_(object-oriented_programming)

### what is hiding of state-process, and why does it matter?

> In computer science, information hiding is the principle of segregation of the design decisions in a computer program that are most likely to change, thus protecting other parts of the program from extensive modification if the design decision is changed. The protection involves providing a stable interface which protects the remainder of the program from the implementation (the details that are most likely to change).
>
> Written another way, information hiding is the ability to prevent certain aspects of a class or software component from being accessible to its clients, using either programming language features (like private variables) or an explicit exporting policy. -- [Wikipedia][ih]

[ih]:https://en.wikipedia.org/wiki/Information_hiding "Information hiding"

Consider a [stack] data structure. There are three basic operations: Pushing a value onto the top (`push`), popping a value off the top (`pop`), and testing to see whether the stack is empty or not (`isEmpty`). These three operations are the stable interface.

[stack]: https://en.wikipedia.org/wiki/Stack_(data_structure)

Many stacks have an array for holding the contents of the stack. This is relatively stable. You could substitute a linked list, but in JavaScript, the array is highly efficient. You might need an index, you might not. You could grow and shrink the array, or you could allocate a fixed size and use an index to keep track of how much of the array is in use. The design choices for keeping track of the head of the list are often driven by performance considerations.

If you expose the implementation detail such as whether there is an index, sooner or later some programmer is going to find an advantage in using the index directly. For example, she may need to know the size of a stack. The ideal choice would be to add a `size` function that continues to hide the implementation. But she's in a hurry, so she reads the `index` directly. Now her code is coupled to the existence of an index, so if we wish to change the implementation to grow and shrink the array, we will break her code.

The way to avoid this is to hide the array and index from other code and only expose the operations we have deemed stable. If and when someone needs to know the size of the stack, we'll add a `size` function and expose it as well.

Hiding information (or "state") is the design principle that allows us to limit the coupling between components of software.

### how do we hide state using javascript?

We've been introduced to JavaScript's objects, and it's fairly easy to see that objects can be used to model what other programming languages call (variously) records, structs, frames, or what-have-you. And given that their elements are mutable, they can clearly model state.

Given an object that holds our state (an array and an index `1`), we can easily implement our three operations as functions. Bundling the functions with the state does not require any special "magic" features. JavaScript objects can have elements of any type, including functions.

>`1` Yes, there's another way to track the size of the array, but we don't need it to demonstrate encapsulation and hiding of state.

To make our stack work, we need a way for our functions to refer to our stack. We'll do that by making sure it has a name. We can do that with an IIFE:
```js
    const stack = (() => {
      const obj = {
        array: [],
        index: -1,
        push (value) {
          return obj.array[obj.index += 1] = value
        },
        pop () {
          const value = obj.array[obj.index];

          obj.array[obj.index] = undefined;
          if (obj.index >= 0) {
            obj.index -= 1
          }
          return value
        },
        isEmpty () {
          return obj.index < 0
        }
      };

      return obj;
    })();

    stack.isEmpty()
      //=> true
    stack.push('hello')
      //=> 'hello'
    stack.push('JavaScript')
     //=> 'JavaScript'
    stack.isEmpty()
      //=> false
    stack.pop()
     //=> 'JavaScript'
    stack.pop()
     //=> 'hello'
    stack.isEmpty()
      //=> true
```
### method-ology

In this text, we lurch from talking about "functions that belong to an object" to "methods." Other languages may separate methods from functions very strictly, but in JavaScript every method is a function, but not all functions are methods.

The view taken in this book is that a function is a method of an object if it belongs to that object and interacts with that object in some way. So the functions implementing the operations on the stack are all absolutely methods of the stack.

But these two wouldn't be methods. Although they "belong" to an object, they don't interact with it:
```js
    {
      min: (x, y) =>
        x < y ? x : y
      max: (x, y) =>
        x > y ? x : y
    }
```
### hiding state

Our stack does bundle functions with data, but it doesn't hide its state. "Foreign" code could interfere with its array or index. So how do we hide these? We already have a closure, let's use it:
```js
    const stack = (() => {
      let array = [],
          index = -1;

      const obj = {
        push (value) { return array[index += 1] = value },
        pop () {
          const value = array[index];

          array[index] = undefined;
          if (index >= 0) {
            index -= 1
          }
          return value
        },
        isEmpty () { return index < 0 }
      };

      return obj;
    })();

    stack.isEmpty()
      //=> true
    stack.push('hello')
      //=> 'hello'
    stack.push('JavaScript')
     //=> 'JavaScript'
    stack.isEmpty()
      //=> false
    stack.pop()
     //=> 'JavaScript'
    stack.pop()
     //=> 'hello'
    stack.isEmpty()
      //=> true
```
We don't want to repeat this code every time we want a stack, so let's make ourselves a "stack maker." The temptation is to wrap what we have above in a function:
```js
    const Stack = () =>
      (() => {
        let array = [],
            index = -1;

        const obj = {
          push (value) { return array[index += 1] = value },
          pop () {
            const value = array[index];

            array[index] = undefined;
            if (index >= 0) {
              index -= 1
            }
            return value
          },
          isEmpty () { return index < 0 }
        };

        return obj;
      })();
```
But there's an easier way :-)
```js
    const Stack = () => {
      const array = [];
      let index = -1;

      return {
        push (value) { return array[index += 1] = value },
        pop () {
          const value = array[index];

          array[index] = undefined;
          if (index >= 0) {
            index -= 1
          }
          return value
        },
        isEmpty () { return index < 0 }
      }
    }

    const stack = Stack();
    stack.push("Hello");
    stack.push("Good bye");

    stack.pop()
      //=> "Good bye"
    stack.pop()
      //=> "Hello"
```
Now we can make stacks freely, and we've hidden their internal data elements. We have methods and encapsulation, and we've built them out of JavaScript's fundamental functions and objects. In [Constructors and Classes](main_5_constructors.md), we'll look at JavaScript's support for class-oriented programming and some of the idioms that functions bring to the party.

> ### is encapsulation "object-oriented?"
>
> We've built something with hidden internal state and "methods," all without needing special `def` or `private` keywords. Mind you, we haven't included all sorts of complicated mechanisms to support inheritance, mixins, and other opportunities for debating the nature of the One True Object-Oriented Style on the Internet.
>
> Then again, the key lesson experienced programmers repeat--although it often falls on deaf ears--is [Composition instead of Inheritance](http://www.c2.com/cgi/wiki?CompositionInsteadOfInheritance). So maybe we aren't missing much.

## Composition and Extension 
### composition

A deeply fundamental practice is to build components out of smaller components. The choice of how to divide a component into smaller components is called *factoring*, after the operation in number theory `1`.

>`1` And when you take an already factored component and rearrange things so that it is factored into a different set of subcomponents without altering its behaviour, you are *refactoring*.

The simplest and easiest way to build components out of smaller components in JavaScript is also the most obvious: Each component is a value, and the components can be put together into a single object or encapsulated with a closure.

Here's an abstract "model" that supports undo and redo composed from a pair of stacks (see [Encapsulating State](#encapsulation)), and a Plain Old JavaScript Object:

We can `set` and `get` attributes on a model

```js
// helper function
//
// For production use, consider what to do about
// deep copies and own keys
const shallowCopy = (source) => {
  const dest = {};

  for (let key in source) {
    dest[key] = source[key]
  }
  return dest
};

const Stack = () => {
  const array = [];
  let index = -1;

  return {
    push (value) {
      array[index += 1] = value
    },
    pop () {
      let value = array[index];
      if (index >= 0) {
        index -= 1
      }
      return value
    },
    isEmpty () {
      return index < 0
    }
  }
}

const Model = function (initialAttributes) {
  const redoStack = Stack();
  let attributes = shallowCopy(initialAttributes || {});

  const undoStack = Stack(),
      obj = {
        set: (attrsToSet) => {
          undoStack.push(shallowCopy(attributes));
          if (!redoStack.isEmpty()) {
            redoStack.length = 0;
          }
          for (let key in (attrsToSet || {})) {
            attributes[key] = attrsToSet[key]
          }
          return obj
        },
        undo: () => {
          if (!undoStack.isEmpty()) {
            redoStack.push(shallowCopy(attributes));
            attributes = undoStack.pop()
          }
          return obj
        },
        redo: () => {
          if (!redoStack.isEmpty()) {
            undoStack.push(shallowCopy(attributes));
            attributes = redoStack.pop()
          }
          return obj
        },
        get: (key) => attributes[key],
        has: (key) => attributes.hasOwnProperty(key),
        attributes: () => shallowCopy(attributes)
      };
    return obj
  };

const model = Model();
model.set({"Doctor": "de Grasse"});
model.set({"Doctor": "Who"});
model.undo()
model.get("Doctor")
  //=> "de Grasse"
```
The techniques used for encapsulation work well with composition. In this case, we have a "model" that hides its attribute store as well as its implementation that is composed of an undo stack and redo stack.

### extension

Another practice that many people consider fundamental is to *extend* an implementation. Meaning, they wish to define a new data structure in terms of adding new operations and semantics to an existing data structure.

Consider a [queue]:
```js
    const Queue = () => {
      let array = [],
          head = 0,
          tail = -1;

      return {
        pushTail: (value) => array[++tail] = value,
        pullHead: () => {
          if (tail >= head) {
            const value = array[head];

            array[head] = undefined;
            ++head;
            return value
          }
        },
        isEmpty: () => tail < head
      }
    };


    const queue = Queue();
    queue.pushTail("Hello");
    queue.pushTail("JavaScript");
    queue.pushTail("Allongé");

    queue.pullHead()
      //=> "Hello"
    queue.pullHead()
      //=> "JavaScript"
```
Now we wish to create a [deque] by adding `pullTail` and `pushHead` operations to our queue.`1` Unfortunately, encapsulation prevents us from adding operations that interact with the hidden data structures.

[queue]: http://duckduckgo.com/Queue_(data_structure)
[deque]: https://en.wikipedia.org/wiki/Double-ended_queue "Double-ended queue"
>`1` Before you start wondering whether a deque is-a queue, we said nothing about types and classes. This relationship is called was-a, or "implemented in terms of a."

This isn't really surprising: The entire point of encapsulation is to create an opaque data structure that can only be manipulated through its public interface. The design goals of encapsulation and extension are always going to exist in tension.

Let's "de-encapsulate" our queue:
```js
    const Queue = function () {
      const queue = {
        array: [],
        head: 0,
        tail: -1,
        pushTail: (value) =>
          queue.array[++queue.tail] = value,
        pullHead: () => {
          if (queue.tail >= queue.head) {
            const value = queue.array[queue.head];

            queue.array[queue.head] = undefined;
            queue.head += 1;
            return value
          }
        },
        isEmpty: () =>
          queue.tail < queue.head
      };
      return queue
    };
```
Now we can extend a queue into a deque:
```js
    const extend = function (consumer, ...providers) {
      for (let i = 0; i < providers.length; ++i) {
        const provider = providers[i];
        for (let key in provider) {
          if (provider.hasOwnProperty(key)) {
            consumer[key] = provider[key]
          }
        }
      }
      return consumer
    };

    const Dequeue = function () {
      const deque = Queue(),
          INCREMENT = 4;

      return Object.assign(deque, {
        size: () => deque.tail - deque.head + 1,
        pullTail: () => {
          if (!deque.isEmpty()) {
            const value = deque.array[deque.tail];

            deque.array[deque.tail] = undefined;
            deque.tail -= 1;
            return value
          }
        },
        pushHead: (value) => {
          if (deque.head === 0) {
            for (let i = deque.tail; i <= deque.head; i++) {
              deque.array[i + INCREMENT] = deque.array[i]
            }
            deque.tail += INCREMENT
            deque.head += INCREMENT
          }
          return deque.array[deque.head -= 1] = value
        }
      })
    };
```
Presto, we have reuse through extension, at the cost of encapsulation.

> Encapsulation and Extension exist in a natural state of tension. A program with elaborate encapsulation resists breakage but can also be difficult to refactor in other ways. Be mindful of when it's best to Compose and when it's best to Extend.

## This and That

Let's take another look at [extensible objects](#extension). Here's a Queue:
```js
    const Queue = () => {
      const queue = {
        array: [],
        head: 0,
        tail: -1,
        pushTail (value) {
          return queue.array[++queue.tail] = value
        },
        pullHead () {
          if (queue.tail >= queue.head) {
            const value = queue.array[queue.head];

            queue.array[queue.head] = undefined;
            queue.head += 1;
            return value
          }
        },
        isEmpty () {
          return queue.tail < queue.head;
        }
      };
      return queue
    };

    const queue = Queue();
    queue.pushTail('Hello');
    queue.pushTail('JavaScript');
```
Let's make a copy of our queue using `Object.assign`:
```js
    const copyOfQueue = Object.assign({}, queue);

    queue !== copyOfQueue
      //=> true
```
Wait a second. We know that array values are references. So it probably copied a reference to the original array. Let's make a copy of the array as well:
```js
    copyOfQueue.array = [];
    for (let i = 0; i < 2; ++i) {
      copyOfQueue.array[i] = queue.array[i]
    }
```
Now let's pull the head off the original:
```js
    queue.pullHead()
      //=> 'Hello'
```
If we've copied everything properly, we should get the exact same result when we pull the head off the copy:
```js
    copyOfQueue.pullHead()
      //=> 'JavaScript'
```
What!? Even though we carefully made a copy of the array to prevent aliasing, it seems that our two queues behave like aliases of each other. The problem is that while we've carefully copied our array and other elements over, *the closures all share the same environment*, and therefore the functions in `copyOfQueue` all operate on the first queue's private data, not on the copies.

> This is a general issue with closures. Closures couple functions to environments, and that makes them very elegant in the small, and very handy for making opaque data structures. Alas, their strength in the small is their weakness in the large. When you're trying to make reusable components, this coupling is sometimes a hindrance.

Let's take an impossibly optimistic flight of fancy:
```js
    const AmnesiacQueue = () =>
      ({
        array: [],
        head: 0,
        tail: -1,
        pushTail (myself, value) {
          return myself.array[myself.tail += 1] = value
        },
        pullHead (myself) {
          if (myself.tail >= myself.head) {
            let value = myself.array[myself.head];

            myself.array[myself.head] = void 0;
            myself.head += 1;
            return value
          }
        },
        isEmpty (myself) {
          return myself.tail < myself.head
        }
      });

    const queueWithAmnesia = AmnesiacQueue();

    queueWithAmnesia.pushTail(queueWithAmnesia, 'Hello');
    queueWithAmnesia.pushTail(queueWithAmnesia, 'JavaScript');
    queueWithAmnesia.pullHead(queueWithAmnesia)
      //=> "Hello"
```
The `AmnesiacQueue` makes queues with amnesia: They don't know who they are, so every time we invoke one of their functions, we have to tell them who they are. You can work out the implications for copying queues as a thought experiment: We don't have to worry about environments, because every function operates on the queue you pass in.

The killer drawback, of course, is making sure we are always passing the correct queue in every time we invoke a function. What to do?

### what's all `this`?

Any time we must do the same repetitive thing over and over and over again, we industrial humans try to build a machine to do it for us. JavaScript is one such machine. When we write a function expression using the compact method syntax (or use the `function` keyword instead of the fat arrow), and then invoke that function using `.` notation, JavaScript binds the "receiver" of a "method invocation" to the special name `this`.

Our `AmnesiacQueue` already uses compact method notation. So, we'll remove `myself` from the parameter list, and rename it to `this` within the body of each function:
```js
    const BetterQueue = () =>
      ({
        array: [],
        head: 0,
        tail: -1,
        pushTail (value) {
          return this.array[this.tail += 1] = value
        },
        pullHead () {
          if (this.tail >= this.head) {
            let value = this.array[this.head];

            this.array[this.head] = undefined;
            this.head += 1;
            return value
          }
        },
        isEmpty () {
          return this.tail < this.head
        }
      });
```
Now we are relying on JavaScript to set the value of `this` whenever we invoke one of these functions using the `.` or `[` and `]` operators.

In other words, when we write:
```js
    const betterQueue = BetterQueue();

    betterQueue.pushTail('Hello');
    betterQueue.pushTail('JavaScript');
    betterQueue.pullHead()
```
We expect that JavaScript will invoke the functions we've bound to `pushTail` and `pullHead`, and automatically bind `betterQueue` to the name `this` within them. And indeed it does: Every time you invoke a function that is a member of an object, JavaScript binds that object to the name `this` in the environment of the function just as if it was an argument.`1`

>`1` JavaScript also does other things with `this` as well, but this is all we care about right now.

Now, does this solve our original problem? Can we make copies of an object? Recall that the problem was that when we used a closure for private data, copying references to an object's functions meant that we were using functions that still referred to the original closure, and therefore shared the same private data.

Now our functions refer to members of the object, and use `this` to ensure  that they are referring to the object receiving a message. Let's see if this does, indeed, allow us to copy objects:
```js
    const copyOfQueue = Object.assign({}, betterQueue)
    copyOfQueue.array = []
    for (let i = 0; i < 2; ++i) {
      copyOfQueue.array[i] = betterQueue.array[i]
    }

    betterQueue.pullHead()
      //=> 'Hello'

    copyOfQueue.pullHead()
      //=> 'Hello'
```
Presto, we now have a way to copy arrays. By getting rid of the closure and taking advantage of `this`, we have functions that are more easily portable between objects, and the code is simpler as well. **This is very important**. Being able to copy objects is an example of a larger concern: Being able to share functions between objects. That's how classes work. That's how extending objects works. Being able to share functions means being able to compose and reuse functionality.

There is more to `this` than we've discussed here. We'll explore things in more detail later, in [What Context Applies When We Call a Function?](#context).

> Closures tightly couple functions to the environments where they are created limiting their flexibility. Using `this` alleviates the coupling. Copying objects is but one example of where that flexibility is needed.

## What Context Applies When We Call a Function?

In [This and That](#this), we learned that when a function is denoted using the `function` keyword, and is called as an object method, the name `this` is bound in its environment to the object acting as a "receiver." For example:
```js
    const someObject = {
      returnMyThis () {
        return this;
      }
    };
    
    someObject.returnMyThis() === someObject
      //=> true
```      
We've constructed a method that returns whatever value is bound to `this` when it is called. It returns the object when called, just as described.

### it's all about the way the function is called

JavaScript programmers talk about functions having a "context" when being called. `this` is bound to the context.`1` The important thing to understand is that the context for a function being called is set by the way the function is called, not the function itself.

>`1` Too bad the language binds the context to the name `this` instead of the name `context`!

This is an important distinction. Consider closures: As we discussed in [Closures and Scope](#closures), a function's free variables are resolved by looking them up in their enclosing functions' environments. You can always determine the functions that define free variables by examining the source code of a JavaScript program, which is why this scheme is known as [Lexical Scope].

[Lexical Scope]: https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping

A function's context cannot be determined by examining the source code of a JavaScript program. Let's look at our example again:
```js
    const someObject = {
      someFunction () {
        return this;
      }
    };

    someObject.someFunction() === someObject
      //=> true
```    
What is the context of the function `someObject.someFunction`? Don't say `someObject`! Watch this:
```js
    const someFunction = someObject.someFunction;

    someFunction === someObject.someFunction
      //=> true
    
    someFunction() === someObject
      //=> false
```      
It gets weirder:
```js
    const anotherObject = {
      someFunction: someObject.someFunction
    }
    
    anotherObject.someFunction === someObject.someFunction
      //=> true
      
    anotherObject.someFunction() === anotherObject
      //=> true
      
    anotherObject.someFunction() === someObject
      //=> false
```      
So it amounts to this: The exact same function can be called in two different ways, and you end up with two different contexts. If you call it using `someObject.someFunction()` syntax, the context is set to the receiver. If you call it using any other expression for resolving the function's value (such as `someFunction()`), you get something else.

Let's investigate:
```js
    (someObject.someFunction)() == someObject
      //=> true
      
    someObject['someFunction']() === someObject
      //=> true
      
    const name = 'someFunction';
    
    someObject[name]() === someObject
      //=> true
```
Interesting!
```js
    let baz;
    
    (baz = someObject.someFunction)() === this
      //=> true
```      
How about:
```js
    const arr = [ someObject.someFunction ];
    
    arr[0]() == arr
      //=> true
```    
It seems that whether you use `a.b()` or `a['b']()` or `a[n]()` or `(a.b)()`, you get context `a`. 
```js
    const returnThis = function () { return this };

    const aThirdObject = {
      someFunction () {
        return returnThis()
      }
    }
    
    returnThis() === this
      //=> true
    
    aThirdObject.someFunction() === this
      //=> true
```      
And if you don't use `a.b()` or `a['b']()` or `a[n]()` or `(a.b)()`, you get the global environment for a context, not the context of whatever function is doing the calling. To simplify things, when you call a function with `.` or `[]` access, you get an object as context, otherwise you get the global environment.

### setting your own context

There are actually two other ways to set the context of a function. And once again, both are determined by the caller. At the very end of [objects everywhere?](#pojoseverywhere), we'll see that everything in JavaScript behaves like an object, including functions. We'll learn that functions have methods themselves, and one of them is `call`.

Here's `call` in action:
```js
    returnThis() === aThirdObject
      //=> false

    returnThis.call(aThirdObject) === aThirdObject
      //=> true
      
    anotherObject.someFunction.call(someObject) === someObject
      //=> true
```      
When You call a function with `call`, you set the context by passing it in as the first parameter. Other arguments are passed to the function in the normal manner. Much hilarity can result from `call` shenanigans like this:
```js
    const a = [1,2,3],
        b = [4,5,6];
        
    a.concat([2,1])
      //=> [1,2,3,2,1]
      
    a.concat.call(b,[2,1])
      //=> [4,5,6,2,1]
```      
But now we thoroughly understand what `a.b()` really means: It's synonymous with `a.b.call(a)`. Whereas in a browser, `c()` is synonymous with `c.call(window)`.

### arguments

JavaScript has another automagic binding in every function's environment. `arguments` is a special object that behaves a little like an array.`1`

>`1` Just enough to be frustrating, to be perfectly candid!

For example:
```js
    const third = function () {
      return arguments[2]
    }

    third(77, 76, 75, 74, 73)
      //=> 75
```      
Gathering arguments with `...` accomplishes most of the use cases people have for using the `arguments` special binding, and in addition, gathering works with both fat arrows and with the `function` keyword, whereas `arguments` only works with the function keyword.

There are a few things that `arguments` can do that gathering cannot do, for example if you declare a function with `function (a, b, c) { ... }`, `arguments` holds the arguments passed to the function even though you haven't declared a parameter to be gathered. It works alongside the declared parameters.

But by and large, we will gather parameters in this book.

### application and contextualization

Hold that thought for a moment. JavaScript also provides a fourth way to set the context for a function. `apply` is a method implemented by every function that takes a context as its first argument, and it takes an array or array-like thing of arguments as its second argument. That's a mouthful, let's look at an example:
```js
    third.call(this, 1,2,3,4,5)
      //=> 3

    third.apply(this, [1,2,3,4,5])
      //=> 3
```      
Now let's put the two together. Here's another travesty:
```js
    const a = [1,2,3],
          accrete = a.concat;
        
    accrete([4,5])
      //=> Gobbledygook!
```
We get the result of concatenating `[4,5]` onto an array containing the global environment. Not what we want! Behold:
```js
    const contextualize = (fn, context) =>
      (...args) =>
        fn.apply(context, args)
    
    const accrete2 = contextualize(a.concat, a);
    accrete2([4,5]);
      //=> [ 1, 2, 3, 4, 5 ]
```      
Our `contextualize` function returns a new function that calls a function with a fixed context. It can be used to fix some of the unexpected results we had above. Consider:
```js
    var aFourthObject = {},
        returnThis = function () { return this; };
        
    aFourthObject.uncontextualized = returnThis;
    aFourthObject.contextualized = contextualize(returnThis, aFourthObject);
    
    aFourthObject.uncontextualized() === aFourthObject
      //=> true
    aFourthObject.contextualized() === aFourthObject
      //=> true
```      
Both are `true` because we are accessing them with `aFourthObject.` Now we write:
```js
    var uncontextualized = aFourthObject.uncontextualized,
        contextualized = aFourthObject.contextualized;
        
    uncontextualized() === aFourthObject;
      //=> false
    contextualized() === aFourthObject
      //=> true
```      
When we call these functions without using `aFourthObject.`, only the contextualized version maintains the context of `aFourthObject`.
      
We'll return to contextualizing methods later, in [Binding](#binding). But before we dive too deeply into special handling for methods, we need to spend a little more time looking at how functions and methods work.

## Method Decorators

In [function decorators](main_0_functions.md#function-decorators), we learned that a decorator takes a function as an argument, returns a function, and there's a semantic relationship between the two. If a function is a verb, a decorator is an adverb.

Decorators can be used to decorate methods provided that they carefully preserve the function's context. For example, here is a naïve version of `maybe` for one argument:
```js
    const maybe = (fn) =>
      x => x != null ? fn(x) : x;
```      
We use it like this:
```js
    const plus1 = x => x + 1;

    plus1(1)
      //=> 2
    plus1(0)
      //=> 1
    plus1(null)
      //=> 1
    plus1(undefined)
      //=> null
      
    const maybePlus1 = maybe(plus1);
    
    maybePlus1(1)
      //=> 2
    maybePlus1(0)
      //=> 1
    maybePlus1(null)
      //=> null
    maybePlus1(undefined)
      //=> undefined
```
This version doesn't preserve the context, so it can't be used as a method decorator. Instead, we have to convert the decoration from a fat arrow to a `function` function:
```js
    const maybe = (fn) =>
      function (x) {
        return x != null ? fn(x) : x;
      };
```
And then use `.call` to preserve `this`:
```js
    const maybe = (fn) =>
      function (x) {
        return x != null ? fn.call(this, x) : x;
      };
```      
Now that we have a "proper function," we can also handle variadic functions and methods. This variation only invokes the decorated function if none of the arguments are `null` or `undefined`:
```js
    const maybe = (fn) =>
      function (...args) {
        for (const i in args) {
          if (args[i] == null) return args[i];
        }
        return fn.apply(this, args);
      };
```
But back to basics. As long as we are correctly preserving `this` by one, using a `function`, and two, invoking the decorated function with `.call(this, ...)` or `.apply(this, ...)`, we can decorate methods as well as functions.

Now we can write things like:
```js
    const someObject = {
      setSize: maybe(function (size) {
        this.size = size;
      })
    }
```
And `this` is correctly set:
```js
    someObject.setSize(5);
    someObject
      //=> { setSize: [Function], size: 5 }

    someObject.setSize(null);
    someObject
      //=> { setSize: [Function], size: 5 }
```
Using `.call` or `.apply` and `arguments` is substantially slower than writing function decorators that don't set the context, so it might be right to sometimes write function decorators that aren't usable as method decorators. However, in practice you're far more likely to introduce a defect by failing to pass the context through a decorator than by introducing a performance pessimization, so the default choice should be to write all function decorators in such a way that they are "context agnostic."

In some cases, there are other considerations to writing a method decorator. If the decorator introduces state of any kind (such as `once` and `memoize` do), this must be carefully managed for the case when several objects share the same method through the mechanism of the [prototype](#prototypes) or through sharing references to the same function.

## Summary
### Objects, Mutation, and State

* State can be encapsulated/hidden with closures.
* Encapsulations can be aggregated with composition.
* Encapsulation resists extension.
* The automagic binding `this` facilitates sharing of functions.
* Functions can be named and declared with a name.
