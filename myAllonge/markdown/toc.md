<a name="top"></a>
***[You Dont Konw JS]** --- **[Functional Light JS]** --- **[Understanding ES6]** --- **[JS Allongé]***

[You Dont Konw JS]: https://github.com/kiyounglee/You-Dont-Know-JS/blob/master/toc.md#top
[Functional Light JS]: https://github.com/kiyounglee/Functional-Light-JS/blob/master/manuscript/toc.md#top
[Understanding ES6]: https://github.com/kiyounglee/understandinges6/blob/master/manuscript/toc.md#top
[JS Allongé]: https://github.com/kiyounglee/javascript-allonge-six/blob/master/myAllonge/markdown/toc.md#top

## [JavaScript Allongé-6th](#javascript-allong%C3%A9-6th-detailed)
*[Prefaces](book_1_preface.md) / [Prelude](book_2_prelude.md) / [Remarks](book_3_closing-time.md) / [Appendices](book_4_appendices.md) - Nov 3, 2017(513p) / [Leanpub](https://leanpub.com/javascriptallongesix/read#leanpub-auto-about-javascript-allong) - Nov 3, 2017(513p) - [By Reg Braithwaite](https://github.com/raganwald)* 
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
  

## [JavaScript Allongé-6th](#top) *(detailed)*
*[Prefaces](book_1_preface.md) / [Prelude](book_2_prelude.md) / [Leanpub](https://leanpub.com/javascriptallongesix/read#leanpub-auto-about-javascript-allong) - Nov 3, 2017(513p) - [By Reg Braithwaite](https://github.com/raganwald)*    
#### :black_medium_square:[ Prelude: Values and Expressions over Coffee](book_2_prelude.md)        
    * values are expressions     * values and identity  
* [*Sub-0: Basic Numbers* - A Rich Aroma](sub_0_numbers.md)   
* [**Main-0: Basic Functions** - The first sip](main_0_functions.md#the-first-sip-basic-functions)   
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
* [*Sub-1: Choice and Truthiness* - Picking the Bean](sub_1_choice.md)   
* [**Main-1: Composing and Decomposing Data**](main_1_Composing.md)   
   * Arrays and Destructuring Arguments   
   * Self-Similarity   
   * Tail Calls (and Default Arguments)   
   * Garbage, Garbage Everywhere   
   * Plain Old JavaScript Objects   
   * Mutation   
   * Reassignment   
   * Copy on Write   
   * Tortoises, Hares, and Teleporting Turtles   
   * Functional Iterators   
   * Making Data Out Of Functions   
* [Recipe-1: Data](main_1r_Composing.md)   
   * mapWith -- Flip -- Object.assign -- Why?   
---   
* [*Sub-2: Basic Strings and Quasi-Literals* - A Warm Cup](sub_2_strings.md)   
* [**Main-2: Objects and State** - Stir the Allongé](main_2_objects.md)   
   * Encapsulating State with Closures   
   * Composition and Extension   
   * This and That   
   * What Context Applies When We Call a Function?   
   * Method Decorators   
   * Summary   
* [Recipe-2: Objects, Mutations, and State](main_2r_objects.md)   
   * Memoize -- getWith -- pluckWith -- Deep Mapping   
---
* [*Sub-3: “Object-Oriented Programming”* - The Coffee Factory](sub_3_oop.md)   
* [**Main-3: Collections** - Served by the Pot](main_3_collections.md)   
    * Iteration and Iterables   
    * Generating Iterables   
    * Lazy and Eager Collections   
    * Interlude: The Carpenter Interviews for a Job   
    * Interactive Generators   
    * Basic Operations on Iterables   
---
* [*Sub-4: Symbols* - A Coffeehouse](sub_4_symbols.md)   
* [**Main-4: Metaobjects** - Life on the Plantation](main_4_metaobjects.md)   
   * Why Metaobjects?   
   * Mixins, Forwarding, and Delegation      
   * Later Binding    
   * Delegation via Prototypes   
   * Shared Prototypes   
---
* [*Sub-5: Impostors* - Decaffeinated](sub_5_impostors.md)   
* [**Main-5: Constructors and Classes** - Finish the Cup](main_5_constructors.md)   
   * Constructors and new   
   * Why Classes in JavaScript?   
   * Classes with class   
   * Object Methods   
   * Why Not Classes?   
   * Summary   
* [Recipe-5: Constructors and Classes](main_5r_constructors.md)   
   * Bound -- Send -- Invoke -- Fluent   
---
* [*Sub-6: Symmetry, Colour, and Charm* - Colourful Mugs](sub_6_colours.md)   
* [**Main-6: Composing Class Behaviour** - Con Panna](main_6_classes.md)   
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

