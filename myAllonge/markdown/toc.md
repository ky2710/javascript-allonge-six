<a name="top"></a>
## [JavaScript Allongé-6th](#middle)
***[You Dont Konw JS]** --- **[Functional Light JS]** --- **[Understanding ES6]** --- **[Javascript Allongé-6th]***      
*[Prefaces](book_1_preface.md) / [Prelude](book_2_prelude.md) / [Remarks](book_3_closing-time.md) / [Appendices](book_4_appendices.md) / [Leanpub](https://leanpub.com/javascriptallongesix/read#leanpub-auto-about-javascript-allong) - Nov 3, 2017(513p) - [By Reg Braithwaite](https://github.com/raganwald)* 
* [**Main-0: Basic Functions**](main_0_functions.md) / [*Sub-0: Basic Numbers*](sub_0_numbers.md)   
  * As Little As Possible About Functions, But No Less -- Ah. I’d Like to Have an Argument, Please.
  * Closures and Scope -- That Constant Coffee Craving -- Naming Functions
  * Combinators and Function Decorators -- Building Blocks -- Magic Names -- Summary   
  *[Recipe-0: Basic Functions](main_0r_functions.md)*   
  * Partial Application -- Unary -- Tap -- Maybe -- Once -- Left-Variadic Functions -- Compose and Pipeline   
* [**Main-1: Composing and Decomposing Data**](main_1_Composing.md) / [*Sub-1: Choice and Truthiness*](sub_1_choice.md)   
  * Arrays and Destructuring Arguments -- Self-Similarity -- Tail Calls (and Default Arguments)
  * Garbage, Garbage Everywhere -- Plain Old JavaScript Objects -- Mutation -- Reassignment -- Copy on Write   
  * Tortoises, Hares, and Teleporting Turtles -- Functional Iterators -- Making Data Out Of Functions   
  *[Recipe-1: Data](main_1r_Composing.md)*  
  * mapWith -- Flip -- Object.assign -- Why?   
* [**Main-2: Objects and State**](main_2_objects.md) / [*Sub-2: Basic Strings and Quasi-Literals*](sub_2_strings.md)   
  * Encapsulating State with Closures -- Composition and Extension -- This and That   
  * What Context Applies When We Call a Function? -- Method Decorators -- Summary   
  *[Recipe-2: Objects, Mutations, and State](main_2r_objects.md)*   
  * Memoize -- getWith -- pluckWith -- Deep Mapping   
* [**Main-3: Collections**](main_3_collections.md) / [*Sub-3: “Object-Oriented Programming”*](sub_3_oop.md)  
  * Iteration and Iterables -- Generating Iterables -- Lazy and Eager Collections   
  * Interlude: The Carpenter Interviews for a Job -- Interactive Generators -- Basic Operations on Iterables   
* [**Main-4: Metaobjects**](main_4_metaobjects.md) / [*Sub-4: Symbols*](sub_4_symbols.md)   
  * Why Metaobjects? -- Mixins, Forwarding, and Delegation -- Later Binding    
  * Delegation via Prototypes -- Shared Prototypes   
* [**Main-5: Constructors and Classes**](main_5_constructors.md) / [*Sub-5: Impostors*](sub_5_impostors.md)   
  * Constructors and new -- Why Classes in JavaScript? -- Classes with class   
  * Object Methods -- Why Not Classes? -- Summary   
  *[Recipe-5: Constructors and Classes](main_5r_constructors.md)*   
  * Bound -- Send -- Invoke -- Fluent   
* [**Main-6: Composing Class Behaviour**](main_6_classes.md) / [*Sub-6: Symmetry, Colour, and Charm*](sub_6_colours.md)   
  * Extending Classes with Mixins -- Functional Mixins -- Emulating Multiple Inheritance   
  * Preventing Property Conflicts -- Reducing Coupling   
* [**Main-7: More Decorators**](main_7_dedorators.md)   
  * Stateful Method Decorators -- Class Decorators beyond ES6/ECMAScript 2015   
  * Method Decorators beyond ES6/ECMAScript 2015 -- Lightweight Traits   
  *[Recipe-7: More Decorator](main_7r_dedorators.md)*   
  * After Method Advice -- Before Method Advice -- Provided and Unless -- Method Advice   
  
<a name="middle"></a>
## [JavaScript Allongé-6th](#top) *- detail*
***[You Dont Konw JS]** --- **[Functional Light JS]** --- **[Understanding ES6]** --- **[Javascript Allongé-6th]***      
*[Prefaces](book_1_preface.md) / [Prelude](book_2_prelude.md) / [Leanpub](https://leanpub.com/javascriptallongesix/read#leanpub-auto-about-javascript-allong) - Nov 3, 2017(513p) - [By Reg Braithwaite](https://github.com/raganwald)*    
#### :black_medium_square:[ Prelude: Values and Expressions over Coffee](book_2_prelude.md)        
    * values are expressions     * values and identity  
* [*Sub-0: Basic Numbers*](sub_0_numbers.md) - A Rich Aroma   
* [**Main-0: Basic Functions**](main_0_functions.md#the-first-sip-basic-functions) - The first sip   
    * function: [As Little As Possible About Functions, But No Less](main_0_functions.md#as-little-as-possible-about-functions-but-no-less)   
    * argument and parameter: [Ah. I’d Like to Have an Argument, Please.](main_0_functions.md#ah-id-like-to-have-an-argument-pleasezzz-fargs)   
    	* call by value -- variables and bindings -- call by sharing
    * [Closures and Scope](main_0_functions.md#closures-and-scope)   
    * const and lexical scope: [That Constant Coffee Craving](main_0_functions.md#that-constant-coffee-craving)   
    * [Naming Functions](main_0_functions.md#naming-functions)     
    * [Combinators and Function Decorators](main_0_functions.md#combinators-and-function-decorators)  
        * higher-order functions -- combinators -- a balanced statement about combinators -- function decorators
    * [Building Blocks](main_0_functions.md#building-blocks)   
        * composition -- partial application
    * this, aruments: [Magic Names](main_0_functions.md#magic-names)     
    * [Summary](main_0_functions.md#summary)     
* [Recipe-0: Basic Functions](main_0r_functions.md)   
   * Partial Application -- Unary -- Tap -- Maybe   
   * Once -- Left-Variadic Functions -- Compose and Pipeline   
---   
* [*Sub-1: Choice and Truthiness*](sub_1_choice.md) - Picking the Bean   
* [**Main-1: Composing and Decomposing Data**](main_1_Composing.md)   
   * [Arrays and Destructuring Arguments](main_1_Composing.md#arrays-and-destructuring-arguments)
   * [Self-Similarity](main_1_Composing.md#self-similarity)
   * [Tail Calls (and Default Arguments)](main_1_Composing.md#tail-calls-and-default-arguments)   
   * [Garbage, Garbage Everywhere](main_1_Composing.md#garbage-garbage-everywhere)   
   * [Plain Old JavaScript Objects](main_1_Composing.md#plain-old-javascript-objects)   
   * [Mutation](main_1_Composing.md#mutation)   
   * [Reassignment](main_1_Composing.md#reassignment)   
   * [Copy on Write](main_1_Composing.md#copy-on-write)   
   * [Tortoises, Hares, and Teleporting Turtles](main_1_Composing.md#tortoises-hares-and-teleporting-turtles)   
   * [Functional Iterators](main_1_Composing.md#functional-iterators)
		* iterating -- unfolding and laziness -- bonus -- caveat
   * [Making Data Out Of Functions](main_1_Composing.md#making-data-out-of-functions)   
* [Recipe-1: Data](main_1r_Composing.md)   
   * mapWith -- Flip -- Object.assign -- Why?   
---   
* [*Sub-2: Basic Strings and Quasi-Literals*](sub_2_strings.md) - A Warm Cup   
* [**Main-2: Objects and State**](main_2_objects.md) - Stir the Allongé   
   * Encapsulating State with Closures   
   * Composition and Extension   
   * This and That   
   * What Context Applies When We Call a Function?   
   * Method Decorators   
   * Summary   
* [Recipe-2: Objects, Mutations, and State](main_2r_objects.md)   
   * Memoize -- getWith -- pluckWith -- Deep Mapping   
---
* [*Sub-3: “Object-Oriented Programming”*](sub_3_oop.md) - The Coffee Factory   
* [**Main-3: Collections**](main_3_collections.md) - Served by the Pot   
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
---
* [*Sub-4: Symbols*](sub_4_symbols.md) - A Coffeehouse   
* [**Main-4: Metaobjects**](main_4_metaobjects.md) - Life on the Plantation   
   * Why Metaobjects?   
   * Mixins, Forwarding, and Delegation      
   * Later Binding    
   * Delegation via Prototypes   
   * Shared Prototypes   
---
* [*Sub-5: Impostors*](sub_5_impostors.md) - Decaffeinated   
* [**Main-5: Constructors and Classes**](main_5_constructors.md) - Finish the Cup   
   * Constructors and new   
   * Why Classes in JavaScript?   
   * Classes with class   
   * Object Methods   
   * Why Not Classes?   
   * Summary   
* [Recipe-5: Constructors and Classes](main_5r_constructors.md)   
   * Bound -- Send -- Invoke -- Fluent   
---
* [*Sub-6: Symmetry, Colour, and Charm*](sub_6_colours.md) - Colourful Mugs   
* [**Main-6: Composing Class Behaviour**](main_6_classes.md) - Con Panna   
   * Extending Classes with Mixins   
   * Functional Mixins   
   * Emulating Multiple Inheritance   
   * Preventing Property Conflicts   
   * Reducing Coupling   
---
* [**Main-7: More Decorators**](main_7_dedorators.md)   
   * Stateful Method Decorators   
   * Class Decorators beyond ES6/ECMAScript 2015   
   * Method Decorators beyond ES6/ECMAScript 2015   
   * Lightweight Traits   
* [Recipe-7: More Decorator](main_7r_dedorators.md)   
   * After Method Advice   
   * Before Method Advice   
   * Provided and Unless   
   * Method Advice   
#### :black_medium_square:[ Final Remarks: Closing Time at the Coffeeshop](book_3_closing-time.md)   

[You Dont Konw JS]: https://github.com/kiyounglee/You-Dont-Know-JS/blob/master/toc.md#top
[Functional Light JS]: https://github.com/kiyounglee/Functional-Light-JS/blob/master/manuscript/toc.md#top
[Understanding ES6]: https://github.com/kiyounglee/understandinges6/blob/master/manuscript/toc.md#top
[Javascript Allongé-6th]: https://github.com/kiyounglee/javascript-allonge-six/blob/master/myAllonge/markdown/toc.md#top
