# Life on the Plantation: Metaobjects
## Why Metaobjects?

> In computer science, a metaobject is an object that manipulates, creates, describes, or implements other objects (including itself). The object that the metaobject is about is called the base object. Some information that a metaobject might store is the base object's type, interface, class, methods, attributes, parse tree, etc. --[Wikipedia](https://en.wikipedia.org/wiki/Metaobject)

It is possible to write software using objects alone. When we need behaviour for an object, we can give it methods by binding functions to keys in the object:

    const sam = {
      firstName: 'Sam',
      lastName: 'Lowry',
      fullName () {
        return this.firstName + " " + this.lastName;
      },
      rename (first, last) {
        this.firstName = first;
        this.lastName = last;
        return this;
      }
    }

We call this a "naïve" object. It has state and behaviour, but it lacks division of responsibility between its state and its behaviour.

This lack of separation has two drawbacks. First, it intermingles properties that are part of the model domain (such as `firstName`), with methods (and possibly other properties, although none are shown here) that are part of the implementation domain. Second, when we needed to share common behaviour, we could have objects share common functions, but does it not scale: There's no sense of organization, no clustering of objects and functions that share a common responsibility.

Metaobjects solve the lack-of-separation problem by separating the domain-specific properties of objects from their implementation-specific properties and the functions that represent their behaviour.

The basic principle of the metaobject is that we separate the mechanics of behaviour from the domain properties of the base object. This has immediate engineering benefits, and it's also the foundation for designing programs with formal classes, expectations, and delegation.

## Mixins, Forwarding, and Delegation

The simplest possible metaobject in JavaScript is a *mixin*. Consider our naïve object:

~~~~~~~~
const sam = {
  firstName: 'Sam',
  lastName: 'Lowry',
  fullName () {
    return this.firstName + " " + this.lastName;
  },
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
}
~~~~~~~~

We can separate its domain properties from its behaviour:

~~~~~~~~
const sam = {
  firstName: 'Sam',
  lastName: 'Lowry'
};

const Person = {
  fullName () {
    return this.firstName + " " + this.lastName;
  },
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};
~~~~~~~~

And use `Object.assign` to mix the behaviour in:

~~~~~~~~
Object.assign(sam, Person);

sam.rename
  //=> [Function]
~~~~~~~~

This allows us to separate the behaviour from the properties in our code.

Our `Person` object is a *mixin*, it provides functionality to be mixed into an object with a function like `Object.assign`. Mixins are not "copied" into objects in the sense of making brand new versions of each of their functions: `Object.assign` copies *references* to each function from the mixin into the target object.

We can test this for ourselves:

~~~~~~~~
sam.fullName === Person.fullName
  //=> true
  
sam.rename === Person.rename
  //=> true
~~~~~~~~

If we want to use the same behaviour with another object, we can do that:

~~~~~~~~
const peck = {
  firstName: 'Sam',
  lastName: 'Peckinpah'
};

Object.assign(peck, Person);
~~~~~~~~

And of course, that object gets references to the original functions as well:

~~~~~~~~
sam.fullName === peck.fullName
  //=> true
  
sam.rename === peck.rename
  //=> true
~~~~~~~~

Thus, many objects can mix one object in.

Things get even better: One object can mix many objects in:

~~~~~~~~
const HasCareer = {
  career () {
    return this.chosenCareer;
  },
  setCareer (career) {
    this.chosenCareer = career;
    return this;
  }
};

Object.assign(peck, Person, HasCareer);

peck.setCareer('Director');
~~~~~~~~

Since many objects can all mix the same object in, and since one object can mix many objects into itself, there is a *many-to-many* relationship between objects and mixins.

### forwarding

Another way to build a metaobject that defines behaviour for another object is by having the object *forward* one or more method calls to a metaobject.

~~~~~~~~
function forward (receiver, metaobject, ...methods) {
  methods.forEach(function (methodName) {
    receiver[methodName] = (...args) => metaobject[methodName](...args)
  });

  return receiver;
};
~~~~~~~~

This function *forwards* methods to another object. Any other object, it could be a metaobject specifically designed to define behaviour, or it could be a domain object that has other responsibilities. Like mixins, one object might forward method invocations to more than one metaobject.

In this example, we start with an investment portfolio metaobject that has a `netWorth` method:

~~~~~~~~
const portfolio = (function () {
  const investments = Symbol();
  
  return {
    [investments]: [],
    addInvestment (investment) {
      this[investments].push(investment);
    },
    netWorth () {
      return this[investments].reduce(
        function (acc, investment) {
          return acc + investment.value;
        },
        0
      );
    }
  };
})();
~~~~~~~~

And next we create an investor who has a portfolio of investments:

~~~~~~~~
const investor = forward({}, portfolio, "addInvestment", "netWorth");

investor.addInvestment({ type: "art", value: 1000000 })
investor.addInvestment({ type: "art", value: 2000000 })
investor.netWorth()
  //=> 3000000
~~~~~~~~

### forwarding

Forwarding is a relationship between an object that receives a method invocation receiver and a provider object. They may be peers. The provider may be contained by the consumer. Or perhaps the provider is a metaobject.

When forwarding, the provider object has its own state. There is no special binding of function contexts, instead the consumer object has its own methods that forward to the provider and return the result. Our `forward` function above handles all of that, iterating over the provider's properties and making forwarding methods in the consumer.

The key idea is that when forwarding, the provider object handles each method *in its own context*. And because there is a forwarding method in the consumer object and a handling method in the provider, the two can be varied independently. Each forwarding function invokes the method in the provider *by name*. So we can do this:

~~~~~~~~
portfolio.netWorth = function () {
  return "I'm actually bankrupt!";
}
~~~~~~~~

We're overwriting the method in the `portfolio` object, but not the forwarding function. So now, our `investor` object will forward invocations of `netWorth` to the new function, not the original.

We say that mixing in is "early bound," while forwarding is "late bound:" We'll look up the method when it's invoked.

### shared forwarding

The premise of a mixin is that every time you mix the metaobject's behaviour into an object, the receiver holds the state for the behaviour being mixed in. Thus, you can mix the same metaobject into many objects, and they each will have their own state.

Forwarding does not work this way. When objects A and B both forward to C, the private state for C is held in C, and thus A and B share state. Sometimes this is what we want. but if it isn't, we must be very careful about using forwarding.

### summarizing what we know so far

So now we have two things: Mixing in a mixin, and forwarding to a first-class object. And we've seen that mixins *execute in the context of the receiver*, but forwarding is *late-bound*.

Which provokes a question: What is evaluated in the receiver's context, but late-bound, not early-bound?

### delegation

Let's build it. Here's a version of the `forward` function, modified to evaluate method invocation in the receiver's context:

~~~~~~~~
function delegate (receiver, metaobject, ...methods) {
  methods.forEach(function (methodName) {
    receiver[methodName] = (...args) => metaobject[methodName].apply(receiver, args)
  });

  return receiver;
};
~~~~~~~~

This new `delegate` function does exactly the same thing as the `forward` function, but the line that does the delegation looks like this:

~~~~~~~~
receiver[methodName] = (...args) => metaobject[methodName].apply(receiver, args)
~~~~~~~~

It uses the receiver as the context instead of the provider. This has all the same coupling implications that our mixins have, of course. And it layers in additional indirection. But unlike a mixin and like forwarding, the indirection gives us some late binding, allowing us to modify the metaobject's methods *after* we have delegated behaviour from a receiver to it.

### delegation vs. forwarding

Delegation and forwarding are both very similar. One metaphor that might help distinguish them is to think of receiving an email asking you to donate some money to a worthy charity.

* If you *forward* the email to a friend, and the friend donates money, the friend is donating their own money and getting their own tax receipt.
* If you *delegate* responding to your accountant, the accountant donates *your* money to the charity and you receive the tax receipt.

In both cases, the other entity does the work when you receive the email.

[fm]: https://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/

## Later Binding

When comparing Mixins to Delegation, we noted that Mixins are early bound and Delegation is late bound. Let's be specific. Given:

~~~~~~~~
const Incrementor = {
  increment () {
    ++this._value;
    return this;
  },
  value (optionalValue) {
    if (optionalValue != null) {
      this._value = optionalValue;
    }
    return this._value;
  }
};

const counter = Object.assign({}, Incrementor);
~~~~~~~~

We are mixing `Incrementor` into `counter`. At some point later, we encounter:

~~~~~~~~
counter.value(42);
~~~~~~~~

What function handles the invocation of `.value`? because we mixed `Incrementor` into `counter`, it's the same function as `Incrementor.counter`. We don't look that up when `counter.value(42)` is evaluated, because that was bound to `counter.value` when we extended `counter`. This is early binding.

However, given:

~~~~~~~~
const counter = {};

delegate(counter, Incrementor);

// ...time passes...

counter.value(42);
~~~~~~~~

We again are most likely invoking `Incrementor.value`, but now we are determining this *at the time `counter.value(42)` is evaluated*. We bound the target of the delegation, `Incrementor`, to `counter`, but we are going to look the actual property of `Incrementor.value` up when it is invoked. This is late binding, and it is useful in that we can make some changes to `Incrementor` after the delegation has been set up, perhaps to add some logging.

It is very nice not to have to do things like this in a very specific order: When things have to be done in a specific order, they are *coupled in time*. Late binding is a decoupling technique.

### but wait, there's more

But we can get *even later than that*. Although the specific function is late bound, the target of the delegation, `Incrementor`, is early bound. We can late bind that too! Here's a variation on `delegate`:

~~~~~~~~
function delegateToOwn (receiver, propertyName, ...methods) {
  methods.forEach(function (methodName) {
    receiver[methodName] = function () {
      const metaobject = receiver[propertyName];
      return metaobject[methodName].apply(receiver, arguments);
    };
  });

  return receiver;
};
~~~~~~~~

This function sets things up so that an object can delegate to one of its own properties, instead of an arbitrary object. It's quite common for an object to forward methods to one of its own properties. In this manner, objects can be constructed using *composition*. 

Let's take another look at the investor example. Here's the `portfolio` we used before. modified to use the receiver's context like a mixin:

~~~~~~~~
const portfolio = (function () {
  const investmentsProperty = Symbol();
  
  return {
    addInvestment (investment) {
      this[investmentsProperty] || (this[investmentsProperty] = []);
      return this[investmentsProperty].push(investment);
    },
    netWorth () {
      this[investmentsProperty] || (this[investmentsProperty] = []);
      return this[investmentsProperty].reduce(
        function (acc, investment) {
          return acc + investment.value;
        },
        0
      );
    }
  };
})();
~~~~~~~~

Next we'll make that a property of our investor, and delegate to the `nestEgg` property by name, not the object itself:

~~~~~~~~
const investor = {
  nestEgg: portfolio
}

delegateToOwn(investor, 'nestEgg', 'addInvestment', 'netWorth');

investor.addInvestment({ type: "art", value: 1000000 })
investor.addInvestment({ type: "art", value: 2000000 })
investor.netWorth()
  //=> 3000000
~~~~~~~~

Our `investor` object delegates the `addInvestment` and `netWorth` methods to its own `nestEgg` property. So far, this is just like the `delegate` method above. But consider what happens if we decide to assign a new portfolio to our investor:

~~~~~~~~
const companyRetirementPlan = {
  netWorth () {
    return 1500000;
  }
}

investor.nestEgg = companyRetirementPlan;

investor.netWorth()
  //=> 1500000
~~~~~~~~

The `delegateToOwn` delegation now delegates to `companyRetirementPlan`, because it is bound to the property name, not to the original object. This seems questionable for portfolios--what happens to the old portfolio when you assign a new one?--but has tremendous application for modeling classes of behaviour that change dynamically.

### state machines

A very common use case for this delegation is when building [finite state machines][ssm]. As described in the book [Understanding the Four Rules of Simple Design][4r] by Corey Haines, you *could* implement [Conway's Game of Life][gol] using `if` statements. Hand waving furiously over other parts of the system, you might get:

[ssm]: https://en.wikipedia.org/wiki/Finite-state_machine
[4r]: https://leanpub.com/4rulesofsimpledesign
[gol]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life

~~~~~~~~
const Universe = {
  // ...
  numberOfNeighbours (location) {
    // ...
  }
};

const thisGame = Object.assign({}, Universe);

const Cell = {
  alive () {
    return this._alive;
  },
  numberOfNeighbours () {
    return thisGame.numberOfNeighbours(this._location);
  },
  aliveInNextGeneration () {
    if (this.alive()) {
      return (this.numberOfNeighbours() === 3);
    }
    else {
      return (this.numberOfNeighbours() === 2 || this.numberOfNeighbours() === 3);
    }
  }
};

const someCell = Object.assign({
  _alive: true,
  _location: {x: -15, y: 12}
}, Cell);
~~~~~~~~

One of the many insights from [Understanding the Four Rules of Simple Design][4r] is that this business of having an `if (alive())` in the middle of a method is a hint that cells are stateful.

We can extract this into a state machine using delegation to a property:

~~~~~~~~
const Alive = {
  alive () {
    return true;
  },
  aliveInNextGeneration () {
    return (this.numberOfNeighbours() === 3);
  }
};

const Dead = {
  alive () {
    return false;
  },
  aliveInNextGeneration () {
    return (this.numberOfNeighbours() === 2 || this.numberOfNeighbours() === 3);
  }
};

const FsmCell = {
  numberOfNeighbours () {
    return thisGame.numberOfNeighbours(this._location);
  }
}

delegateToOwn(Cell, '_state', ['alive', 'aliveInNextGeneration']);

const someFsmCell = Object.assign({
  _state: Alive,
  _location: {x: -15, y: 12}
}, FsmCell);
~~~~~~~~

`someFsmCell` delegates `alive` and `aliveInNextGeneration` to its `_state` property, and you can change its state with assignment:

~~~~~~~~
someFsmCell._state = Dead;
~~~~~~~~

In practice, states would be assigned en masse, but this demonstrates one of the simplest possible state machines. In the wild, most business objects are state machines, sometimes with multiple, loosely coupled states. Employees can be:

- In or out of the office;
- On probation, on contract, or permanent;
- Part time or full time.

Delegation to a property representing state takes advantage of late binding to break behaviour into smaller components that have cleanly defined responsibilities.

### late bound forwarding

The exact same technique can be used for forwarding to a property, and forwarding to a property can also be used for some kinds of state machines. Forwarding to a property has lower coupling than delegation, and is preferred where appropriate.

## Delegation via Prototypes

At this point, we're discussed separating behaviour from object properties using metaobjects while avoiding discussion of prototypes. This is deliberate, because what we have achieved is developing a vocabulary for describing what a prototype is.

Let's review what we know so far:

~~~~~~~~
const Person = {
  fullName: function () {
    return this.firstName + " " + this.lastName;
  },
  rename: function (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};
~~~~~~~~

So far, just like any other metaobject we'd use as a mixin, or perhaps with delegation.

~~~~~~~~
const sam = Object.create(Person);
sam
  //=> {}
~~~~~~~~

This is different. Instead of creating an object and then using `Object.assign` to incorporate behaviour from a metaobject, we're using `Object.create`, a built-in method that creates the object while simultaneously associating it with a prototype.

The methods `fullName` and `rename` do not appear in its string representation. We'll find out why in a moment.

~~~~~~~~
sam.fullName
  //=> [Function]
sam.rename
  //=> [Function]
sam.rename('Samuel', 'Ballard')
  //=> { firstName: 'Samuel', lastName: 'Ballard' }
sam.fullName()
  //=> 'Samuel Ballard'
~~~~~~~~

And yet, they appear to be properties of `sam`, and we can invoke them in the usual fashion. Furthermore, we can tell that when the methods are invoked, the current context is being set to the receive, `sam`: That's why invoking `rename` sets `sam.firstName` and `sam.lastName`.

So far this is almost identical to using a mixin or delegation, but not a private mixin or forwarding because methods are evaluated in `sam`'s scope. The only difference appears to be how `sam` is displayed in the console. We recall that the big difference between a mixin and delegation is whether the methods are early or late bound.

So, if we *change* a method in `Person`, then if prototypes are early bound, `sam`'s behaviour will not change. Whereas if methods are late bound, `sam`'s behaviour will change. Let's try it:

~~~~~~~~
Person.fullName = function () {
  return this.lastName + ', ' + this.firstName;
};

sam.fullName()
  //=> 'Ballard, Samuel'
~~~~~~~~

Aha! Prototypes have *delegation semantics*: They are late bound, and evaluated in the receiver's context. This is exactly why many programmers say that prototypes are a delegation mechanism.

We've already seen delegation implemented via *method proxies*. Now we see it implemented via prototypes.

### prototypes are strictly many-to-one.

Delegation is a many-to-many relationship. One object can directly delegate to more than one metaobject, and more than one object can directly delegate to the same metaobject. This is not the case with prototypes: `Object.create` only allows you to specific *one* prototype for an object you're creating. You can *change* the prototype of an Object with `Object.setprototypeOf`, but each object can onlyhave one prototype at a time.

### sharing prototypes

Several objects can share one prototype:

~~~~~~~~
const sam = Object.create(Person);
const saywhatagain = Object.create(Person);
~~~~~~~~

`sam` and `saywhatagain` both share the `Person` prototype, so they both share the `rename` and `fullName` methods. But they each have their own properties, so:

~~~~~~~~
sam.rename('Samuel', 'Ballard');
saywhatagain.rename('Samuel', 'Jackson');

sam.fullName()
  //=> 'Samuel Ballard'

saywhatagain.fullName()
  //=> 'Samuel Jackson'
~~~~~~~~

The limitation of this scheme becomes apparent when we consider behaviours that need to be composed. Given `Person`, `IsAuthor`, and `HasBooks`, if we have some people that are authors, some that have children, some that aren't authors and don't have children, and some authors that have children, prototypes cannot directly manage these behaviours without duplication.

### prototypes are open for extension

With forwarding and delegation, the body of the method being proxied is late-bound because it is looked up by name when the method is invoked. This differs from mixins, where the body of the method is early bound by reference at the time the metaobject is mixed in.

~~~~~~~~
const methodNames = (object) =>
  Object.keys(object).filter(key => typeof(object[key]) == 'function')
    
function delegate (receiver, metaobject, ...methods = methodNames(metaobject)) {
  methods.forEach(function (methodName) {
    receiver[methodName] = (...args) => metaobject[methodName].apply(receiver, args)
  });

  return receiver;
};

const lowry = {};
delegate(lowry, Person);
lowry.rename('Sam', 'Lowry');

Person.fullName = function () {
  return this.firstName[0] + '. ' + this.lastName;
};
lowry.fullName();
  //=> 'S. Lowry'
~~~~~~~~

Prototypes and delegation both allow you to change the body of a method after a metaobject has been bound to an object.

But what happens if we add an entirely new method to our metaobject?

~~~~~~~~
Person.surname = function () {
  return this.lastName;
}
~~~~~~~~

An object using our method proxies *does not* delegate the new method to its metaobject, because we never created a method proxy for `surname`:

~~~~~~~~
lowry.surname()
  //=> TypeError: Object #<Object> has no method 'surname'
~~~~~~~~

Whereas, an object using a prototype *does* delegate the new method to the prototype:

~~~~~~~~
sam.surname()
 //=> 'Ballard'
~~~~~~~~

Prototypes late bind the method bodies *and* they late bind the identities of the methods being delegated. So you can add and remove methods to a prototype, and the behaviour of all of the objects bound to that prototype will be changed.

We say that *prototypes are open for extension*, because you can extend their behaviour *after* creating objects with them. We say that *mixins are closed for extension*, because behaviour added to a mixin does not change any of the objects that have already incorporated it.

### summarizing

Prototypes are a special kind of delegation mechanism that is built into JavaScript. Delegating through prototypes is:

1. Late bound on method bodies, just like delegation through method proxies;
2. Late bound on the method identities, which is superior to delegation through method proxies;
3. Evaluated in the receiver's context, just like delegation.
4. Open for extension, unlike mixins, forwarding, and explicit delegation.

Prototypes are usually the first form of metaobject that many developers learn in JavaScript, and quite often the last.

### ...one more thing!

There is one more way that delegation via prototypes differs from delegation via method proxies, and it's very important. We recall from above that object delegating to a prototype appear differently in the console than objects delegating via method proxies:

~~~~~~~~
sam
  //=> { firstName: 'Samuel', lastName: 'Ballard' }

lowry
  //=>
    { fullName: [Function],
      rename: [Function],
      firstName: 'Sam',
      lastName: 'Lowry' }
~~~~~~~~

The reason is very simple: The code for representing an object in the console iterates over its "own" properties, properties that belong to the object itself and not its prototype. In the case of `sam`, those are `firstName` and `lastName`, but not `fullName` or `rename` because those are properties of the prototype.

Whereas in the case of `lowry`, `fullName` and `rename` are properties of `Person`, but there are also function proxies that are properties of the `lowry` object itself.

We can test this for ourselves using the `.hasOwnProperty` method:

~~~~~~~~
sam.hasOwnProperty('fullName');
  //=> false
lowry.hasOwnProperty('fullName');
  //=> true
~~~~~~~~

One of the goals of metaobjects is to separate domain properties (such as `firstName`) from behaviour (such as `.fullName()`). All of our metaobject techniques allow us to do that in our written code, but prototypes do this extremely effectively in the runtime structure of the objects themselves.

This is extremely useful.

## Shared Prototypes

We can create a very simple object and associate it with a prototype:

~~~~~~~~
const Person = {
  fullName () {
    return this.firstName + " " + this.lastName;
  },
  rename (first, last) {
    this.firstName = first;
    this.lastName = last;
    return this;
  }
};

const sam = Object.create(Person);
~~~~~~~~

This associates behaviour with our object:

~~~~~~~~
sam.rename('sam', 'hill');
sam.fullName();
  //=> 'sam hill'
~~~~~~~~

There is no way to associate more than one prototype with the same object, but we can associate more than one object with the same prototype:

~~~~~~~~
const bewitched = Object.create(Person);
bewitched.rename('Samantha', 'Stephens');
~~~~~~~~

Although they share the prototype, their individual properties (as access with `this`), are separate:

~~~~~~~~
sam
  //=> { firstName: 'sam', lastName: 'hill' }
bewitched
  //=> { firstName: 'Samantha', lastName: 'Stephens' }
~~~~~~~~

This is very convenient.

### prototype chains

Consider our `HasCareer` mixin:

~~~~~~~~
const HasCareer = {
  career () {
    return this.chosenCareer;
  },
  setCareer (career) {
    this.chosenCareer = career;
    return this;
  }
};
~~~~~~~~

We can use it as a prototype, of course. But we already want to use `Person` as a prototype. What can we do? Obviously, we can combine `Person` and `HasCareer` into a "fat prototype" called `PersonWithCareer`. This is not great, a general principle of software is that entities should have a single clearly defined responsibility.

Even if we weren't hung up on single responsibility, another issue is that not all people have careers, so we need one prototype for people, and another for people with careers.

The catch is, another principle of good design is that every piece of knowledge should have one unambiguous representation. The knowledge of what makes a person falls into this category. If we were to add another method to `Person`, would we remember to add it to `PersonWithCareer`?

Let's work from two principles:

1. Any object can have an object as its prototype, and any object can be a prototype.
2. The behaviour of an object consists of all of its own behaviour, plus all the behaviour of its prototype.

When we say *any* object can have a prototype, does that include objects used as prototypes? Yes. Objects used as prototypes can have prototypes of their own.

Let's try it. First things first:

~~~~~~~~
const PersonWithCareer = Object.create(Person);
~~~~~~~~

Now let's mix `HasCareer` into `PersonWithCareer`:

~~~~~~~~
Object.assign(PersonWithCareer, HasCareer);
~~~~~~~~

And now we can use `PersonWithCareer` as a prototype:

~~~~~~~~
const goldie = Object.create(PersonWithCareer);
goldie.rename('Samuel', 'Goldwyn');
goldie.setCareer('Producer');
~~~~~~~~

Why does this work?

Imagine that we were writing a JavaScript (or other OO) language implementation. Method invocation is incredibly messy, full of special optimizations and so forth, but perhaps we only have ten days to get it done, so we might proceed like this without even thinking about prototype chains:

~~~~~~~~
function invokeMethod(receiver, methodName, listOfArguments) {
  return invokeMethodWithContext(receiver, receiver, methodName, listOfArguments);
}

function invokeMethodWithContext(context, receiver, methodName, listOfArguments) {
  const prototype;

  if (receiver.hasOwnProperty(methodName)) {
    return receiver[methodName].apply(context, listOfArguments);
  }
  else if (prototype = Object.getPrototypeOf(receiver)) {
    return invokeMethodWithContext(context, prototype, methodName, listOfArguments);
  }
  else {
    throw 'Method Missing ' + methodName;
  }
}
~~~~~~~~

Very simple: If the object implements the method, invoke it with `.apply`. If the object doesn't implement it but has a prototype, ask the prototype to implement it in the original receiver's context.

What if the prototype doesn't implement it but has a prototype of its own? Well, we'll recursively try that object too. Conceptually, this is what happens when we write:

~~~~~~~~
goldie.fullName()
  //=> 'Samuel Goldwyn'
~~~~~~~~

In theory, the JavaScript engine walks up a chain starting with the `goldie` object, followed by our `PersonWithCareer` prototype followed by our `Person` prototype.

### trees

Chaining prototypes is a useful technique, however it has some limitations. Because objects can have only one prototype, you cannot model all combinations of responsibilities solely with prototype chains. The classic example is known as "The W Pattern:"

Let's consider three prototypes to be used for employees in a factory: `Laborer`, `Manager`, and `OnProbation`.

All employees are either `Laborer` or `Manager`, but not both. So far, very easy, they can be prototypes. Some labourers are also on probation, as are some managers. How do we handle this with prototype chains?

![Labourers, Managers, and OnProbation](images/w.jpg)

Well, we can't have `Laborer` or `Manager` share `OnProbation` as a prototype, because then *all* labourers and managers would be on probation. And if we make `OnProbation` have `Laborer` as its prototype, there's no way to have a manager also be on probation without making it also a laborer, and that's not allowed.

Quite simply, a tree is an insufficient mechanism for modeling this relationship.

Prototype chains model trees, but most domain responsibilities cannot be represented as trees, so we must either revert to using "fat prototypes," or find another way to represent responsibilities, such as mixing metaobjects into prototypes.

### prototypes and mixins

We've seen how to use `Object.assign` to mix functionality directly into objects. Prototypes are objects, so it follows that we can mix functionality into prototypes. We used this technique when we created the `PersonWithCareer` shared prototype.

We can extend this technique when we run into limitations with prototype chains:

~~~~~~~~
const Laborer = {
 // ...
};
const Manager = {
 // ...
};
const Probationary = {
  // ...
};

const LaborerOnProbation = Object.assign({}, Laborer, Probationary);
const ManagerOnProbation = Object.assign({}, Manager, Probationary);
~~~~~~~~

Using mixins, we have created prototypes that model combining labor/management with probationary status.

### caveat programmer

Whether we're using prototype chains or mixins, we're introducing coupling. As discussed in [Mixins, Forwarding, and Delegation](#mixins-forwarding-and-delegation), prototypes that are brought into proximity with each other (by placing them anywhere in the same chain, or by mixing them into the same object) become deeply coupled because they both have complete access to an object's private internal state through `this`.

To reduce this coupling, we have to find a way to insulate prototypes from each other. Techniques like forwarding, while straightforward to use directly on an object or through a singleton prototype, require special handling when used in a shared prototype.

We'll discuss this at more length when we look at classes.
