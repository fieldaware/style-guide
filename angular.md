---
layout: page
title: AngularJS
---

## Recommended Reading

* [Angular Best-Practices from Angular team](https://github.com/angular/angular.js/wiki/Best-Practices)
* [Angular Style Guide](https://github.com/johnpapa/angular-styleguide)
* [The Top 10 Mistakes AngularJS Developers Make](https://www.airpair.com/angularjs/posts/top-10-mistakes-angularjs-developers-make)

## General Tips

- use `controllerAs` view syntax
- use getter syn
- Defer Controller Logic to Services
- Many Small, Self Contained Modules
- Folders-by-Feature Structure
- Use minification-safe approach of declaring dependencies
- Provide a Unique Directive Prefix
- Manipulate DOM in a Directive

## Common

Avoid using variables when using a module. Use chaining with getter syntax
*Why?*: This produces more readable code and avoids variable collisions or leaks.

```javascript
/* recommended */
angular.module('myModule').controller(...);
```

Declare modules without a variable using the setter syntax. Only set once and get for all other instances.

```javascript
/* recommended */

// to set a module
angular.module('app', []);

// to get a module
angular.module('app');
```

Use named functions instead of passing an anonymous function in as a callback.
*Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

```javascript
/* recommended */

// dashboard.js
angular
    .module('app')
    .controller('DashboardController', DashboardController);

function DashboardController() { }
```

## Controllers
### Syntax

Use the `controllerAs` syntax over the classic controller with $scope syntax.
*Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the controllerAs syntax is closer to that of a JavaScript constructor than the classic $scope syntax.

*Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. customer.name instead of name), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

*Why?*: Helps avoid using $parent calls in Views with nested controllers.

```html
<!-- recommended -->
<div ng-controller="CustomerController as customer">
    {{ customer.name }}
</div>
```

### Bindable Members Up Top

Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.

*Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

*Why?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read.

```javascript
/* recommended */
function SessionsController() {
    var vm = this;

    vm.gotoSession = gotoSession;
    vm.refresh = refresh;
    vm.search = search;
    vm.sessions = [];
    vm.title = 'Sessions';

    ////////////

    function gotoSession() {
      /* */
    }

    function refresh() {
      /* */
    }

    function search() {
      /* */
    }
}
```

### Defer Controller Logic to Services

Defer logic in a controller by delegating to services and factories.

*Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

*Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

*Why?*: Removes dependencies and hides implementation details from the controller.

*Why?*: Keeps the controller slim, trim, and focused.

### Keep Controllers Focused

Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view.

*Why?*: Reusing controllers with several views is brittle and good end-to-end (e2e) test coverage is required to ensure stability across large applications.

### Assigning Controllers
  
When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes.

### Data Services

Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

*Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

*Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

*Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as $http. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

## Services & Factories

Services are instantiated with the `new` keyword, use this for public methods and variables. Since these are so similar to factories, use a factory instead for consistency.

Services & Factories should have a single responsibility, that is encapsulated by their context. Once a factory|service begins to exceed that singular purpose, a new factory|service should be created.

#### Accessible Members Up Top

Expose the callable members of the service (its interface) at the top, using a technique derived from the Revealing Module Pattern.

*Why?*: Placing the callable members at the top makes it easy to read and helps you instantly identify which members of the service can be called and must be unit tested (and/or mocked).

*Why?*: This is especially helpful when the file gets longer as it helps avoid the need to scroll to see what is exposed.

*Why?*: Setting functions as you go can be easy, but when those functions are more than 1 line of code they can reduce the readability and cause more scrolling. Defining the callable interface via the returned service moves the implementation details down, keeps the callable interface up top, and makes it easier to read.

```javascript
/* recommended */
function dataService() {
    var someValue = '';
    var service = {
        save: save,
        someValue: someValue,
        validate: validate
    };
    return service;

    ////////////

    function save() {
        /* */
    };

    function validate() {
        /* */
    };
}
```

#### Function Declarations to Hide Implementation Details

Use function declarations to hide implementation details. Keep your accessible members of the factory up top. Point those to function declarations that appears later in the file. For more details see this post.

*Why?*: Placing accessible members at the top makes it easy to read and helps you instantly identify which functions of the factory you can access externally.

*Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

*Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

*Why?*: You never have to worry with function declarations that moving var a before var b will break your code because a depends on b.

*Why?*: Order is critical with function expressions

```javascript
/**
* avoid
* Using function expressions
*/
function dataservice($http, $location, $q, exception, logger) {
  var isPrimed = false;
  var primePromise;

  var getAvengers = function() {
      // implementation details go here
  };

  var getAvengerCount = function() {
      // implementation details go here
  };

  var getAvengersCast = function() {
     // implementation details go here
  };

  var prime = function() {
     // implementation details go here
  };

  var ready = function(nextPromises) {
      // implementation details go here
  };

  var service = {
      getAvengersCast: getAvengersCast,
      getAvengerCount: getAvengerCount,
      getAvengers: getAvengers,
      ready: ready
  };

  return service;
}
```

```javascript
/**
* recommended
* Using function declarations
* and accessible members up top.
*/
function dataservice($http, $location, $q, exception, logger) {
    var isPrimed = false;
    var primePromise;

    var service = {
        getAvengersCast: getAvengersCast,
        getAvengerCount: getAvengerCount,
        getAvengers: getAvengers,
        ready: ready
    };

    return service;

    ////////////

    function getAvengers() {
        // implementation details go here
    }

    function getAvengerCount() {
        // implementation details go here
    }

    function getAvengersCast() {
        // implementation details go here
    }

    function prime() {
        // implementation details go here
    }

    function ready(nextPromises) {
        // implementation details go here
    }
}
```

## Changelog

##### 2016.02.09

Initial Draft





