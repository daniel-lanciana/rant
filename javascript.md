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
```

## Prototype inheritance (i.e. parent object reference)

Chained references to the parent object for the purposes of inheritance. Object.prototype is the top of the chain.

```javascript  
  // Example
  function Plant() {
    this.eat = function() { console.log('yuck'); };
    this.cook = function() { console.log('cook'); };
  };
  function Fruit(name) {
    this.name = name;
    this.eat = function() { console.log('yum'); };
  };
  Fruit.prototype.print = function() { console.log(this.name); }
  Fruit.prototype = new Plant();
  var banana = new Fruit('banana');
  banana.eat();             // yum
  banana.prototype.eat();   // yuck
  banana.print();           // banana
  banana.cook();            // cook
  
  // Alternative non-standard syntax for prototype. Avoid.
  foo.__proto__.print();
  
  // Object.create() --> set prototype?
  
  // Delete inherited property
  delete foo.prototype.val;
```

## Conversion

```javascript
  // Convert object to/from string
  fooString = JSON.stringify(foo);
  foo = JSON.parse(fooString);
```
  
## Scope

Always declare local variables otherwise treated as global!

```javascript
  // Function scope
  var name = 'Richard';
  function showName () {
  	var name = 'Jack';
	  console.log(name);    // Jack
  }
  console.log(name);      // Richard

  // No block scope (overwrites global variable)
  var name = 'Richard';
  if (name) {
	  name = 'Jack'; 
	  console.log(name);    // Jack
  }
  console.log(name);      // Jack
  
  // SetTimeout runs in global scope
  var bar = 123;
  var fooj = {
	  bar: 456,
	  stuff: function () {
      setTimeout (function  () {
	      console.log(this.bar);    // 123
      }, 100);
	  }
  }
  
  // Hoisting function takes precedence over variable declaration
  var foo;
  function foo() {}
  console.log(typeof foo);  // function
  
  // Hoisting variable assignments takes precedence over function
  var foo = 123;
  function foo() {}
  console.log(typeof foo);  // string
  
  // Function expressions not hoisted
  var myName = function() {} 
```
