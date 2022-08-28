<a name="top"></a>
***[YDK JS]** -- **[YDK JS Yet]** -- **[Functional Light JS]** -- **[Understanding ES6]** -- **[JS Allongé-6th]***   

[YDK JS]: https://github.com/ky27100/You-Dont-Know-JS/tree/1st-ed/toc.md#top
[YDK JS Yet]: https://github.com/ky27100/You-Dont-Know-JS/blob/2nd-ed/toc.md#top
[Functional Light JS]: https://github.com/ky27100/Functional-Light-JS/blob/master/manuscript/toc.md#top
[Understanding ES6]: https://github.com/ky27100/understandinges6/blob/master/manuscript/toc.md#top
[JS Allongé-6th]: https://github.com/ky27100/javascript-allonge-six/blob/master/myAllonge/markdown/toc.md#top

## [JavaScript Allongé-6th](#middle)
*[Prefaces](book_1_preface.md) / [Prelude](book_2_prelude.md) / [Remarks](book_3_closing-time.md) / [Appendices](book_4_appendices.md) / [Leanpub](https://leanpub.com/javascriptallongesix/read#leanpub-auto-about-javascript-allong) - Nov 3, 2017(513p) - [By Reg Braithwaite](https://github.com/raganwald)* 

* [**Prelude: Values and Expressions over Coffee**](book_2_prelude.md#prelude-values-and-expressions-over-coffee)
* [**0 : Basic Functions**](main_0_functions.md)
	* [*Sub-0: Basic Numbers*](sub_0_numbers.md) 
	* [Recipe-0: Basic Functions](main_0r_functions.md)  
* [**1 : Composing and Decomposing Data**](main_1_Composing.md)
	* [*Sub-1: Choice and Truthiness*](sub_1_choice.md) 	
	* [Recipe-1: Data](main_1r_Composing.md) 
* [**2 : Objects and State**](main_2_objects.md)
	* [*Sub-2: Basic Strings and Quasi-Literals*](sub_2_strings.md)	
	* [Recipe-2: Objects, Mutations, and State](main_2r_objects.md)   
* [**3 : Collections**](main_3_collections.md)
	* [*Sub-3: “Object-Oriented Programming”*](sub_3_oop.md)  
* [**4 : Metaobjects**](main_4_metaobjects.md)
	* [*Sub-4: Symbols*](sub_4_symbols.md) 
* [**5 : Constructors and Classes**](main_5_constructors.md)
	* [*Sub-5: Impostors*](sub_5_impostors.md) 
	* [Recipe-5: Constructors and Classes](main_5r_constructors.md)   
* [**6 : Composing Class Behaviour**](main_6_classes.md)
	* [*Sub-6: Symmetry, Colour, and Charm*](sub_6_colours.md)
* [**7 : More Decorators**](main_7_dedorators.md)   
	* [Recipe-7: More Decorator](main_7r_dedorators.md)   
* [**Final Remarks: Closing Time at the Coffeeshop**](book_3_closing-time.md#closing-time-at-the-coffeeshop-final-remarks)

---
<a name="middle"></a>
***[YDK JS]** -- **[YDK JS Yet]** -- **[Functional Light JS]** -- **[Understanding ES6]** -- **[JS Allongé-6th]***   

## [JavaScript Allongé-6th](#top) *- detail*

* [**Prelude: Values and Expressions over Coffee**](book_2_prelude.md#prelude-values-and-expressions-over-coffee)        
	* values are expressions -- values and identity  

* [*Sub-0: Basic Numbers*](sub_0_numbers.md) 
* [**0 : Basic Functions**](main_0_functions.md) 
	* [Function basic](main_0_functions.md#as-little-as-possible-about-functions-but-no-less) : As Little As Possible About Functions, But No Less
	* [Argument & Parameter](main_0_functions.md#ah-id-like-to-have-an-argument-pleasezzz-fargs) : Ah. I’d Like to Have an Argument, Please.   
	* [Closures and Scope](main_0_functions.md#closures-and-scope)   
	* [const and lexical scope](main_0_functions.md#that-constant-coffee-craving) : That Constant Coffee Craving   
	* [Naming Functions](main_0_functions.md#naming-functions)     
	* [Combinators & Function Decorators](main_0_functions.md#combinators-and-function-decorators)  
	* [Building Blocks](main_0_functions.md#building-blocks) : Compositon, Partial application   
	* [Magic Names](main_0_functions.md#magic-names) : this, arguments     
	* [Summary](main_0_functions.md#summary)     
* [Recipe-0: Basic Functions](main_0r_functions.md)   
	* Partial Application -- Unary -- Tap -- Maybe -- Once -- Left-Variadic Functions -- Compose and Pipeline   

* [*Sub-1: Choice and Truthiness*](sub_1_choice.md)  
* [**1 : Composing & Decomposing Data**](main_1_Composing.md)
	* [Arrays and Destructuring Arguments](main_1_Composing.md#arrays-and-destructuring-arguments)
	* [Self-Similarity](main_1_Composing.md#self-similarity) : List(Array)
	* [Tail Calls (and Default Arguments)](main_1_Composing.md#tail-calls-and-default-arguments)   
	* [Garbage, Garbage Everywhere](main_1_Composing.md#garbage-garbage-everywhere)   
	* [Plain Old JavaScript Objects](main_1_Composing.md#plain-old-javascript-objects) 
	* [Mutation](main_1_Composing.md#mutation)
	* [Reassignment](main_1_Composing.md#reassignment)
	* [Copy on Write(Copy on Read)](main_1_Composing.md#copy-on-write)   
	* [Tortoises, Hares, and Teleporting Turtles](main_1_Composing.md#tortoises-hares-and-teleporting-turtles)   
	* [Functional Iterators](main_1_Composing.md#functional-iterators)
	* [Making Data Out Of Functions](main_1_Composing.md#making-data-out-of-functions)   
* [Recipe-1: Data](main_1r_Composing.md)   
	* mapWith -- Flip -- Object.assign -- Why?   
  
* [*Sub-2: Basic Strings and Quasi-Literals*](sub_2_strings.md)   
* [**2 : Objects & State**](main_2_objects.md)
	* [Encapsulating State with Closures](main_2_objects.md#encapsulating-state-with-closures)
	* :boom:[Composition and Extension](main_2_objects.md#composition-and-extension)
	* [This and That](main_2_objects.md#this-and-that) 
	* [What Context Applies When We Call a Function?](main_2_objects.md#what-context-applies-when-we-call-a-function)
	* [Method Decorators](main_2_objects.md#method-decorators)
	* [Summary](main_2_objects.md#summary)
* [Recipe-2: Objects, Mutations, and State](main_2r_objects.md)   
   * Memoize -- getWith -- pluckWith -- Deep Mapping   

* [*Sub-3: “Object-Oriented Programming”*](sub_3_oop.md)  
* [**3 : Collections**](main_3_collections.md)
	* [Iteration and Iterables](main_3_collections.md#iteration-and-iterables)   
		* functional iterators -- iterator objects -- iterables -- iterables out to infinity
		* ordered collections -- operations on ordered collections -- from -- Summary
	* [Generating Iterables](main_3_collections.md#generating-iterables)   
		* recursive iterators -- state machines -- javascript's generators -- generators are coroutines
		* generators and iterables -- more generators -- yielding iterables -- rewriting iterable operations -- Summary
	* [Lazy and Eager Collections](main_3_collections.md#lazy-and-eager-collections)   
	* [Interlude: The Carpenter Interviews for a Job](main_3_collections.md#interlude-the-carpenter-interviews-for-a-job)   
	* [Interactive Generators](main_3_collections.md#interactive-generators)   
	* [Basic Operations on Iterables](main_3_collections.md#basic-operations-on-iterables)   

* [*Sub-4: Symbols*](sub_4_symbols.md)   
* [**4 : Metaobjects**](main_4_metaobjects.md#life-on-the-plantation-metaobjects)
	* [Why Metaobjects?](main_4_metaobjects.md#why-metaobjects) 
	* [Mixins, Forwarding, and Delegation](main_4_metaobjects.md#mixins-forwarding-and-delegation)       
	* [Later Binding](main_4_metaobjects.md#later-binding)     
	* [Delegation via Prototypes](main_4_metaobjects.md#delegation-via-prototypes)    
	* [Shared Prototypes](main_4_metaobjects.md#shared-prototypes)    

* [*Sub-5: Impostors*](sub_5_impostors.md)   
* [**5 : Constructors & Classes**](main_5_constructors.md#finish-the-cup-constructors-and-classes)
	* [Constructors and `new`](main_5_constructors.md#constructors-and-new)    
	* [Why Classes in JavaScript?](main_5_constructors.md#why-classes-in-javascript)    
	* [Classes with class](main_5_constructors.md#classes-with-class)    
	* [Object Methods](main_5_constructors.md#object-methods)    
	* [Why Not Classes?](main_5_constructors.md#why-not-classes)    
	* [Summary](main_5_constructors.md#summary) 
* [Recipe-5: Constructors and Classes](main_5r_constructors.md)   
	* Bound -- Send -- Invoke -- Fluent   

* [*Sub-6: Symmetry, Colour, and Charm*](sub_6_colours.md#colourful-mugs-symmetry-colour-and-charm)   
* [**6 : Composing Class Behaviour**](main_6_classes.md#con-panna-composing-class-behaviour)
	* [Extending Classes with Mixins](main_6_classes.md#extending-classes-with-mixins)    
	* [Functional Mixins](main_6_classes.md#functional-mixins)    
	* [Emulating Multiple Inheritance](main_6_classes.md#emulating-multiple-inheritance)    
	* [Preventing Property Conflicts](main_6_classes.md#preventing-property-conflicts)    
	* [Reducing Coupling](main_6_classes.md#reducing-coupling)    

* [**7 : More Decorators**](main_7_dedorators.md#more-decorators)   
	* [Stateful Method Decorators](main_7_dedorators.md#stateful-method-decorators)    
	* [Class Decorators beyond ES6/ECMAScript 2015](main_7_dedorators.md#class-decorators-beyond-es6ecmascript-2015)    
	* [Method Decorators beyond ES6/ECMAScript 2015](main_7_dedorators.md#method-decorators-beyond-es6ecmascript-2015)    
	* [Lightweight Traits](main_7_dedorators.md#lightweight-traits)    
* [Recipe-7: More Decorator](main_7r_dedorators.md)   
	* After Method Advice -- Before Method Advice -- Provided and Unless -- Method Advice      

* [**Final Remarks: Closing Time at the Coffeeshop**](book_3_closing-time.md#closing-time-at-the-coffeeshop-final-remarks)   
	* javascript beyond es6/ecmascript 2015 -- the lightweight way    
	
