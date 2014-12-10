![](https://cloud.githubusercontent.com/assets/1894523/5178178/5773775c-74bb-11e4-94f9-55293ba920c8.png)

## About

Open-source MVC DI (Dependency Injection) framework maintained by Google suited to single-page applications. Uses the prefix 'ng', which is supposed to sound like 'angular'. Angular leverages Node.js to run the back-end in Javascript (command-line JS engine). Directive-driven.

## Notes

* Verbose -- both Javascript and directives.
* Always check to see if a directive already exists (e.g. ng-minlength)
* To avoid name collision, some Angular objects have a $ prefix. Do not use $ prefixes.
* Prototypical inheritance and scopes can be a bit hairy

## Components

* [Grunt](http://gruntjs.com/) for running tasks (builds, starting server, watching for changes and reloading)
* [Bower](http://bower.io/) for front-end dependency management (e.g. jQuery)
* [npm](https://www.npmjs.org/) (Node Package Manager) for back-end Node dependency management (e.g. xml2json)

## Concepts

* For ansyc calls, use promises not callbacks
  * Mature, tested pattern for lifecycle of an asynchronous task
  * Bluebird better than (Kris Kowall's) Q. Almost as fast as callbacks...and better features. 5x slower in debug mode.
  * Advantages: catches errors, supported by jQuery, run once (once resolved), nodify/denodify existing functions
```javascript
// A promise
function readFileOrDefault(file, defaultContent) {
    return fs.readFile(file)
     .then(function(fileContent) {
        return fileContent;
    }, function(err) {
        return defaultContent;
    });
}
```
* Generators: new to Javascript 1.7, function that acts like an iterator, executes one statement at a time, .next(), yeild passed in objects. Libraries: [suspend](https://github.com/jmar777/suspend), [genny](https://github.com/spion/genny)
```javascript
// A generator
function foo*(x) {
    while(true) {
        x = x * 2;
        yield x;
    }
}

var g = foo(2);
g.next(); // -> 4
g.next(); // -> 8
```
  * Thunk: a partially-evaluated function that accepts a single callback argument. Library: [co](https://github.com/tj/co)
```javascript
// A thunk
function foo(callback) {
   // read file
}
```
* Modules relate to functionality and contain various classes (e.g. controller, service)

## Scope

* Child elements prototypically inherit from parent (as opposed to class-ical)
  * Prototype means if an object or method does not exist in a child element, the prototype chain is consulted (i.e. traverses up all parent scopes until found or hits root)
  * If we read childScope.propertyX, and childScope has propertyX, then the prototype chain is not consulted. If we set childScope.propertyX, the prototype chain is not consulted.
  * Modifying objects (by value) creates on the child, modifying object properties (by reference) modifies the parent object (if exists). Example assuming child has no object named 'array': child.array[1] = 1 (modifies parent object), child.array = [1, 2] (creates new object on child)
  * There are four types of scopes:
    * normal prototypal scope inheritance -- ng-include, ng-switch, ng-controller, directive with scope: true
    * normal prototypal scope inheritance with a copy/assignment -- ng-repeat. Each iteration of ng-repeat creates a new child scope, and that new child scope always gets a new property.
    * isolate scope -- directive with scope: {...}. This one is not prototypal, but '=', '@', and '&' provide a mechanism to access parent scope properties, via attributes.
    * transcluded scope -- directive with transclude: true. This one is also normal prototypal scope inheritance, but it is also a sibling of any isolate scope.
    * For all scopes (prototypal or not), Angular always tracks a parent-child relationship (i.e., a hierarchy), via properties $parent and $$childHead and $$childTail.
* Default directives don't create a new scope, which can lead to DOM clobbering (overwriting an existing element, leaking state). 
  * Solutions for primatives (to modify parent not create on child) are $parent.foo or setter in parent. Issues with 2-way data binding for forms not updating the controller scope!
  * Services should be used to share data between controllers -- not scope inheritance (tightly coupled)
  * Directives should define isolated scopes to avoid accidentally modifying the parent scope. Operator '=' for 2-way binding, '@' for 1-way binding (i.e. read-only)
  * Attributes must be specified for each parent property for each binding
```
<my-directive interpolated="{{parentProp1}}" twowayBinding="parentProp2"> 
scope: { interpolatedProp: '@interpolated', twowayBindingProp: '=twowayBinding' }
```

## Routing

### index.html

* The ng-app directive defines the application by name
* Links have a hash before them, which are intercepted (hence the single-page app pattern)
* Scope (i.e. model) variables are within two curly braces
* The ng-view directive displays rendered templates from links. Without it routing is not fired (and the console log empty).
* The {{vendorScripts}} injects Bower dependencies (e.g. angular.js) to the page. Define additional scripts after.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World</title>
    <link rel="stylesheet" href="css/main.css" />
  </head>
  <body ng-app="app">
    <div class="contains">
      <nav>
        <ul>
            <li><a href="#/foo">foo</a></li>
        </ul>
        <p>{{message}}</p>
        <div ng-view></div>
      </nav>
      <!--(if target unoptimised)>
        {{vendorScripts}}
        <script src="js/app.js"></script>
        <script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.2.1/angular-route.min.js"></script>
      <!(endif)-->   
    </div>
  </body>
</html>
```

### js/app.js

* Referenced from index.html
* The template 'foo.tpl.htm' is basic HTML and can contain whatever

```javascript
(function (angular) {
  'use strict';

  // Name the application 'app', matches the ng-app directive in index.html
  var ang = angular
    .module('app', [
      'ngRoute'
    ]);

  // Routing config, define request mapping to a controller and view
  ang.config(['$routeProvider', function ($routeProvider) {
    $routeProvider
      .when('/foo', {
        controller: 'fooController',
        templateUrl: 'views/foo.tpl.htm'
      });
  }]);

  // Controller optional (I think), add variables to the scope (i.e. model)
  var controllers = {};
  controllers.fooController = function($scope) {
    $scope.message = 'Hello World';
  };
  ang.controller(controllers);
})(window.angular);
```

## Directives

JSTL (Java Standard Tag Library) semantic tag attributes for Angular. Great for re-usable components in HTML pages such as form elements. The example below outputs a form text input, mandatory message and the message 'Hello' three times.

* Types:
  * Decorator:
    * Adding behaviour (e.g. listerners, new properties, add to DOM) or intercept behaviour (override events, global listeneers) of existing elements
    * Can co-exist
    * No need to change HTML
    * Usually elements, but possible to decorate attribute-directives
    * Add to the 'link:' function (i.e. when element linked to DOM and receives $scope)
  * Component:
    * Has state (usually isolate scope: {...})
    * Change HMTL to add attributes
    * Be careful of adding multiple directives to an element (may break each other)
    * Internal state -- optionally linked to parent properties (e.g. scope: {...})
  * Template:
    * Basic templating (e.g. templateUrl)
    * Usually no state required
    * Doesn't support dynamic directives (e.g. can't add ng-hide to an element on-the-fly)
  * Collaborative:
    * Relationship between directives (e.g. inputs and forms)
    * Collaboration between directives through controller methods (i.e. scope communication before DOM operations)

### index.html

* Add directives to existing elements such as <div>
* Use either 'data-' or 'x-' prefixes to remain HTML5 compliant (reserved for custom tags)
* Can't be a self-closing tag (e.g. &lt;div ... /&gt;)

```html
<div data-text data-label="First name" data-mandatory="true">
   Hello
</div>
```

### template.html

* Variables (e.g. {{id}}) defined in the link method of the directive Javascript
* Demostrates use of predefined Angular directives such as ng-if and ng-repeat, which can be placed on any element
* No simple ng-repeat iterator (e.g. for 1 to 5)?! Use getTimes() workaround.
* The ng-transclude directive yields and outputs the contents of the parent directive -- in this case 'Hello'

```html
<div class="question">
    <label>{{label}}</label>
    <input type="text" id="{{id}}">
    <div ng-if="mandatory === 'true'">
        <span class="required-field">Required</span>
    </div>
    <div ng-repeat="i in [1,2,3]">
        <div ng-transclude></div>
    </div>
</div>
```

### directives.js

* Separate module name 'myDirectives' for namespacing
* The directive 'data-text' maps to just 'text' -- the 'data-' prefix is ignored
* Uses the dependency Slugifier to convert text to friendly URLs. 'slugifier' is the dependency name, while Slug is the exposed object in the directive
* By default directives are singleton and not very reusable (e.g. all form inputs end up being whatever the last directive is). To overcome, include the scope as either true or {}.
* In the link function, define scope variables available within the template

```javascript
(function (angular) {
  'use strict';

  angular.module('myDirectives', ['slugifier'])
    .directive('text', ['Slug', function(Slug) {
      return {
        scope: {},
        templateUrl: 'template.html',
        link: function(scope, element, attrs) {
          // Create a friendly ID from the label
          scope.id = Slug.slugify(attrs.label);
          scope.label = attrs.label;
          scope.mandatory = attrs.mandatory;
        }
      };
    }])
  ;  
})(window.angular);
```

## Navigation

* Apply the class 'active' to the current link using ng-class, which calls isActive() in the controller

### index.html

```html
<div ng-controller="NavController">
 <a ui-sref="foo" ng-class="{ active: isActive('/foo') }">My link</a></li>
</div>
```

### controller.js

```javascript
angular.module('dcazApp')
    .controller('NavController', ['$scope', '$location', function($scope, $location) {
      $scope.isActive = function (viewLocation) {
        return viewLocation === $location.path();
      };
    }]);
```

## Libraries

* [UI Bootstrap](http://angular-ui.github.io/bootstrap/)
* [AngularUI Router](https://github.com/angular-ui/ui-router) (enhanced angular-route, nested, URL updating, decorator dynamic routes, states)
* [Stylus](http://learnboost.github.io/stylus/) (SASS-like CSS preprocessor)
* [es5-shim](https://github.com/es-shims/es5-shim) (monkey patch JS to EcmaScript 5 in older browsers)
* [Respond.js](https://github.com/scottjehl/Respond) (polyfill CSS3 min/max in IE8 for responsive design)
* [Karma](http://karma-runner.github.io/0.12/index.html) (test runner, includes Jasmine test framework and PhantomJS browser)

## Tools

* [Plunker](http://plnkr.co/): Edit and run AngularJS code online
* [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk): Chrome extension
* [WebStorm](https://www.jetbrains.com/webstorm/)
* [angular-slugify](https://github.com/paulsmith/angular-slugify): Friendly URL conversion of strings (slug is a term used by journalists referring to headline)

## TODO

* CSS/Sprites
* Unit tests (directives, services, controllers)
* Model (multi-form)
* Services (API)

## References

* http://spion.github.io/posts/why-i-am-switching-to-promises.html (promises)
* http://stackoverflow.com/questions/14049480/what-are-the-nuances-of-scope-prototypal-prototypical-inheritance-in-angularjs (prototypical inheritance and scopes)
* https://code.angularjs.org/1.2.23/docs/api/ng/directive/a (API docs for directives)
