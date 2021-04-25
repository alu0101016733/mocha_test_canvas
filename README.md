
# Unit testing with document and canvas:

##### Author: Thaddäus Haase
##### E-Mail: alu0101016733@ull.edu.es

## IMPORTANT:

If your code, be it class, function or else, looks for variables in a certain scope. Then you should provide it in that scope, in this example I will save it in the global Object to have access to it from anywhere.

## Needed packages:

[jsdom](https://www.npmjs.com/package/jsdom) : jsdom is a pure-JavaScript implementation of many web standards, notably the WHATWG DOM and HTML Standards, for use with [Node.js](https://nodejs.org/).  

[canvas](https://www.npmjs.com/package/canvas) : node-canvas is a [Cairo](https://www.cairographics.org/)-backed Canvas implementation for [Node.js](https://nodejs.org/).

### Installation as Development dependancy

```console
user@user:~/project$ npm i -D jsdom
user@user:~/project$ npm i -D canvas
```

## Setup in test file:

For a simple explanation and demonstration I will just use an example test File.

### Required imports
```javascript
// First import the modules for testing:
import { expect } from 'chai';
import { RandomWalk } from 'random-walk.js'; // my class to test

// Since we are working with html document we will need jsdom
import { JSDOM } from 'jsdom';
```

We don't need to import canvas since Node.js has access to it.

### Test Setup

Since jsdom or file read can need some time we will make use of the async code test from mocha in some of the funcions, for information please visit: [Mocha asynchronous-code](https://mochajs.org/#asynchronous-code)


#### Providing functions that are not defined in imports

When a function is not defined in the local scope, Javascript will start to look up the dependency tree to check if it is defined in another location. We will make use of that and simply provide a function with the same behavior like the function in our code expects.

For an example I will provide requestAnimationFrame for my code in the global Object (scope):

First we take a look at the documentation ([requestAnimationFrame](https://developer.mozilla.org/es/docs/Web/API/Window/requestAnimationFrame)) and then we just mirror the behavior modified to our needs for our code.

Example:

```javascript
/*
  * Since we'll be using requestAnimationFrame for some testing
  * we need to define a time funcion. in this case the time will
  * Adnvance 1000 each time it is called
  */
  let timeNext = 1000;
  const timeSimulator = () => {
    timeNext += 1000;
    return timeNext;
  }
  /*
  * providing a function to simulate requestAnimationFrame time passing
  * in the global scope. The behavior is the same as the 
  * requestAnimationFrame() from:
  * https://developer.mozilla.org/es/docs/Web/API/Window/requestAnimationFrame
  */
  global.requestAnimationFrame = (cb) => cb(timeSimulator());
```

We could to this for any function or object that we need, but since there are libraries for it and it's a lot easier and convenient we will just use those for the rest of our test code.

#### Testing code with async and await

Sometimes when we want to test code that requires for example working with files we have to be sure that the file is loaded before continuing testing, luckily mocha provides this posibility for us.

As en example I will be using a beforeEach() test function. Note that document is provided in the global scope since our class will use document and look for it there when it doesn't find it any closer.

```javascript
/*
* In my case I will create a new object for each test.
* Note the async and await statements to wait for completion
* before starting a test
*/
beforeEach(async () => {

  // We could define a simple jsdom
  // const dom = await new JSDOM('<!DOCTYPE html><html><body></body></html>');

  // open a html file and use it as a jsdom to provide document in global.
  const dom = await JSDOM.fromFile('./src/index.html');
  // we will make document accesible from global
  global.document = await dom.window.document;
});
```

## Testing (not really) private methods and methods without return

Testing private methods or functions without return value is, in my understanding not directly posible since private functions are not exported. But we can test functions with no return values, to know that they at least run fine by expecting them to return undefined. With this we can check that they will have correct Syntax.

Example:

```javascript
it('RandomWalk can be started', async () => {
  const randomWalk = await new RandomWalk(document.body);
  expect(randomWalk.start()).to.eql(undefined);
});
```