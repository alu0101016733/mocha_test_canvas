
# Unit testing with document and canvas:

## Needed packages:

[jsdom](https://www.npmjs.com/package/jsdom) : jsdom is a pure-JavaScript implementation of many web standards, notably the WHATWG DOM and HTML Standards, for use with [Node.js](https://nodejs.org/).  
[canvas](https://www.npmjs.com/package/canvas) : node-canvas is a [Cairo](https://www.cairographics.org/)-backed Canvas implementation for [Node.js](https://nodejs.org/).

### Installation as Development dependancy

```console
user@user:~/project$ npm i -D jsdom
user@user:~/project$ npm i -D canvas
```

## Setup in test file:

For a simple explanation and demostration I will just use an example test File for the current project.

### Required imports
```javascript
// First import the modules for testing:
import { expect } from 'chai';
import { RandomWalk } from 'random-walk.js';

// Since we are working with html document we will need jsdom
import { JSDOM } from 'jsdom';
```

We don't need to import canvas sine Node.js has access to it.

### Test Setup

Since jsdom or file read can need some time we will make use of the async code test from mocha: [Mocha asynchronous-code](https://mochajs.org/#asynchronous-code)

```javascript
// creating
describe('Random Walk', () => {
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
  * in the global environment. The behavior is the same as the 
  * requestAnimationFrame() from:
  * https://developer.mozilla.org/es/docs/Web/API/Window/requestAnimationFrame
  */
  global.requestAnimationFrame = (cb) => cb(timeSimulator());

  // object variable i will be using for testing purposes
  let randomWalk;

  /*
  * In my case I will create a new object for each test.
  * Note the async and await statements to wait for completion
  * before starting a test
  */
  beforeEach(async () => {
    /*
    * to use a jsdom from file we can uncomment the following line
    * const dom = await JSDOM.fromFile('./src/index.html');
    */

    // We define a simple jsdom
    const dom = await new JSDOM('<!DOCTYPE html><html><body style="height:600; width:600;"></body></html>');
    // we will make document accesible from global
    global.document = await dom.window.document;
    randomWalk = await new RandomWalk(document.body, 2, 100);
  });

  // Now for the testing functions
  describe('RandomWalk object tests', () => {
    /*
    * Since we work with async methods we should add the async keyword
    * to our tests, since in the example test I don't use any await currently
    * it won't hurt to define it in case of adding or overwriding wom variable
    * locally with await.
    */
    it('RandomWalk is an object', async () => {
      expect(randomWalk).to.be.an('Object');
    });

    /*
    * Testing private methods or functions without return value is, in my
    * understanding not directly posible since private functions are not
    * exported. But we can test functions with no return values to at least
    * run fine by expecting them to return undefined. With this we can at least
    * check that they will have correct Syntax.
    * Example:
    */
    it('RandomWalk can be started', async () => {
      expect(randomWalk.start()).to.eql(undefined);
    });
    
  });
});
```


# Datos de mi setup:
#### Coverage:
npm i -D c8