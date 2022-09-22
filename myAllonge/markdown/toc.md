<a name="top"></a>
***[YDK JS]** -- **[YDK JS Yet]** -- **[Functional Light JS]** -- **[Understanding ES6]** -- **[JS Allongé-6th]***   

[YDK JS]: https://github.com/ky27100/You-Dont-Know-JS/blob/1st-ed/toc.md#top
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
	* values are expressions 
	* values and identity  
* [*Sub-0: Basic Numbers*](sub_0_numbers.md) 
* [**0 : Basic Functions**](main_0_functions.md)
	* Function basic : As Little As Possible About Functions, But No Less
	* Argument & Parameter : Ah. I’d Like to Have an Argument, Please.   
	* Closures and Scope   
	* const and lexical scope : That Constant Coffee Craving   
	* Naming Functions     
	* Combinators & Function Decorators  
	* Building Blocks : Compositon, Partial application   
	* Magic Names : this, arguments     
	* Summary
	* [Recipe-0: Basic Functions](main_0r_functions.md)
		* Partial Application / Unary / Tap / Maybe / Once 
		* Left-Variadic Functions / Compose and Pipeline
* [*Sub-1: Choice and Truthiness*](sub_1_choice.md)  
* [**1 : Composing & Decomposing Data**](main_1_Composing.md)
	* Arrays and Destructuring Arguments
	* Self-Similarity : List(Array)
	* Tail Calls (and Default Arguments)   
	* Garbage, Garbage Everywhere   
	* Plain Old JavaScript Objects 
	* Mutation
	* Reassignment
	* Copy on Write(Copy on Read)   
	* Tortoises, Hares, and Teleporting Turtles   
	* Functional Iterators
	* Making Data Out Of Functions   
	* [Recipe-1: Data](main_1r_Composing.md)   
		* mapWith / Flip / Object.assign / Why?     
* [*Sub-2: Basic Strings and Quasi-Literals*](sub_2_strings.md)   
* [**2 : Objects & State**](main_2_objects.md)
	* Encapsulating State with Closures
	* :boom:Composition and Extension
	* This and That 
	* What Context Applies When We Call a Function?
	* Method Decorators
	* Summary
	* [Recipe-2: Objects, Mutations, and State](main_2r_objects.md)   
		* Memoize / getWith / pluckWith / Deep Mapping   
* [*Sub-3: “Object-Oriented Programming”*](sub_3_oop.md)  
* [**3 : Collections**](main_3_collections.md)
	* Iteration and Iterables   
	* Generating Iterables   
	* Lazy and Eager Collections   
	* Interlude: The Carpenter Interviews for a Job   
	* Interactive Generators   
	* Basic Operations on Iterables   
* [*Sub-4: Symbols*](sub_4_symbols.md)   
* [**4 : Metaobjects**](main_4_metaobjects.md#life-on-the-plantation-metaobjects)
	* Why Metaobjects? 
	* Mixins, Forwarding, and Delegation       
	* Later Binding     
	* Delegation via Prototypes    
	* Shared Prototypes    
* [*Sub-5: Impostors*](sub_5_impostors.md)   
* [**5 : Constructors & Classes**](main_5_constructors.md#finish-the-cup-constructors-and-classes)
	* Constructors and `new`    
	* Why Classes in JavaScript?    
	* Classes with class    
	* Object Methods    
	* Why Not Classes?    
	* Summary 
	* [Recipe-5: Constructors and Classes](main_5r_constructors.md)   
		* Bound / Send / Invoke / Fluent   
* [*Sub-6: Symmetry, Colour, and Charm*](sub_6_colours.md#colourful-mugs-symmetry-colour-and-charm)   
* [**6 : Composing Class Behaviour**](main_6_classes.md#con-panna-composing-class-behaviour)
	* Extending Classes with Mixins    
	* Functional Mixins    
	* Emulating Multiple Inheritance   
	* Preventing Property Conflicts    
	* Reducing Coupling    
* [**7 : More Decorators**](main_7_dedorators.md#more-decorators)   
	* Stateful Method Decorators    
	* Class Decorators beyond ES6/ECMAScript 2015    
	* Method Decorators beyond ES6/ECMAScript 2015    
	* Lightweight Traits    
	* [Recipe-7: More Decorator](main_7r_dedorators.md)   
		* After Method Advice / Before Method Advice / Provided and Unless / Method Advice      
* [**Final Remarks: Closing Time at the Coffeeshop**](book_3_closing-time.md#closing-time-at-the-coffeeshop-final-remarks)   
	* javascript beyond es6/ecmascript 2015 
	* the lightweight way    
