![](https://cloud.githubusercontent.com/assets/1894523/5178178/5773775c-74bb-11e4-94f9-55293ba920c8.png)

## About

Open-source MVC DI (Dependency Injection) framework maintained by Google suited to single-page applications. Uses the prefix 'ng', which is supposed to sound like 'angular'. Angular leverages Node.js to run the back-end in Javascript (command-line JS engine). Directive-driven.

## Notes

* Verbose -- both Javascript and directives.
* To avoid name collision, some Angular objects have a $ prefix. Do not use $ prefixes.
* Scopes can get very confusing!

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
* Directives are HTML markup attached via semantic tags (e.g. <input validate-mandatory... />), can extend each other, controller option pre-linking (i.e. scope, not DOM operations). Basically taglibs.
```javascript
// A directive
app.directive('ngSparkline', function() {
  return {
    restrict: 'A',
    require: '^ngCity',
    scope: {
      ngCity: '@'
    },
    template: '<div class="sparkline"><h4>Weather for {{ngCity}}</h4></div>',
    controller: ['$scope', '$http', function($scope, $http) {
      $scope.getTemp = function(city) {}
    }],
    link: function(scope, iElement, iAttrs, ctrl) {
      scope.getTemp(iAttrs.ngCity);
    }
  }
});
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

JSTL (Java Standard Tag Library) for Angular. Great for re-usable components in HTML pages such as form elements. The example below outputs a form text input, mandatory message and the message 'Hello' three times.

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
* [angular-slugify](https://github.com/paulsmith/angular-slugify): Friendly URL conversion of strings

## References

* http://spion.github.io/posts/why-i-am-switching-to-promises.html (promises)
