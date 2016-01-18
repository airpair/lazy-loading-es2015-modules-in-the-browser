Synopsis
> Writing and delivering ES2015 modules to the browser has never been easier, thanks to Babel and System.js. Learn how to load ES2015 modules either during the page load or at a later time, a.k.a. lazy-loaded.

ES2015 is already production-ready through transpilers such as **Babel**<sup>[Babel](https://babeljs.io/)</sup>. Now you no longer need to fight the AMD vs CommonJS war - described on my article **The mind-boggling universe of JavaScript Module strategies**<sup>[The mind-boggling universe of JavaScript Module strategies](https://www.airpair.com/javascript/posts/the-mind-boggling-universe-of-javascript-modules)</sup> - since you can simply write ES2015 modules, have them transpiled to ES5 and delivered to the browser, which can either happen synchronously (just like CommonJS modules) or asynchronously (just like AMD modules).

This article demonstrates how to load ES2015 modules synchronously (during page load) and asynchronously (lazy-load) using System.js<sup>[systemjs on Github](https://github.com/systemjs/systemjs)</sup> over Babel.

## Page-loaded code vs Lazy-loaded code

When developing JavaScript code to be executed on the browser, you always have to decide WHEN do you want it to be executed.

You certainly have some code to run during the **page load**, as for instance the structural setup of a SPA using frameworks as Angular, Ember, Backbone or React. Such code must be referenced on the main HTML document returned to the browser after a page request, most likely through a ```<script>``` tag.

On the other hand, you might have some other code that should only be executed if certain conditions happen, as for instance an overlay that would only appear after the user clicks on a button. Watch this: if the user never clicks on that button, the code will never be executed. Hence, it is not needed during the page load and it can be left out of the page load code, so that it would only be downloaded on demand, when the user actually clicks on that button for the first time. 

Such approach of asynchronously loading deferred code, or **lazy-loading**, is practically certain to substantially improve the page performance, in terms of getting smaller page load time and Speed Index<sup>[Speed Index](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index)</sup>.

In order to learn more about the performance impacts on Page-load time code vs Lazy-loaded code, check out my article **Leveling up: Simple steps to optimize the Critical Rendering Path**<sup>[Leveling up: Simple steps to optimize the Critical Rendering Path](https://www.airpair.com/javascript/posts/the-tipping-point-of-clientside-performance)</sup>.

## The pitfalls of AMD

The AMD standard was created for asynchronous module loading on the browser, being one of the first successful alternatives to the spaghetti mess of global JavaScript files scattered around your page. According to the Require.js documentation<sup>[Require.js documentation — Why AMD?](http://requirejs.org/docs/whyamd.html#amd)</sup>:

> The AMD format comes from wanting a module format that was better than today’s “write a bunch of script tags with implicit dependencies that you have to manually order” and something that was easy to use directly in the browser.

It is based on empowering the Module Design Pattern<sup>[Learning JavaScript Design Patterns: The Module Pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)</sup> with a module loader, dependency injection, alias resolution and asynchronous capabilities. One of it main usages is to perform **lazy-loading** of modules.

Despite being a formidable idea, it brings some inherent complexity: the need to understand live module timelines, which wasn't necessary before. This means that developers need to know when each asynchronous module is expected to do its work. Failing to understand such timelines may lead to situations that may work some times and may not work some other times, due to race conditions, which can be quite difficult to debug.

Because of things like that, projects ended up failing to properly implement AMD (I was part of 2 projects that have abandoned AMD simply because no one was able to fix certain production bugs caused by poor AMD understanding and implementation). This way, AMD lost quite a bit of its momentum and traction, unfortunately.

In order to learn more about AMD pitfalls, check out **Moving Past RequireJS**<sup>[Moving Past RequireJS](http://benmccormick.org/2015/05/28/moving-past-requirejs/)</sup>.

## ES2015 modules 101

Before we go any further, let's go over ES2015 modules. If you are already familiar with them, you may use a quick refresher.

Modules have been finally adopted as an official part of the JavaScript language in ES2015. Its Modules specification is powerful yet simple to grasp, and piggybacks on the main ideas of CommonJS modules. 

### Scope

Basically, a ES2015 module will live in its own file. All its "globals" variables will be scoped to just this file. Modules can export data and also import other modules.

### Exporting and importing

Export a ES2015 module's interface through the keyword ```export``` before each item you want to export (a variable, function or class). In the following example, we are exporting ```Dog``` and ```Wolf```:

```javascript
// zoo.js
var getBarkStyle = function(isHowler) {
  return isHowler? 'woooooow!': 'woof, woof!';
};

export class Dog {
  constructor(name, breed) {
    this.name = name;
    this.breed = breed;
  }

  bark() {
    return `${this.name}: ${getBarkStyle(this.breed === 'husky')}`;
  };
}

export class Wolf {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `${this.name}: ${getBarkStyle(true)}`;
  };
}
```

Let's see how to import this module in a Mocha/Chai unit test, using the syntax ```import <object> from <path>```. As for ```<object>``` we can take advantage of another ES2015 feature here - *destructuring objects*<sup>[Destructuring and parameter handling in ECMAScript 6](http://www.2ality.com/2015/01/es6-destructuring.html)</sup> - where we can basically create our own object from items being imported from a module. We can then decide to just import ```expect``` from ```chai``` as well as ```Dog``` and ```Wolf``` from ```Zoo```.

```javascript
// zoo_spec.js
import { expect } from 'chai';
import { Dog, Wolf } from '../src/zoo';

describe('the zoo module', () => {
  it('should instantiate a regular dog', () => {
    var dog = new Dog('Sherlock', 'beagle');
    expect(dog.bark()).to.equal('Sherlock: woof, woof!');
  });

  it('should instantiate a husky dog', () => {
    var dog = new Dog('Whisky', 'husky');
    expect(dog.bark()).to.equal('Whisky: woooooow!');
  });

  it('should instantiate a wolf', () => {
    var wolf = new Wolf('Direwolf');
    expect(wolf.bark()).to.equal('Direwolf: woooooow!');
  });
});
```

### Default

If you only have one item to export, you can use ```export default``` to export your item as an object instead of exporting a container object with your item inside:

```javascript
// cat.js
export default class Cat {
  constructor(name) {
    this.name = name;
  }

  meow() {
    return `${this.name}: You gotta be kidding that I'll obey you, right?`;
  }
}
```

Importing default modules is simpler, as object destructuring is no longer needed. You can simply directly import the item from the module.

```javascript
// cat_spec.js
import { expect } from 'chai';
import Cat from '../src/cat';

describe('the cat module', () => {
  it('should instantiate a cat', () => {
    var cat = new Cat('Bugsy');
    expect(cat.meow()).to.equal('Bugsy: You gotta be kidding that I\'ll obey you, right?');
  });
});
```

In order to learn more about ES2015 modules, check out **Exploring ES6 — Modules**<sup>[Exploring ES6 book — Modules](http://exploringjs.com/es6/ch_modules.html)</sup>.

## ES2015 Module Loader and System.js

As surprising as it can be, ES2015 doesn't actually have a module loader specification. There _was_ a valid and accepted specification for a dynamic module loader<sup> [es6-module-loader on Github](https://github.com/ModuleLoader/es6-module-loader)</sup>, from which **System.js** was created. However, such specification was removed and a new loader specification is being proposed by WhatWG<sup>[WhatWG's new loader proposal](https://whatwg.github.io/loader/)</sup>, hopefully for ES2016.
    
Nevertheless, System.js is considered by many the most accurate module loader implementation to date. It loads ES2015 modules, AMD, CommonJS and global scripts in the browser and NodeJS. It provides an asynchronous module loader (to pair with Require.js) and ES2015 transpiling through Babel, Traceur<sup>[traceur-compiler on Github](https://github.com/google/traceur-compiler)</sup> or TypeScript<sup>[Typescript](http://www.typescriptlang.org/)</sup>.

System.js implements asynchronous module loading using a Promises-based API. This is a very powerful and flexible approach, since promises can be chained and combined, so for instance if you want to load multiple modules in parallel, you can use ```Promises.all``` and just fire your listener when all the promises have been resolved.

## Importing modules synchronously and asynchronously

In order to illustrate the loading of modules in both synchronous and asynchronous fashion I've created a working sample project, which will load our ```Cat``` module synchronously during the page load, and lazy-load the ```Zoo``` module asynchronously when the user clicks on a button. The code is available on a Github project<sup>[lazy-load-es2015-systemjs on Github](https://github.com/tiagorg/lazy-load-es2015-systemjs)</sup>.

This is the main self-explanatory code which illustrate the synchronous loading through ```import```:

```javascript
// main.js
// Importing Cat module synchronously
import Cat from 'cat';

// DOM content node
let contentNode = document.getElementById('content');

// Rendering cat
let myCat = new Cat('Bugsy');
contentNode.innerHTML += myCat.meow();
```

Now, the part for the asynchronous loading through ```System.import()```:

```javascript
// Button to lazy load Zoo
contentNode.innerHTML += `<p><button id='loadZoo'>Lazy load <b>Zoo</b></button></p>`;

// Listener to lazy load Zoo
document.getElementById('loadZoo').addEventListener('click', e => {

  // Importing Zoo module asynchronously
  System.import('zoo').then(Zoo => {

  	// Rendering dog
	let myDog = new Zoo.Dog('Sherlock', 'beagle');
	contentNode.innerHTML += `${myDog.bark()}`;

	// Rendering wolf
	let myWolf = new Zoo.Wolf('Direwolf');
	contentNode.innerHTML += `<br/>${myWolf.bark()}`;
  });
});
```

## Voilà

When the page is first loaded, the only modules which are loaded are ```Cat``` and ```Main```:

![Page-loaded modules](https://cloud.githubusercontent.com/assets/764487/12385934/cdcd65c6-bd75-11e5-9cf1-2d3cb49a4abb.png "Page-loaded modules")

Once the user clicks on the button, the ```Zoo``` module is then loaded:

![Lazy-loaded modules](https://cloud.githubusercontent.com/assets/764487/12385935/cf92bdf2-bd75-11e5-83c0-2f773aed7edb.png "Lazy-loaded modules")

## Allons-y!

Mastering the art of only loading essential modules during the page load and effectively lazy-loading all the other modules is extremely helpful for your page performance. You can start savvy loading your ES2015 modules right now with System.js, and soon with the official Module Loader spec.

Photo credit: <a href="https://www.flickr.com/photos/25171569@N02/9822344846/">jjjj56cp</a> via <a href="http://visualhunt.com">Visual Hunt</a> / <a href="http://creativecommons.org/licenses/by-nc-sa/2.0/">CC BY-NC-SA</a>
