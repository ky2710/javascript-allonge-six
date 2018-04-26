# The Golden Crema: Appendices and Afterwords

![La Marzocco](images/la_marzocco.jpg)
## How to run the examples {#online}

At the time this book was written, ECMAScript 2015 was not yet widely available. All of the examples in this book were tested using either [Google Traceur Compiler], [Babel], or both. Traceur and Babel are both *transpilers*, they work by parsing ECMAScript 2015 code, then emitting valid ECMAScript-5 code that produces the same semantics.

[Google Traceur Compiler]: https://github.com/google/traceur-compiler
[Babel]: http://babeljs.io/

For example, this ECMAScript 2015 code:

    const before = (decoration) =>
      (method) =>
        function () {
          decoration.apply(this, arguments);
          return method.apply(this, arguments)
        };

Is translated into this ECMAScript-5 code:

    "use strict"

    var before = function (decoration) {
      return function (method) {
        return function () {
          decoration.apply(this, arguments);
          return method.apply(this, arguments);
        };
      };
    };
    
![The Babel "try it out" page](images/6to5.png)
    
If we make it even more idiomatic, we could write:

    const before = (decoration) =>
        (method) =>
          function (...args) {
            decoration.apply(this, args);
            return method.apply(this, args)
          };
          
And it would be "transpiled" into:

    var before = function (decoration) {
      return function (method) {
        return function () {
          for (let _len = arguments.length, args = Array(_len), _key = 0; _key < _len; _key++) {
            args[_key] = arguments[_key];
          }

          decoration.apply(this, args);
          return method.apply(this, args);
        };
      };
    };

Both tools offer an online area where you can type ECMAScript code into a web browser and see the ECMAScript-5 equivalent, and you can run the code as well. To see the result of your expressions, you may have to use the console in your web browser.

So instead of just writing:

    (() => 2 + 2)()
    
And having `4` displayed, you'd need to write:

    console.log(
      (() => 2 + 2)()
    )

And `4` would appear in your browser's development console.

You can also install the transpilers on your development system and use them with [Node] on [the command line][repl]. The care and feeding of `node` and `npm` are beyond the scope of this book, but both tools offer clear instructions for those who have already installed `node`.

[repl]: https://en.wikipedia.org/wiki/REPL "Read–eval–print loop"
[Node]: http://nodejs.org/
## Thanks!

### Daniel Friedman and Matthias Felleisen

![The Little Schemer](images/little-schemer.jpg)

*JavaScript Allongé* was inspired by [The Little Schemer] by Daniel Friedman and Matthias Felleisen. But where *The Little Schemer's* primary focus is recursion, *JavaScript Allongé's* primary focus is **functions as first-class values**.

{pagebreak}

### Richard Feynman

![QED: The Strange Theory of Light and Matter](images/qed.jpg)

Richard Feynman's [QED] was another inspiration: A book that explains Quantum Electrodynamics and the "Sum of the Histories" methodology using the simple expedient of explaining how light reflects off a mirror, and showing how most of the things we think are happening--such as light travelling on a straight line,  the angle of reflection equalling the angle of refraction, or that a beam of light only interacts with a small portion of the mirror, or that it reflects off a plane--are all wrong. And everything is explained in simple, concise terms that build upon each other logically.

[JavaScript]: https://developer.mozilla.org/en-US/docs/JavaScript
[The Little Schemer]: http://www.amzn.com/0262560992?tag=raganwald001-20
[QED]: http://www.amzn.com/0691125759?tag=raganwald001-20

{pagebreak}
## Reading JavaScript Allongé on Kindle

JavaScript Allongé has over 400 pages and many photographs. For this reason, the `.mobi` version of the book is too big to be sent to your Kindle via email, and that is the feature that LeanPub uses when you purchase the book.

![Send to Kindle](images/kindle.png)

So, if you wish to read JavaScript Allongé on your Kindle:

- Download it to your Windows or OS X device (a/k/a "PC" or "Macintosh").
- Use [Send to Kindle for PC](http://www.amazon.com/gp/sendtokindle/pc) or [Send to Kindle for Mac](http://www.amazon.com/gp/sendtokindle/mac) to send it to the Kindle.
- From time to time while editing, an uncompressed image sneaks into the manuscript. When this happens, the `.mobi` may exceed the 50MB limit for the Send to Kindle desktop application. If this happens, please attach your Kindle to your computer with a USB cable and synchronize directly.

Thank you!
## Copyright Notice

The original words in this book are (c) 2012-2015, Reginald Braithwaite. All rights reserved.
### images

* The picture of the author is (c) 2008, [Joseph Hurtado](http://www.flickr.com/photos/trumpetca/), All Rights Reserved.
* [Cover image](http://www.flickr.com/photos/avlxyz/4907262046) (c) 2010, avlxyz. [Some rights reserved][by-sa].
* [Double ristretto menu](http://www.flickr.com/photos/digitalcolony/5054568279/) (c) 2010, Michael Allen Smith. [Some rights reserved][by-sa].
* [Short espresso shot in a white cup with blunt handle](http://www.flickr.com/photos/everydaylifemodern/1353570874/) (c) 2007, EVERYDAYLIFEMODERN. [Some rights reserved][by-nd].
* [Espresso shot in a caffe molinari cup](http://www.flickr.com/photos/everydaylifemodern/434299813/) (c) 2007, EVERYDAYLIFEMODERN. [Some rights reserved][by-nd].
* [Beans in a Bag](http://www.flickr.com/photos/the_rev/2295096211/) (c) 2008, Stirling Noyes. [Some Rights Reserved][by].
* [Free Samples](http://www.flickr.com/photos/thedigitelmyr/6199419022/) (c) 2011, Myrtle Bech Digitel. [Some Rights Reserved][by-sa].
* [Free Coffees](http://www.flickr.com/photos/sagamiono/4391542823/) image (c) 2010, Michael Francis McCarthy. [Some Rights Reserved][by-sa].
* [La Marzocco](http://www.flickr.com/photos/digitalcolony/3924227011/) (c) 2009, Michael Allen Smith. [Some rights reserved][by-sa].
* [Cafe Diplomatico](http://www.flickr.com/photos/15481483@N06/6231443466/) (c) 2011, Missi. [Some rights reserved][by-sa].
* [Sugar Service](http://www.flickr.com/photos/tjgfernandes/2785677276/) (c) 2008 Tiago Fernandes. [Some rights reserved][by].
* [Biscotti on a Rack](http://www.flickr.com/photos/kirstenloza/4805716699/) (c) 2010 Kirsten Loza. [Some rights reserved][by].
* [Coffee Spoons](http://www.flickr.com/photos/jenny-pics/5053954146/) (c) 2010 Jenny Downing. [Some rights reserved][by].
* [Drawing a Doppio](http://www.flickr.com/photos/33388953@N04/4017985434/) (c) 2008 Osman Bas. [Some rights reserved][by].
* [Cupping Coffees](http://www.flickr.com/photos/tangysd/5953453156/) (c) 2011 Dennis Tang. [Some rights reserved][by-sa].
* [Three Coffee Roasters](http://www.flickr.com/photos/digitalcolony/4000837035/) (c) 2009 Michael Allen Smith. [Some rights reserved][by-sa].
* [Blue Diedrich Roaster](http://www.flickr.com/photos/digitalcolony/4309812256/) (c) 2010 Michael Allen Smith. [Some rights reserved][by-sa].
* [Red Diedrich Roaster](http://www.flickr.com/photos/bike/3237859728/) (c) 2009 Richard Masoner. [Some rights reserved][by-sa].
* [Roaster with Tree Leaves](http://www.flickr.com/photos/lacerabbit/2102801319/) (c) 2007 ting. [Some rights reserved][by-nd].
* [Half Drunk](http://www.flickr.com/photos/nalundgaard/4785922266/) (c) 2010 Nicholas Lundgaard. [Some rights reserved][by-sa].
* [Anticipation](http://www.flickr.com/photos/paulmccoubrie/6828131856/) (c) 2012 Paul McCoubrie. [Some rights reserved][by-nd].
* [Ooh!](http://www.flickr.com/photos/mikecogh/7676649034/) (c) 2012 Michael Coghlan. [Some rights reserved][by-sa].
* [Intestines of an Espresso Machine](http://www.flickr.com/photos/yellowskyphotography/5641003165/) (c) 2011 Angie Chung. [Some rights reserved][by-sa].
* [Bezzera Espresso Machine](http://www.flickr.com/photos/andynash/6204253236/) (c) 2011 Andrew Nash. [Some rights reserved][by-sa].
*[Beans Ripening on a Branch](http://www.flickr.com/photos/28705377@N04/5306009552/) (c) 2008 John Pavelka. [Some rights reserved][by].
* [Cafe Macchiato on Gazotta Della Sport](http://www.flickr.com/photos/shavejonathan/2343081208/) (c) 2008 Jon Shave. [Some rights reserved][by].
* [Jars of Coffee Beans](http://www.flickr.com/photos/ilovememphis/7103931235/) (c) 2012 Memphis CVB. [Some rights reserved][by-nd].
* [Types of Coffee Drinks](http://www.flickr.com/photos/mikecogh/7561440544/) (c) 2012 Michael Coghlan. [Some rights reserved][by-sa].
* [Coffee Trees](http://www.flickr.com/photos/dtownsend/6171015997/) (c) 2011 Dave Townsend. [Some rights reserved][by-sa].
* [Cafe do Brasil](http://www.flickr.com/photos/93425126@N00/313053257/) (c) 2003 Temporalata. [Some rights reserved][by-sa].
* [Brown Cups](http://www.flickr.com/photos/digitalcolony/2833809436/) (c) 2007 Michael Allen Smith. [Some rights reserved][by-sa].
* [Mirage](http://www.flickr.com/photos/citizenhelder/5006498068/) (c) 2010 Mira Helder. [Some rights reserved][by].
* [Coffee Van with Bullet Holes](http://www.flickr.com/photos/joncrel/237026246/) (c) 2006 Jon Crel. [Some rights reserved][by-nd].
* [Disassembled Elektra](http://www.flickr.com/photos/nalundgaard/3163852170/) (c) 2009 Nicholas Lundgaard. [Some rights reserved][by-sa].
* [Nederland Buffalo Bills Coffee Shop](http://www.flickr.com/photos/47000103@N05/6525288841/) (c) 2009 Charlie Stinchcomb. [Some rights reserved][by-sa].
* [For the love of coffee](http://www.flickr.com/photos/lotzman/978418891/) (c) 2007 Lotzman Katzman. [Some rights reserved][by].
* [Saltspring Processing Facility Pictures](http://www.flickr.com/photos/kk/sets/72157626168201654/with/5484839102/) (c) 2011 Kris Krug. [Some rights reserved][by-sa].
* [Coffee and Mathematics](https://www.flickr.com/photos/kellan/434503323) (c) 2007 [Some rights reserved][by].
* [Coffee and a Book](https://www.flickr.com/photos/whitneyinchicago/3835218626) (c) 2009 [Some rights reserved][by].
* [Stacked Coffee Cups](https://www.flickr.com/photos/sankarshan/5165312159) (c) 2010 Sankarshan Sen. [Some rights reserved][by-sa].
* [Coffee Cow](https://www.flickr.com/photos/candy-s/7619358284) (c) 2012 [Candy Schwartz](https://www.flickr.com/photos/candy-s/) [Some rights reserved][by].
* [CERN Coffee](https://www.flickr.com/photos/lorentey/22193876) (c) 2005 [Karoly Lorentey](https://www.flickr.com/photos/lorentey/) [Some rights reserved][by].
* [Coffee Labels](https://www.flickr.com/photos/kk/5484876862) (c) 2011 Kris Krüg [Some rights reserved][by-sa].
* [banco do café](https://www.flickr.com/photos/f_mafra/2956649121) (c) 2008 Fernando Mafra [Some rights reserved][by-sa].
* [coffee pots](https://www.flickr.com/photos/jforth/3360599750/) (c) 2009 Jonas Forth [Some rights reserved][by-nd].
* [5 Barrel Roaster](https://www.flickr.com/photos/dlytle/8720139854) (c) 2013 David Lytle [Some rights reserved][by].
* [Pantone mugs](https://www.flickr.com/photos/joebehr/5504285781) (c) 2011 Joe Wolf [Some rights reserved][by-nd].
* [Coffee and Chess](https://www.flickr.com/photos/adders/8372085101) (c) 2013 Adam Tinworth [Some rights reserved][by-nd].
* [Vac Pot Upper Chamber](https://www.flickr.com/photos/digitalcolony/2843767532) (c) 2007 Michael Allen Smith [Some rights reserved][by-sa].
* [Decaf espresso](https://www.flickr.com/photos/arisvrakas/4217869291) (c) 2009 Aris Vrakas [Some rights reserved][by].
* [Con Panna](https://www.flickr.com/photos/vscript/8708520929) (c) 2013 Vee Satayamas [Some rights reserved][by].
* [Tiny's Coffeehouse](https://www.flickr.com/photos/peterme/1271652) (c) 2004 Peter Merholz [Some rights reserved][by-sa].
* [Thinking about programming](https://www.flickr.com/photos/renaud-camus/6165559492) (c) 2011 Renaud Camus [Some rights reserved][by].
* [Biscotti og kaffe](https://www.flickr.com/photos/cyclonebill/2606398721) (c) 2008 [Some rights reserved][by-sa].
* [Espresso, Empty](https://www.flickr.com/photos/tillwe/8154272083) (c) 2012 Till Westermayer [Some rights reserved][by-sa].
* [The End](https://www.flickr.com/photos/peddhapati/11671457605) (c) 2013 peddhapati [Some rights reserved][by].
* [The Future of Coffee is Black](https://www.flickr.com/photos/mjaysplanet/8416343475) (c)2013 mjaysplanet [Some rights reserved][by-sa].

[by-sa]: http://creativecommons.org/licenses/by-sa/2.0/deed.en
[by-nd]: http://creativecommons.org/licenses/by-nd/2.0/deed.en
[by]: http://creativecommons.org/licenses/by/2.0/deed.en
## About The Author

When he's not shipping JavaScript, Ruby, CoffeeScript and Java applications scaling out to millions of users, Reg "Raganwald" Braithwaite has authored [libraries][lib] for JavaScript, CoffeeScript, and Ruby programming such as Allong.es, Method Combinators, Katy, JQuery Combinators, YouAreDaChef, andand, and others.

[lib]: http://github.com/raganwald

He writes about programming on "[Raganwald][raganwald]," as well as general-purpose ruminations on "[Braythwayt Dot Com][braythwayt]".

[raganwald]: http://raganwald
[braythwayt]: http://braythwayt.com

### contact

Twitter: [@raganwald](https://twitter.com/raganwald)
Email: [reg@braythwayt.com](mailto:reg@braythwayt.com)

![Reg "Raganwald" Braithwaite ](images/reg2.jpg)
