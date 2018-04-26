# A Pull of the Lever: Prefaces

![Caffe Molinari](images/caffemolinari.jpg)

> "Café Allongé, also called Espresso Lungo, is a drink midway between an Espresso and Americano in strength. There are two different ways to make it. The first, and the one I prefer, is to add a small amount of hot water to a double or quadruple Espresso Ristretto. Like adding a splash of water to whiskey, the small dilution releases more of the complex flavours in the mouth.
>
> "The second way is to pull an extra long double shot of Espresso. This achieves approximately the same ratio of oils to water as the dilution method, but also releases a different mix of flavours due to the longer extraction. Some complain that the long pull is more bitter and detracts from the best character of the coffee, others feel it releases even more complexity.
>
> "The important thing is that neither method of preparation should use so much water as to result in a sickly, pale ghost of Espresso. Moderation in all things." 
## About JavaScript Allongé

JavaScript Allongé is a first and foremost, a book about *programming with functions*. It's written in JavaScript, because JavaScript hits the perfect sweet spot of being both widely used, and of having proper first-class functions with lexical scope. If those terms seem unfamiliar, don't worry: JavaScript Allongé takes great delight in explaining what they mean and why they matter.

*JavaScript Allongé* begins at the beginning, with values and expressions, and builds from there to discuss types, identity, functions, closures, scopes, collections, iterators, and many more subjects up to working with classes and instances.

It also provides recipes for using functions to write software that is simpler, cleaner, and less complicated than alternative approaches that are object-centric or code-centric. JavaScript idioms like function combinators and decorators leverage JavaScript's power to make code easier to read, modify, debug and refactor.

*JavaScript Allongé* teaches you how to handle complex code, and it also teaches you how to simplify code without dumbing it down. As a result, *JavaScript Allongé* is a rich read releasing many of JavaScript's subtleties, much like the Café Allongé beloved by coffee enthusiasts everywhere.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript

### why the "six" edition?

ECMAScript 2015 (formerly called ECMAScript 6 or "ES6"), is ushering in a very large number of improvements to the way programmers can write small, powerful components and combine them into larger, fully featured programs. Features like destructuring, block-structured variables, iterables, generators, and the class keyword are poised to make JavaScript programming more expressive.

Prior to ECMAScript 2015, JavaScript did not include many features that programmers have discovered are vital to writing great software. For example, JavaScript did not include block-structured variables. Over time, programmers discovered ways to roll their own versions of important features.

For example, block-structured languages allow us to write:

    for (int i = 0; i < array.length; ++i) {
      // ...
    }

And the variable `i` is scoped locally to the code within the braces. Prior to ECMAScript 2015, JavaScript did not support block-structuring, so programmers borrowed a trick from the Scheme programming language, and would write:

    var i;

    for (i = 0; i < array.length; ++i) {
      (function (i) {
        // ...
      })(i)
    }

To create the same scoping with an Immediately Invoked Function Expression, or "IIFE."

Likewise, many programming languages permit functions to have a variable number of arguments, and to collect the arguments into a single variable as an array. In Ruby, we can write:

    def foo (first, *rest)
      # ...
    end

Prior to ECMAScript 2015, JavaScript did not support collecting a variable number of arguments into a parameter, so programmers would take advantage of an awkward work-around and write things like:

    function foo () {
      var first = arguments[0],
          rest  = [].slice.call(arguments, 1);

      // ...
    }

The first edition of JavaScript Allongé explained these and many other patterns for writing flexible and composable programs in JavaScript, but the intention wasn't to explain how to work around JavaScript's missing features: The intention was to explain why the style of programming exemplified by the missing features is important.

Working around the missing features was a necessary evil.

But now, JavaScript is gaining many important features, in part because the governing body behind JavaScript has observed that programmers are constantly working around the same set of limitations. With ECMASCript 2015, we can write:

    for (let i = 0; i < array.length; ++i) {
      // ...
    }

And `i` is scoped to the for loop. We can also write:

    function foo (first, ...rest) {
      // ...
    }

And presto, `rest` collects the rest of the arguments without a lot of malarky involving slicing `arguments`. Not having to work around these kinds of missing features makes JavaScript Allongé a *better book*, because it can focus on the *why* to do something and *when* to do it, instead of on the how to make it work

JavaScript Allongé, The "Six" Edition packs all the goodness of JavaScript Allongé into a new, updated package that is relevant for programmers working with (or planning to work with) the latest version of JavaScript.

### that's nice. is that the only reason?

Actually, no.

If it were just a matter of updating the syntax, the original version of JavaScript Allongé could have simply iterated, slowly replacing old syntax with new. It would have continued to say much the same things, only with new syntax.

*But there's more to it than that*. The original JavaScript Allongé was not just written to teach JavaScript: It was written to describe certain ideas in programming: Working with small, independent entities that compose together to make bigger programs. Thus, the focus on things like writing decorators.

As noted above, JavaScript was chosen as the language for Allongé because it hit a sweet spot of having a large audience of programmers and having certain language features that happen to work well with this style of programming.

ECMAScript 2015 does more than simply update the language with some simpler syntax for a few things and help us avoid warts. It makes a number of interesting programming techniques easy to explain and easy to use. And these techniques dovetail nicely with Allongé's focus on composing entities and working with functions.

Thus, the "six" edition introduces [classes](#classes) and [mixins](#mixins). It introduces the notion of implementing private properties with [symbols](#symbols). It introduces [iterators and generators](#collections). But the common thread that runs through all these things is that since they are all simple objects and simple functions, we can use the same set of "programming with functions" techniques to build programs by composing small, flexible, and decoupled entities.

We just call some of those functions [constructors](#new), others [decorators](#decorators), others [functional mixins](#functional-mixins), and yet others, [policies](#policies).

Introducing so many new ideas did require a major rethink of the way the book was organized. And introducing these new ideas did add substantially to its bulk. But even so, in a way it is still explaining the exact same original idea that programs are built out of small, flexible functions composed together.

## What JavaScript Allongé is. And isn't.

![JavaScript Allongé is a book about thinking about programs](images/thinking.jpg)

JavaScript Allongé is a book about programming with functions. From functions flow many ideas, from decorators to methods to delegation to mixins, and onwards in so many fruitful directions.

The focus in this book on the underlying ideas, what we might call the fundamentals, and how they combine to form new ideas. The intention is to improve the way we think about programs. That's a good thing.

But while JavaScript Allongé attempts to be provocative, it is not *prescriptive*. There is absolutely no suggestion that any of the techniques shown here are the only way to do something, the best way, or even an acceptable way to write programs that are intended to be used, read, and maintained by others.

Software development is a complex field. Choices in development are often driven by social considerations. People often say that software should be written for people to read. Doesn't that depend upon the people in question? Should code written by a small team of specialists use the same techniques and patterns as code maintained by a continuously changing cast of inexperienced interns?

Choices in software development are also often driven by requirements specific to the type of software being developed. For example, business software written in-house has a very different set of requirements than a library written to be publicly distributed as open-source.

Choices in software development must also consider the question of consistency. If a particular codebase is written with lots of helper functions that place the subject first, like this:

{:lang="js"}
~~~~~~~~
const mapWith = (iterable, fn) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        yield fn(element);
      }
    }
  });
~~~~~~~~

Then it can be jarring to add new helpers written that place the verb first, like this:

{:lang="js"}
~~~~~~~~
const filterWith = (fn, iterable) =>
  ({
    [Symbol.iterator]: function* () {
      for (let element of iterable) {
        if (!!fn(element)) yield element;
      }
    }
  });
~~~~~~~~

There are reasons why the second form is more flexible, especially when used in combination with partial application, but does that outweigh the benefit of having an entire codebase do everything consistently the first way or the second way?

Finally, choices in software development cannot ignore the tooling that is used to create and maintain software. The use of source-code control systems with integrated diffing rewards making certain types of focused changes. The use of [linters][lint] makes checking for certain types of undesirable code very cheap. Debuggers encourage the use of functions with explicit or implicit names. Continuous integration encourages the creation of software in tandem with and factored to facilitate the creation of automated test suites.

JavaScript Allongé does not attempt to address the question of JavaScript best practices in the wider context of software development, because JavaScript Allongé isn't a book about practicing, it's a book about thinking.

[lint]: https://en.wikipedia.org/wiki/Lint_(software)

### how this book is organized

*JavaScript Allongé* introduces new aspects of programming with functions in each chapter, explaining exactly how JavaScript works. Code examples within each chapter are small and emphasize exposition rather than serving as patterns for everyday use.

Following some of the chapters are a series of recipes designed to show the application of the chapter's ideas in practical form. While the content of each chapter builds naturally on what was discussed in the previous chapter, the recipes may draw upon any aspect of the JavaScript programming language.

## Foreword to the "Six" edition

ECMAScript 6 (short name: ES6; official name: ECMAScript 2015) was ratified as a standard on June 17. Getting there took a while – in a way, the origins of ES6 date back to the year 2000: After ECMAScript 3 was finished, TC39 (the committee evolving JavaScript) started to work on ECMAScript 4. That version was planned to have numerous new features (interfaces, namespaces, packages, multimethods, etc.), which would have turned JavaScript into a completely new language. After internal conflict, a settlement was reached in July 2008 and a new plan was made – to abandon ECMAScript 4 and to replace it with two upgrades:

* A smaller upgrade would bring a few minor enhancements to ECMAScript 3. This upgrade became ECMAScript 5.

* A larger upgrade would substantially improve JavaScript, but without being as radical as ECMAScript 4. This upgrade became ECMAScript 6 (some features that were initially discussed will show up later, in upcoming ECMAScript versions).

ECMAScript 6 has three major groups of features:

* Better syntax for features that already exist (e.g. via libraries). For example: classes and modules.

* New functionality in the standard library. For example:
    * New methods for strings and arrays
    * Promises (for asynchronous programming)
    * Maps and sets

* Completely new features. For example: Generators, proxies and WeakMaps.

With ECMAScript 6, JavaScript has become much larger as a language. _JavaScript Allongé, the "Six" Edition_ is both a comprehensive tour of its features and a rich collection of techniques for making better use of them. You will learn much about functional programming and object-oriented programming. And you’ll do so via ES6 code, handed to you in small, easily digestible pieces.

-- Axel Rauschmayer
[Blogger](http://www.2ality.com), [trainer](http://ecmanauten.de) and author of “[Exploring ES6](http://exploringjs.com)”
## Forewords to the First Edition

### michael fogus

As a life-long bibliophile and long-time follower of Reg's online work, I was excited when he started writing books. However, I'm very conservative about books -- let's just say that if there was an aftershave scented to the essence of "Used Book Store" then I would be first in line to buy. So as you might imagine I was "skeptical" about the decision to release JavaScript Allongé as an ongoing ebook, with a pay-what-you-want model. However, Reg sent me a copy of his book and I was humbled. Not only was this a great book, but it was also a great way to write and distribute books. Having written books myself, I know the pain of soliciting and receiving feedback.

The act of writing is an iterative process with (very often) tight revision loops. However, the process of soliciting feedback, gathering responses, sending out copies, waiting for people to actually read it (if they ever do), receiving feedback and then ultimately making sense out of how to use it takes weeks and sometimes months. On more than one occasion I've found myself attempting to reify feedback with content that either no longer existed or was changed beyond recognition. However, with the Leanpub model the read-feedback-change process is extremely efficient, leaving in its wake a quality book that continues to get better as others likewise read and comment into infinitude.

In the case of JavaScript Allongé, you'll find the Leanpub model a shining example of effectiveness. Reg has crafted (and continues to craft) not only an interesting book from the perspective of a connoisseur, but also an entertaining exploration into some of the most interesting aspects of his art. No matter how much of an expert you think you are, JavaScript Allongé has something to teach you... about coffee. I kid.

As a staunch advocate of functional programming, much of what Reg has written rings true to me. While not exclusively a book about functional programming, JavaScript Allongé will provide a solid foundation for functional techniques. However, you'll not be beaten about the head and neck with dogma. Instead, every section is motivated by relevant dialog and fortified with compelling source examples. As an author of programming books I admire what Reg has managed to accomplish and I envy the fine reader who finds JavaScript Allongé via some darkened channel in the Internet sprawl and reads it for the first time.

Enjoy.

-- Fogus, [fogus.me](http://www.fogus.me)

### matthew knox

A different kind of language requires a different kind of book.

JavaScript holds surprising depths--its scoping rules are neither strictly lexical nor strictly dynamic, and it supports procedural, object-oriented (in several flavors!), and functional programming.  Many books try to hide most of those capabilities away, giving you recipes for writing JavaScript in a way that approximates class-centric programming in other languages.  Not JavaScript Allongé.  It starts with the fundamentals of values, functions, and objects, and then guides you through JavaScript from the inside with exploratory bits of code that illustrate scoping, combinators, context, state, prototypes, and constructors.

Like JavaScript itself, this book gives you a gentle start before showing you its full depth, and like a Cafe Allongé, it's over too soon.  Enjoy!

--Matthew Knox, [mattknox.com](http://mattknox.com)
## A Personal Word About The Recipes

As noted, *JavaScript Allongé* alternates between chapters describing the semantics of JavaScript's functions with chapters containing recipes for writing programs with functions. You can read the book in order or read the chapters explaining JavaScript first and return to the recipes later.

The recipes share a common theme: They hail from a style of programming inspired by the creation of small functions that compose with each other. Using these recipes, you'll learn when it's appropriate to write:

    return mapWith(maybe(getWith('name')))(customerList);
    
Instead of:

    return customerList.map(function (customer) {
      if (customer) {
        return customer.name
      }
    });
    
As well as how it works and how to refactor it when you need. This style of programming is hardly the most common thing anyone does in JavaScript, so the argument can be made that more "practical" or "commonplace" recipes would be helpful. If you never read any other books about JavaScript, if you avoid blog posts and screen casts about JavaScript, if you don't attend workshops or talks about JavaScript, then I agree that this is not One Book to Rule Them All.

But given that there are other resources out there, and that programmers are curious creatures with an unslakable thirst for personal growth, we choose to provide recipes that you are unlikely to find anywhere else in anything like this concentration. The recipes reinforce the lessons taught in the book about functions in JavaScript.

You'll find all of the recipes collected online at [https://github.com/raganwald/allong.es](https://github.com/raganwald/allong.es). They're free to share under the MIT license.

[Reginald Braithwaite](http://braythwayt.com)  
reg@braythwayt.com  
@raganwald
## Legend

Some text in monospaced type like `this` in the text represents some code being discussed. Some monospaced code in its own lines also represents code being discussed:

    this.async = do (async = undefined) ->

      async = (fn) ->
        (argv..., callback) ->
          callback(fn.apply(this, argv))
          
Sometimes it will contain some code for you to type in for yourself. When it does, the result of typing something in will often be shown using `//=>`, like this:

    2 + 2
      //=> 4

T> A paragraph marked like this is a "key fact." It summarizes an idea without adding anything new.

X> A paragraph marked like this is a suggested exercise to be performed on your own.

A> A paragraph marked like this is an aside. It can be safely ignored. It contains whimsey and other doupleplusunserious logorrhea that will *not* be on the test.
## About JavaScript Allongé

JavaScript Allongé is a first and foremost, a book about *programming with functions*. It's written in JavaScript, because JavaScript hits the perfect sweet spot of being both widely used, and of having proper first-class functions with lexical scope. If those terms seem unfamiliar, don't worry: JavaScript Allongé takes great delight in explaining what they mean and why they matter.

*JavaScript Allongé* begins at the beginning, with values and expressions, and builds from there to discuss types, identity, functions, closures, scopes, collections, iterators, and many more subjects up to working with classes and instances.

It also provides recipes for using functions to write software that is simpler, cleaner, and less complicated than alternative approaches that are object-centric or code-centric. JavaScript idioms like function combinators and decorators leverage JavaScript's power to make code easier to read, modify, debug and refactor.

*JavaScript Allongé* teaches you how to handle complex code, and it also teaches you how to simplify code without dumbing it down. As a result, *JavaScript Allongé* is a rich read releasing many of JavaScript's subtleties, much like the Café Allongé beloved by coffee enthusiasts everywhere.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript

### why the "six" edition?

ECMAScript 2015 (formerly called ECMAScript 6 or "ES6"), is ushering in a very large number of improvements to the way programmers can write small, powerful components and combine them into larger, fully featured programs. Features like destructuring, block-structured variables, iterables, generators, and the class keyword are poised to make JavaScript programming more expressive.

Prior to ECMAScript 2015, JavaScript did not include many features that programmers have discovered are vital to writing great software. For example, JavaScript did not include block-structured variables. Over time, programmers discovered ways to roll their own versions of important features.

For example, block-structured languages allow us to write:

    for (int i = 0; i < array.length; ++i) {
      // ...
    }

And the variable `i` is scoped locally to the code within the braces. Prior to ECMAScript 2015, JavaScript did not support block-structuring, so programmers borrowed a trick from the Scheme programming language, and would write:

    var i;

    for (i = 0; i < array.length; ++i) {
      (function (i) {
        // ...
      })(i)
    }

To create the same scoping with an Immediately Invoked Function Expression, or "IIFE."

Likewise, many programming languages permit functions to have a variable number of arguments, and to collect the arguments into a single variable as an array. In Ruby, we can write:

    def foo (first, *rest)
      # ...
    end

Prior to ECMAScript 2015, JavaScript did not support collecting a variable number of arguments into a parameter, so programmers would take advantage of an awkward work-around and write things like:

    function foo () {
      var first = arguments[0],
          rest  = [].slice.call(arguments, 1);

      // ...
    }

The first edition of JavaScript Allongé explained these and many other patterns for writing flexible and composable programs in JavaScript, but the intention wasn't to explain how to work around JavaScript's missing features: The intention was to explain why the style of programming exemplified by the missing features is important.

Working around the missing features was a necessary evil.

But now, JavaScript is gaining many important features, in part because the governing body behind JavaScript has observed that programmers are constantly working around the same set of limitations. With ECMASCript 2015, we can write:

    for (let i = 0; i < array.length; ++i) {
      // ...
    }

And `i` is scoped to the for loop. We can also write:

    function foo (first, ...rest) {
      // ...
    }

And presto, `rest` collects the rest of the arguments without a lot of malarky involving slicing `arguments`. Not having to work around these kinds of missing features makes JavaScript Allongé a *better book*, because it can focus on the *why* to do something and *when* to do it, instead of on the how to make it work

JavaScript Allongé, The "Six" Edition packs all the goodness of JavaScript Allongé into a new, updated package that is relevant for programmers working with (or planning to work with) the latest version of JavaScript.

### that's nice. is that the only reason?

Actually, no.

If it were just a matter of updating the syntax, the original version of JavaScript Allongé could have simply iterated, slowly replacing old syntax with new. It would have continued to say much the same things, only with new syntax.

*But there's more to it than that*. The original JavaScript Allongé was not just written to teach JavaScript: It was written to describe certain ideas in programming: Working with small, independent entities that compose together to make bigger programs. Thus, the focus on things like writing decorators.

As noted above, JavaScript was chosen as the language for Allongé because it hit a sweet spot of having a large audience of programmers and having certain language features that happen to work well with this style of programming.

ECMAScript 2015 does more than simply update the language with some simpler syntax for a few things and help us avoid warts. It makes a number of interesting programming techniques easy to explain and easy to use. And these techniques dovetail nicely with Allongé's focus on composing entities and working with functions.

Thus, the "six" edition introduces [classes](#classes) and [mixins](#mixins). It introduces the notion of implementing private properties with [symbols](#symbols). It introduces [iterators and generators](#collections). But the common thread that runs through all these things is that since they are all simple objects and simple functions, we can use the same set of "programming with functions" techniques to build programs by composing small, flexible, and decoupled entities.

We just call some of those functions [constructors](#new), others [decorators](#decorators), others [functional mixins](#functional-mixins), and yet others, [policies](#policies).

Introducing so many new ideas did require a major rethink of the way the book was organized. And introducing these new ideas did add substantially to its bulk. But even so, in a way it is still explaining the exact same original idea that programs are built out of small, flexible functions composed together.

