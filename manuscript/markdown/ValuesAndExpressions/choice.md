# Interlude: Choice and Truthiness

![Decaf and the Antidote](images/antidote.jpg)

We've seen operators that act on numeric values, like `+` and `%`. In addition to numbers, we often need to represent a much more basic idea of truth or falsehood. Is this array empty? Does this person have a middle name? Is this user logged in?

JavaScript does have "boolean" values, they're written `true` and `false`:

    true
      //=> true
      
    false
      //=> false
      
`true` and `false` are value types. All values of `true` are `===` all other values of true. We can see that is the case by looking at some operators we can perform on boolean values, `!`, `&&`, and `||`. To being with, `!` is a unary prefix operator that negates its argument. So:

    !true
      //=> false
      
    !false
      //=> true
      
The `&&` and `||` operators are binary infix operators that perform "logical and" and "logical or" respectively:

    false && false //=> false
    false && true  //=> false
    true  && false //=> false
    true  && true  //=> true

    false || false //=> false
    false || true  //=> true
    true  || false //=> true
    true  || true  //=> true
    
Now, note well: We have said what happens if you pass boolean values to `!`, `&&`, and `||`, but we've said nothing about expressions or about passing other values. We'll look at those presently.

### truthiness and the ternary operator

In JavaScript, there is a notion of "truthiness." Every value in JavaScript is "truthy" except `false`, `null`, and `undefined`. The opposite of "truthy" is "falsy," and the aforementioned `false`, `null`, and `undefined` are considered "falsy."

The reason why this matters is that the various logical operators (as well as the if statement) actually operate on *truthiness*, not on boolean values. This affects the way the `!`, `&&`, and `||` operators work. We'll look at them in a moment, but first, we'll look at one more operator.

JavaScript inherited an operator from the C family of languages, the *ternary* operator. It's the only operator that takes *three* arguments. It looks like this: `first ? second : third`. It evaluates `first`, and if `first` is "truthy", it evaluates `second` and that is its value. If `first` is not truthy, it evaluates `third` and that is its value.

This is a lot like the `if` statement, however it is an *expression*, not a statement, and that can be very valuable. It also doesn't introduce braces, and that can be a help or a hindrance if we want to introduce a new scope or use statements.

Here're some simple examples of the ternary operator:

    true ? 'Hello' : 'Good bye'
      //=> 'Hello'

    null ? 'Hello' : 'Good bye'
      //=> 'Good bye'
      
    [1, 2, 3, 4, 5].length === 5 ? 'Pentatonic' : 'Quasimodal'
      //=> 'Pentatonic'

The fact that either the second or the third (but not both) expressions are evaluated can have important repercussions. Consider this hypothetical example:

    const status = isAuthorized(currentUser) ? deleteRecord(currentRecord) : 'Forbidden';
    
We certainly don't want JavaScript trying to evaluate `deleteRecord(currentRecord)` unless `isAuthorized(currentUser)` returns `true`.

### truthiness and operators

Our logical operators `!`, `&&`, and `||` are a little more subtle than our examples above implied. `!` is the simplest. It always returns `false` if its argument is truthy, and `true` is its argument is not truthy:

    !5
      //=> false
    
    !undefined
      //=> true
      
Programmers often take advantage of this behaviour to observe that `!!(someExpression)` will always evaluate to `true` is `someExpression` is truthy, and to `false` if it is not. So in JavaScript (and other languages with similar semantics), when you see something like `!!currentUser()`, this is an idiom that means "true if currentUser is truthy." Thus, a function like `currentUser()` is free to return `null`, or `undefined`, or `false` if there is no current user.

Thus, `!!` is the way we write "is truthy" in JavaScript. How about `&&` and `||`? What haven't we discussed?

First, and unlike `!`, `&&` and `||` do not necessarily evaluate to `true` or `false`. To be precise:

- `&&` evaluates its left-hand expression.
  - If its left-hand expression evaluates to something falsy, `&&` returns the value of its left-hand expression without evaluating its right-hand expression.
  - If its left-hand expression evaluates to something truthy, `&&` evaluates its right-hand expression and returns the value of the right-hand expression.
- `||` evaluates its left-hand expression.
  - If its left-hand expression evaluates to something truthy, `||` returns the value of its left-hand expression without evaluating its right-hand expression.
  - If its left-hand expression evaluates to something false, `||` evaluates its right-hand expression and returns the value of the right-hand expression.

If we look at our examples above, we see that when we pass `true` and `false` to `&&` and `||`, we do indeed get `true` or `false` as a result. But when we pass other values, we no longer get `true` or `false`:

    1 || 2
      //=> 1
    
    null && undefined
      //=> null
      
    undefined && null
      //=> undefined
      
In JavaScript, `&&` and `||` aren't really boolean logical operators in the strict logical sense. They don't operate strictly on logical values, and they aren't reflexive: `a || b` is not always equal to `b || a`, and the same goes for `&&`.

The difference can be subtle, but important for certain use cases.