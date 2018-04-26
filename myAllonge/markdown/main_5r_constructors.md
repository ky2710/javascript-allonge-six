# Recipes with Constructors and Classes

![These recipes are being roasted to perfection.](images/diedrich-roaster.jpg)

### Disclaimer

The recipes are written for practicality, and their implementation may introduce JavaScript features that haven't been discussed in the text to this point, such as methods and/or prototypes. The overall *use* of each recipe will fit within the spirit of the language discussed so far, even if the implementations may not.
## Bound {#bound}

Earlier, we saw a recipe for [getWith](#getWith) that plays nicely with properties:

    const getWith = (attr) => (object) => object[attr]

Simple and useful. But now that we've spent some time looking at objects with methods we can see that `get` (and `pluck`) has a failure mode. Specifically, it's not very useful if we ever want to get a *method*, since we'll lose the context. Consider some hypothetical class:

    function InventoryRecord (apples, oranges, eggs) {
      this.record = {
        apples: apples,
        oranges: oranges,
        eggs: eggs
      }
    }
    
    InventoryRecord.prototype.apples = function apples () {
      return this.record.apples
    }
    
    InventoryRecord.prototype.oranges = function oranges () {
      return this.record.oranges
    }
    
    InventoryRecord.prototype.eggs = function eggs () {
      return this.record.eggs
    }
    
    const inventories = [
      new InventoryRecord( 0, 144, 36 ),
      new InventoryRecord( 240, 54, 12 ),
      new InventoryRecord( 24, 12, 42 )
    ];
    
Now how do we get all the egg counts?

    mapWith(getWith('eggs'))(inventories)
      //=> [ [Function: eggs],
      //     [Function: eggs],
      //     [Function: eggs] ]

And if we try applying those functions...

    mapWith(getWith('eggs'))(inventories).map(
      unboundmethod => unboundmethod()
    )
      //=> TypeError: Cannot read property 'eggs' of undefined
      
It doesn't work, because these are unbound methods we're "getting" from each object. The context has been lost! Here's a new version of `get` that plays nicely with methods:

    const bound = (messageName, ...args) =>
      (args === [])
        ? instance => instance[messageName].bind(instance)
        : instance => Function.prototype.bind.apply(
                        instance[messageName], [instance].concat(args)
                      );

    mapWith(bound('eggs'))(inventories).map(
      boundmethod =>  boundmethod()
    )
      //=> [ 36, 12, 42 ]

`bound` is the recipe for getting a bound method from an object by name. It has other uses, such as callbacks. `bound('render')(aView)` is equivalent to `aView.render.bind(aView)`. There's an option to add a variable number of additional arguments, handled by:

    instance => Function.prototype.bind.apply(
                  instance[messageName], [instance].concat(args)
                );
        
The exact behaviour will be covered in [Binding Functions to Contexts](#binding). You can use it like this to add arguments to the bound function to be evaluated:

    InventoryRecord.prototype.add = function (item, amount) {
      this.record[item] || (this.record[item] = 0);
      this.record[item] += amount;
      return this;
    }
    
    mapWith(bound('add', 'eggs', 12))(inventories).map(
      boundmethod => boundmethod()
    )
      //=> [ { record: 
      //       { apples: 0,
      //         oranges: 144,
      //         eggs: 48 } },
      //     { record: 
      //       { apples: 240,
      //         oranges: 54,
      //         eggs: 24 } },
      //     { record: 
      //       { apples: 24,
      //         oranges: 12,
      //         eggs: 54 } } ]
      
 ## Send {#send}

Previously, we saw that the recipe [bound](#bound) can be used to get a bound method from an instance. Unfortunately, invoking such methods is a little messy:

    mapWith(bound('eggs'))(inventories).map(
      boundmethod => boundmethod() 
    )
      //=> [ 36, 12, 42 ]

As we noted, it's ugly to write

    boundmethod => boundmethod()

So instead, we write a new recipe:

    const send = (methodName, ...args) =>
      (instance) => instance[methodName].apply(instance, args);

    mapWith(send('apples'))(inventories)
      //=> [ 0, 240, 24 ]
      
`send('apples')` works very much like `&:apples` in the Ruby programming language. You may ask, why retain `bound`? Well, sometimes we want the function but don't want to evaluate it immediately, such as when creating callbacks. `bound` does that well.
## Invoke {#invoke}

[Send](#send) is useful when invoking a function that's a member of an object (or of an instance's prototype). But we sometimes want to invoke a function that is designed to be executed within an object's context. This happens most often when we want  to "borrow" a method from one "class" and use it on another object.

It's not an unprecedented use case. The Ruby programming language has a handy feature called [instance_exec]. It lets you execute an arbitrary block of code in the context of any object. Does this sound familiar? JavaScript has this exact feature, we just call it `.apply` (or `.call` as the case may be). We can execute any function in the context of any arbitrary object.

[instance_exec]: http://www.ruby-doc.org/core-1.8.7/Object.html#method-i-instance_exec

The only trouble with `.apply` is that being a method, it doesn't compose nicely with other functions like combinators. So, we create a function that allows us to use it as a combinator:

    const invoke = (fn, ...args) =>
      instance => fn.apply(instance, args);

For example, let's say someone else's code gives you an array of objects that are in part, but not entirely like arrays. Something like:

    const data = [
      { 0: 'zero', 
        1: 'one', 
        2: 'two',
        length: 3},
      { 0: 'none',
        length: 1 },
      // ...
    ];
    
If they were arrays, and we wanted to copy them, we would use:

    mapWith(send('slice', 0))(data)
  
Because arrays have a `.send` method. But our quasi-arrays have no such thing. So... We want to borrow the `.slice` method from arrays, but have it work on our data. `invoke([].slice, 0)` does the trick:

    mapWith(invoke([].slice, 0))(data)
      //=> [
             ["zero","one","two"],
             ["none"],
             // ...
           ]

### instance eval

`invoke` is useful when you have the function and are looking for the instance. It can be written "the other way around," for when you have the instance and are looking for the function:

    const instanceEval = instance =>
      (fn, ...args) => fn.apply(instance, args);
## Fluent {#fluent}

Object and instance methods can be bifurcated into two classes: Those that query something, and those that update something. Most design philosophies arrange things such that update methods return the value being updated. For example:

    class Cake {
      setFlavour (flavour) { 
        return this.flavour = flavour 
      },
      setLayers (layers) { 
        return this.layers = layers 
      },
      bake () {
        // do some baking
      }
    }
    
    const cake = new Cake();
    cake.setFlavour('chocolate');
    cake.setLayers(3);
    cake.bake();

Having methods like `setFlavour` return the value being set mimics the behaviour of assignment, where `cake.flavour = 'chocolate'` is an expression that in addition to setting a property also evaluates to the value `'chocolate'`.

The [fluent] style presumes that most of the time when you perform an update, you are more interested in doing other things with the receiver than the values being passed as argument(s). Therefore, the rule is to return the receiver unless the method is a query:

    class Cake {
      setFlavour (flavour) { 
        this.flavour = flavour;
        return this;
      },
      setLayers (layers) { 
        this.layers = layers;
        return this;
      },
      bake () {
        // do some baking
        return this;
      }
    }

The code to work with cakes is now easier to read and less repetitive:

    const cake = new Cake().
                   setFlavour('chocolate').
                   setLayers(3).
                   bake();

For one-liners like setting a property, this is fine. But some functions are longer, and we want to signal the intent of the method at the top, not buried at the bottom. Normally this is done in the method's name, but fluent interfaces are rarely written to include methods like `setLayersAndReturnThis`.

[fluent]: https://en.wikipedia.org/wiki/Fluent_interface

When we write our own prototypes, the `fluent` method decorator solves this problem:

    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }

Now you can write methods like this:

    function Cake () {}

    Cake.prototype.setFlavour = fluent( function (flavour) { 
      this.flavour = flavour;
    });

It's obvious at a glance that this method is "fluent."

When we use the `class` keyword, we can decorate functions in a similar manner:

    class Cake {
      setFlavour (flavour) { 
        this.flavour = flavour;
      },
      setLayers (layers) { 
        this.layers = layers;
      },
      bake () {
        // do some baking
      }
    }
    Cake.prototype.setFlavour = fluent(Cake.prototype.setFlavour);
    Cake.prototype.setLayers = fluent(Cake.prototype.setLayers);
    Cake.prototype.bake = fluent(Cake.prototype.bake);
    
Or, we could write ourselves a slight variation:

    const fluent = (methodBody) =>
      function (...args) {
        methodBody.apply(this, args);
        return this;
      }
    
    const fluentClass = (clazz, ...methodNames) {
      for (let methodName of methodNames) {
        clazz.prototype[methodName] = fluent(clazz.prototype[methodName]);
      }
      return clazz;
    }
    
Now we can simply write:

    fluentClass(Cake, 'setFlavour', 'setLayers', 'bake');
