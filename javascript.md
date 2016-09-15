## Data Types

```javascript
  // Primitive (Number, String, Boolean, Undefined, Null). Immutable.
  var foo = 123;
  
  // Object. Mutable.
  var foo = {};
  
  // Access object properties
  foo.val;
  foo['val'];
  
  // References (i.e. value is a pointer)
  var foo = { val: 123 };
  var bar = foo;
  bar.val = 456;
  assertTrue(foo.val === 456);  // true

  // Literal declaration
  var foo = {
    hello: 'world'
  };
  
  // Constructor declaration
  var foo = new Object();
  foo.hello = 'world';
  
  // Constructor pattern
  function Hello(message) {
    this.message = message;
  }
  var foo = Hello('world');
  
  // Property exists on object (not inherited)
  var foo = { val: 123 };
  assertTrue('val' in foo);  // true
  assertTrue('toString' in foo);  // false
  assertTrue(foo.hasOwnProperty('val'));  // true
  assertTrue(foo.hasOwnProperty('toString'));  // false
  
  // Enumerate properties of object. Inherited properties not specifically enumerable are not revealed.
  for (var prop in foo) {
    // val
  }
  
  // Delete object property. Returns true even if failed or non-existant?!
  delete foo.val;
  
  // Prototype inheritance (i.e. parent). Set when new object is created. Object.prototype is top of chain.
  function Hello(message) {
    this.message = message;
  }
  Hello.prototype.print = function() { console.log(this.message); }
  var myHello = new Hello('world');   // prototype object is Hello.prototype
  myHello.print();      // world
  
  // Inheritance of objects
  function Plant() {
    this.eat = function() {};
  };
  function Fruit(name) {
    this.name = name;
  };
  Fruit.prototype = new Plant();
  var banana = new Fruit('banana');
  banana.eat();
  
  // Alternative non-standard syntax for prototype. Avoid.
  foo.__proto__.print();
  
  // Object.create() --> set prototype?
  
  // Delete inherited property
  delete foo.prototype.val;
  
  // Convert object to string
  fooString = JSON.stringify(foo);
  
  // Convert JSON to object
  foo = JSON.parse(fooString);
```
