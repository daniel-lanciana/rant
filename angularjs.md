![](https://cloud.githubusercontent.com/assets/1894523/5178178/5773775c-74bb-11e4-94f9-55293ba920c8.png)

## About

Open-source MVC DI (Dependency Injection) framework maintained by Google suited to single-page applications. Uses the prefix 'ng', which is supposed to sound like 'angular'. Angular leverages Node.js to run the back-end in Javascript (command-line JS engine). Directive-driven.

## Notes

* Suited for single-page ADD (API-Driven Development, REemitSTful backend) apps
* Not suited to dynamic form creation (expressions evaluated after directives causes binding issues)
* Lots of automagic in the background -- great when working, PAINFUL when not (wiring, internal state, logs lacking)
* Verbose -- both Javascript and directives.
* Always check to see if a directive already exists (e.g. ng-minlength)
* To avoid name collision, some Angular objects have a $ prefix. Do not use $ prefixes.
* For readability, parameters can have underscores which are ignored at runtime (i.e. _foo_ is the same as foo, but allows developers to easily differentiate)
* Prototypical inheritance and scopes can be a bit hairy
* IE8 support discontinued in Angular 1.3, IE8 compatibility issues with 1.2 (e.g. <ng-form> directive)
* Sharing the same name for a radio button group within an ng-repeat causes user actions (e.g. clicking radios) to be shared between all repeatable blocks! Work for inputs. Inconsistent.
* Non-intuitive error messages (e.g. "Failed to instantiate module app")
* Code-repetition (adding a dependecy = bower/npm, index, angular.module, karma.conf, karma IDE-only, projectPath config)
* Only allows direct DOM manipulation/rendering within directives
* Angular v2 will be radically different and not backwards compatable (i.e. Play! 1 vs 2)
* No support for dynamic form name attributes -- instead must wrap each element in an ng-form tag and use a generic name (e.g. 'foo')!
* Always have a dot in ng-model (to ensure model is always an object, required for Javascript prototype inheritance used for scoping)
* Don't abuse $emit and $broadcast -- can make code hard to follow and maintain
* Controller should be lean. DOM manipulation done in directives only. Logic and AJAX done in services.

## Components

* [Grunt](http://gruntjs.com/) for running tasks (builds, starting server, watching for changes and reloading)
* [Bower](http://bower.io/) for front-end dependency management (e.g. jQuery)
* [npm](https://www.npmjs.org/) (Node Package Manager) for back-end Node dependency management (e.g. xml2json)
* [Angular-Validator](https://github.com/turinggroup/angular-validator) for flexible input validation

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
* For environments (e.g. dev, prod), use grunt-ng-constant to manage an environment-specific config file. For mocking APIs, use a library like Mockjax.
* Injection order must be replicated in order in the function within a controller.
```javascript
angular.module('myModule').controller('myController', ['$scope', '$state', 'FOO',
  function ($scope, $state, FOO) {
    ...
```
* Module lifecyle comprised of config (set up providers, directives, filters...) then run (compile DOM, run application) phases

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

## Dependency Injection

* $provider service creates injectable things (i.e. services)
* Provider automatically named fooProvider (where foo is the name of the service) 
* Provider, factory and value are shortcuts (less configuration) for adding the same provider
```javascript
// All 3 do exactlyt the same thing
myModule.provider('foo', function() {
  this.$get = function() {
    return function(name) {
      alert("Hello, " + name);
    };
  };
});

// No get required
myModule.factory('foo', function() {
  return function(name) {
    alert("Hello, " + name);
  };
});

// Only one function
myModule.value('foo', function(name) {
  alert("Hello, " + name);
});
```
* Constant and value always return a static value. Constant has no provider suffix and can be injected into a module's config.
```javascript
// Value returns a simple object
myModule.value('foo', 999);

// Factory builds/returns a reused (more complex) object
myModule.factory('foo', function(aaa) {
    return 'a value' + aaa;
});

// Service singleton containing functions
function MyService() {
    this.doIt = function() {
        console.log('done');
    }
}
myModule.service('foo', MyService);

// Constants are static
myModule.constant('foo', 'hello world');

// Providers more flexible form of factory (see above for example)

// Usage
myModule.controller("MyController", function($scope, foo) {
```
* $injector actually creates instances from $provide
* $injector knows how to inject itself!
* Injector returns singleton of services
* Can inject into any controller, directive, service, filter or $get method of a provider (all these called with $injector.invoke)
* Can inject providers only (e.g. fooProvider not foo) into a module config() 
```javascript
// Get instance of service
var foo = $injector.get('foo');

// Inject services into functions
var myFunction = function(foo) {
  foo('Ford Prefect');
};
$injector.invoke(myFunction);
```
* Controllers, filters and directives use their own providers and can't be injected into other things
* Dependencies between modules
```javascript
var foo = angular.module('foo', []);
foo.value('myValue', '12345');
// Inject the foo dependency, exposing all foo's providers (in this case 'myValue')
var bar = angular.module('bar', ['foo']);
bar.controller("MyController", function($scope, myValue) { }
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

JSTL (Java Standard Tag Library) semantic tag attributes for Angular. Great for re-usable components in HTML pages such as form elements. The example below outputs a form text input, mandatory message and the message 'Hello' three times. Leverages the Angular compiler to attach behavior to any HTML element or create new elements. 

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
* Attribute expressions are PAINFUL
  * If directives have inner HTML, must remove attributes off the parent element. Issue removing expression attributes required using a prefix (e.g. 'data-id' instead of 'id')
  * Dynamically setting a directive name attribute doesn't work and causes ng-pattern validation to break
  * ng-show/hide based on form elements doesn't work with expressions
  * No simple solution (wrap in ng-form directive, complex programatic directive)
  * Inconsistency between attributes (e.g. ng-if takes 'foo.bar' but ng-require takes '{{foo.bar}}') -- should gracefully fallback
  * For access to the actual variable (not the expression), must call $compile within a directive. This can cause an infinite loop, and requires setting terminal:true and priority:10000 to ensure it gets compiled first.
  * Default ng-pattern validation does not display error messages if the name attribute contains an expression! WTF!!
* The input states $dirty and $pristine are a very nice touch
* Compile v Link
  * Compile is for changing the DOM structure before rendering. After compile, link is called.
  * Link is for existing element listeners and copying content from the scope (e.g. to display)

```javascript
app.directive("simple", function(){
    return {
        link: function(scope, element, attributes){
        },
        controller: function($scope, $element){
        }
    };
});
```

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
<div ng-controller="myController">
 <a ui-sref="foo" ng-class="{ active: isActive('/foo') }">My link</a></li>
</div>
```

### controller.js

```javascript
angular.module('myApp')
    .controller('myController', ['$scope', '$location', function($scope, $location) {
      $scope.isActive = function (viewLocation) {
        return viewLocation === $location.path();
      };
    }]);
```

## Unit Testing

* Uses Jasmine (and Jasmine matchers = more powerful comparitors) executed by the Karma runner
* Karma can be run by Grunt build tool (e.g. grunt test) or standalone in WebStorm
   * Must define all source and tests files to run -- in order (fragile)
   * No ability to run (or re-run) a single test?!?
   * To run tests, right click karma.conf.js and select 'run'
   * List of files as regular expression flakey (occasionally only picks up one file, not all!?)
   * For testing templateUrl in directives, need to use the ng-html2js-preprocessor plugin, which can be a bit magical to set up. When working it loads all templates and exposes in an angular module for tests to use.
* Test file naming convention is foo.spec.js

```javascript
// Dependency Injecting http, promises, scope and another named service ('anotherService') into the MyService service 
var service, dependency;

beforeEach(inject(function (MyService, $httpBackend, $q, $rootScope, anotherService) {
  service = MyService;
  dependency = anotherService;
  
  httpBackend = $httpBackend;
  q = $q;
  scope = $rootScope;
}));

// Dependecy Injecting a controller (essentially the same as a service)
var controller, dependency;

inject(function ($controller, $rootScope, anotherService) {
  scope = $rootScope.$new();
  dependency = anotherService;

  controller = $controller('MyController', {
    $scope: scope,
    argument1: 'foo',
    argument2: dependency.getBar()
  }  
});

// Mocking a .constant('myConfig')
var config;

beforeEach(function() {
  module('app', function ($provide) {
    $provide.constant('myConfig', config);
  });
});

// Stubbing a .service('myService') function isNumber()
module(function($provide) {
  $provide.service('myService', function() {
    this.isNumber = jasmine.createSpy('isNumber').andCallFake(function(num) {
      return true;
    });
  });
});  

// Testing a REST URL with a returned promise
describe('creating a new assessment', function () {
  it('should call the endpoint', function () {
    myService.doSomething();
    httpBackend.expect('GET', 'http://foo.com').respond(200, '');
    httpBackend.flush();
    
    promise.then(function (data) {
      expect(data).toBe(testResponse);
    });

    // Flush the promise
    scope.$digest();
  });
});  
```

### Controller

```javascript
'use strict';

// DSL notation from Jasmine
describe('myController', function () {
  // Class variable for use in tests
  var controller, state, scope, service;

  beforeEach(function () {
    angular.mock.module('myAngularModule');
    angular.mock.module('myServiceModule');

    // Provider service name e.g. angular.module.provider('myServiceName')
    inject(function ($controller, $state, $rootScope, myServiceName) {
      state = $state;
      // No controller scope available, use root scope instead
      scope = $rootScope.$new();
      service = myServiceName;
      // Initialise and instantiate the controller for testing
      controller = $controller('myController', {
        $scope: scope,
        $state: state,
        $service: service
      });
    });
  });

  it('should be able to access the controller', function () {
    expect(controller).not.toEqual(null);
  });
  
  it('should return hello world', function () {
    expect(controller.printme()).toEqual('hello world');
  });

  it('should call the method foo()', function () {
    spyOn(state, 'go');
    controller.foo();
    expect($state.go).toHaveBeenCalled();
  });
});
```
### Directive

```javascript
'use strict';

// DSL notation from Jasmine
describe('myDirective', function () {
  var compile, scope;

  // Inject the directive
  beforeEach(angular.mock.module('myDirective'));
  // Import the input HTML templates (ng-html2js-preprocessor)
  beforeEach(module('myTemplates'));
  // Inject the compile and scope dependencies
  beforeEach(inject(['$compile', '$rootScope', function ($c, $r) {
    compile = $c;
    scope = $r.$new();
  }]));

  it('should output a DIV with "Hello World" inside', function () {
    var elem = angular.element('<my-hello-world-directive></my-hello-world-directive>');
    $compile(elem)(scope);
    scope.$digest();
    expect(elem[0].outerHTML).toEqual('<div>Hello World</div>');
  });
});  
```

### karma.conf.js

```javascript
module.exports = function(config) {
  config.set({
    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: '../../',
    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: [
      'jasmine', 
      'jasmine-matchers'
    ],
    // list of files / patterns to load in the browser
    files: [
      'bower_components/angular/angular.js',
      'bower_components/angular-mocks/angular-mocks.js',
      'bower_components/angular-ui-router/release/angular-ui-router.js',
      'node_modules/karma-jasmine-matchers/node_modules/jasmine-expect/dist/jasmine-matchers.js',
      'src/modules/common/_app.js',               // Need to load the main module first
      'src/modules/**/*.js',                      // Source files
      'src/modules/templates/*.html',             // HTML templates (ng-html2js-preprocessor)
      'test/modules/**/*.spec.js'                 // Tests
    ],
    // preprocess matching files before serving them to the browser (ng-html2js-preprocessor)
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      'src/modules/templates/*.html': ['ng-html2js']
    },
    // test results reporter to use
    // available reporters: https://npmjs.org/browse/keyword/karma-reporter
    reporters: ['progress'],
    // web server port
    port: 9876,
    // enable / disable colors in the output (reporters and logs)
    colors: true,
    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,
    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: false,
    // start these browsers
    // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
    browsers: ['PhantomJS'],
    plugins: [
      'karma-phantomjs-launcher',
      'karma-jasmine-jquery',
      'karma-jasmine',
      'karma-jasmine-matchers',
      'karma-ng-html2js-preprocessor' 
    ],
    ngHtml2JsPreprocessor: {
      // If your build process changes the path to your templates,
      // use stripPrefix and prependPrefix to adjust it.
      stripPrefix: 'src/modules/templates/',
      prependPrefix: 'views/templates/',
      // the name of the Angular module to create and expose
      moduleName: 'myTemplates'
    },
    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false
  });
};
```

## Libraries

* [UI Bootstrap](http://angular-ui.github.io/bootstrap/)
* [AngularUI Router](https://github.com/angular-ui/ui-router) (enhanced angular-route, nested, URL updating, decorator dynamic routes, states)
* [Stylus](http://learnboost.github.io/stylus/) (SASS-like CSS preprocessor)
* [es5-shim](https://github.com/es-shims/es5-shim) (monkey patch JS to EcmaScript 5 in older browsers)
* [Respond.js](https://github.com/scottjehl/Respond) (polyfill CSS3 min/max in IE8 for responsive design)
* [Karma](http://karma-runner.github.io/0.12/index.html) (test runner, includes Jasmine test framework and PhantomJS browser)
* [Grunt Ng Constant](https://github.com/werk85/grunt-ng-constant) (environment properties config file)

## Tools

* [Plunker](http://plnkr.co/): Edit and run AngularJS code online
* [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk): Chrome extension
* [WebStorm](https://www.jetbrains.com/webstorm/)
* [angular-slugify](https://github.com/paulsmith/angular-slugify): Friendly URL conversion of strings (slug is a term used by journalists referring to headline. Broken with Grunt minify!)

## TODO

* CSS/Sprites
* Unit tests (directives, services, controllers)
* Model (multi-form)
* Services (API)
* https://github.com/sydcanem/angularjs-testing-cheat-sheet (testing cheat sheet)
* http://jasmine.github.io/2.0/introduction.html (Jasmine docs)
* http://quickleft.com/blog/angularjs-unit-testing-for-real-though (unit testin article)

## References

* http://spion.github.io/posts/why-i-am-switching-to-promises.html (promises)
* http://stackoverflow.com/questions/14049480/what-are-the-nuances-of-scope-prototypal-prototypical-inheritance-in-angularjs (prototypical inheritance and scopes)
* https://code.angularjs.org/1.2.23/docs/api/ng/directive/a (API docs for directives)
* http://mindthecode.com/how-to-use-environment-variables-in-your-angular-application/ (multiple environments)
* https://github.com/angular/angular.js/wiki/Understanding-Dependency-Injection (DI explained)
* https://github.com/angular/angular.js/issues/1404 (dynamic validation workarounds)
