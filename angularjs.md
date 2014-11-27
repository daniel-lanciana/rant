![](https://cloud.githubusercontent.com/assets/1894523/5178178/5773775c-74bb-11e4-94f9-55293ba920c8.png)

## About

Open-source MVC DI (Dependency Injection) framework maintained by Google suited to single-page applications. Uses the prefix 'ng', which is supposed to sound like 'angular'. Angular leverages Node.js to run the back-end in Javascript (command-line JS engine). Directive-driven.

## Notes

* Verbose

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

## Angular Routing

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

## Libraries

* [UI Bootstrap](http://angular-ui.github.io/bootstrap/)
* [Stylus](http://learnboost.github.io/stylus/) (SASS-like CSS preprocessor)
* [es5-shim](https://github.com/es-shims/es5-shim) (monkey patch JS to EcmaScript 5 in older browsers)
* [Respond.js](https://github.com/scottjehl/Respond) (polyfill CSS3 min/max in IE8 for responsive design)
* [Karma](http://karma-runner.github.io/0.12/index.html) (test runner, includes Jasmine test framework and PhantomJS browser)

## Tools

* [Plunker](http://plnkr.co/): Edit and run AngularJS code online
* [AngularJS Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk): Chrome extension
* [WebStorm](https://www.jetbrains.com/webstorm/)

## References

* http://spion.github.io/posts/why-i-am-switching-to-promises.html (promises)
