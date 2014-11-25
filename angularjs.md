## Observations

* For ansyc calls, use promises not callbacks
  * Mature, tested pattern for lifecycle of an asynchronous task
  * Bluebird better than (Kris Kowall's) Q. Almost as fast as callbacks...and better features. 5x slower in debug mode.
  * Advantages: catches errors, supported by jQuery, run once (once resolved), nodify/denodify existing functions
  * Generators: new to ES6, function*, function that acts like an iterator, executes one statement at a time, .next(), yeild passed in objects. Libraries: [suspend](https://github.com/jmar777/suspend), [genny](https://github.com/spion/genny)
  * Thunk: a partially-evaluated function that accepts a single callback argument. Library: [co](https://github.com/tj/co)
```javascript
// A thunk
function foo(callback) {
   // read file
}
```
* Modules relate to functionality and contain various classes (e.g. controller, service)
* Directives are HTML markup attached via semantic tags (e.g. <input validate-mandatory... />)
* Verbose
* Bower for front-end dependencies (e.g. jQuery), node_modules for back-end dependencies (e.g. xml2json)

## Tools

* [Plunker](http://plnkr.co/): Edit and run AngularJS code online
* [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk): Chrome extension

## References

