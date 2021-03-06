---
layout: page
title: Front-End (Javascript/CSS/SASS)
---

## Start here

One of the best comunity driven resources for coding standards is [Steve Kwan's best practices][steve-kwan-best-practices]. The recommended reading is excellent and it's well worth following the links therein.

## Recommended Reading & Keynotes

This one is already mentioned in [Steve Kwan's best practices][steve-kwan-best-practices] but is worth mentioning twice since:

[Javascript - The Good Parts][javascript-the-good-parts] (Douglas Crockford's)

[Learning Javascript Design Patterns][learning-javascript-design-patterns] (Addy Osmani)

[Google v8 optimizations][google-v8-optimizations]

Memory Management [keynote][memory-management-keynote] and [video][memory-management-video]

[Javascript the better parts][js-better-parts] (Douglas Crockford)


## General Tips

- Get peer reviews of your code: Make 'em short; do them early; small reviews get much faster feedback.
- New files need to be added to the build profile.
- When native javascript APIs exist, use these instead of those provided by a framework. i.e. Native `array.forEach` vs `dojo.base.array.forEach`.
- Similarly, whenever possible, use native HTML5 widgets over framework widgets such as dojo's `dijit`. Only use other widgets when absolutely necessary and preferably instantiate those widgets programmatically.
- Use [JSDoc3][jsdoc3] to document your code.
- Write *TESTS*! Especially for your models.
- Use compass mixins instead of multiple rules with browser prefixes.
- Don't use `parseOnLoad`, parse small page sections instead.
- Don't use declarative mode to instantiate dojo widgets. In case there's no alternative, define the value as an element property instead of an option of the data-dojo-props attribute.

## Performance Gotchas

- Minimize DOM manipulation, use an in memory DOM if necessary to minimize the performance issues related to DOM layout recalculation.
- Use `lang.clone` just when it is strictly necessary, it can cause performance issues.
- Minimize the number of [delete statements][google-v8-optimizations] in order to prevent [performance issues][memory-management-keynote]:

  ``` javascript
  // Remove hidden classes...
  delete object.a;

  // Use null set instead so the hidden classes are still valid.
  object.a = null;
  ```


## Syntax

### Use AMD syntax

Use AMD syntax for all new work. When opportunity allows, refactor legacy files to use AMD syntax. Typically this is when you need to make changes to them.

#### Example

```javascript

define([
    'requirement_1',
    'requirement_2'
], function(requirement_1, requirement_2) {
    // private scope here

    return /* return public API here: typically a module, function or class */
});
```

### Loops: for(...) / while(...)

#### arrays

```javascript
// Old style for loop
for(var i = 0; i < data.length; i++) {
    data[i].label = "This is element number " + i;
}

// New style. It avoids hoisting problems and avoids scope pollution
// by keeping the "i" variable local to the body of the "loop".
data.forEach(function(item, i) {
    item.label = "This is element number " + i;
});
```

#### objects

```javascript
// Old style using actual loops
for(var prop in person){
    if(person.hasOwnProperty(prop)){
        console.log("Person's " + prop + " is " + person[prop]);
    }
}

// New style. It's shorter, avoids hoisting problems and avoids scope pollution
// by keeping the "prop" variable local to the body of the "loop".
Object.keys(person).forEach(function(prop){
    console.log("Person's " + prop + " is " + person[prop]);
});

// You can even pass the loop a `this` context
Object.keys(person).forEach(function(prop){
    console.log("Person's " + prop + " is " + person[prop]);
    console.log("Outer function's context:", this.my_parent_property);
}, this);
```

#### short-circuiting

```javascript
// Old style for loop
var found = false;
for (var i = 0; !found && i < array.length; i++) {
    found = array[i] === 5;
}

// New declarative style
var found = array.some(function(x) {
    return x === 5;
});
```

#### for vs while

In cases where looping functions from `Array.prototype` like the aforementioned
are impractical and a plain loop is needed, prefer `for` loops over `while`
loops if the number of iterations is known, e.g. when iterating over an array:

```javascript
// Short-circuited while loop
var i = 0;
var found = false;
while (!found && i < array.length) {
    found = array[i] === 5;
    i++;
}

console.log(found ? 'Found at index ' + (i - 1) : 'Not found');

// For loops are more concise and capture the control logic in a single place,
// making it harder to, say, forget to update the counter
var found = false;
for (var i = 0; !found && i < array.length; i++) {
    found = array[i] === 5;
}

// Contrary to languages with similar syntax, the counter variable has the same
// scope as the loop, so this will work:
console.log(found ? 'Found at index ' + (i - 1) : 'Not found');
```

### Object instantiation

#### Avoid shared properties

Initialize class properties in the constructor function to avoid shared properties across instances. See the example below:

``` javascript
// TestExample.js
define([
    'dojo/_base/declare'
], function(declare){
    var TestModel = declare([], {
        a: [],
        b: [],
        constructor: function(){
          this.a = [1,2,3];
        }
    });


    return {
        init: function(){
            var t = new TestModel();
            var t1 = new TestModel();

            console.log(t.a,'=== [1, 2, 3]'); // [1, 2, 3] === [1, 2, 3]
            console.log(t.b,'=== []'); // [] === []
            // the issue is that property b is resolved from the
            // __proto__ (parent) context.
            console.log(t.__proto__.b, '=== []'); // [] === []
            // but a is resolve from the instance itself
            console.log(t.a, '=== [1,2,3]'); // [1,2,3] === [1,2,3]
            // not from the parent context
            console.log(t.__proto__.a, '=== [ ]'); // [] === []

            // Saying that t and t1 should have the same data
            console.log(t.a, '===', t1.a); // [1,2,3] === [1,2,3]
            console.log(t.b, '===', t1.b); // [] === []

            // So what happen if we modify the properties?

            // If we modify property a of `t` then `t1`
            // is not modified because we are changing the instance property.
            t.a.push(4);
            console.log(t.a, '!==', t1.a); // [1,2,3,4] !== [1,2,3]

            // If we modify property b of `t` then `t1` is modified because
            // b is a __proto__ property shared between all instances...
            t.b.push(1);
            console.log(t.b, '===', t1.b); // [1] === [1]
        }
    }
});

```

### Using a ```FilteringSelect + QueryReadStore```
Over the years, we've had no small amount of performance issues relating to the python implementation of ```FilteringSelect + QueryReadStore```combo. There is now a common utility python function that encpsulates most of the validation and business logic of ```FilteringSelect```. Function ```webapp/controllers/util.py:common_filtering_select_processor``` validates arguments and encapsulates searching, listing and paging functionality respecting the protocol of ```QueryReadStore```. Two things are noteworthy, 1. *always* introduced upper bounds in the python code, as ```QueryReadStore``` cannot be trusted and 2. there is a spurious query that occurs the very first time that a page containing the ```FilteringSelect``` widget is loaded and the ```FilteringSelect``` widget is clicked.


## CSS/SASS

### General

We use [compass][compass], which in turn uses [sass][sass].

For icons, we use icomoon: you can see the [full set of available icons here](icomoon/demo.html).

#### Flexbox for Layout & Flow

Forget about the standard box model; use flexbox. Compass provides [flexbox mixins][flexbox-mixins]. For a visual overview, see [CSS Tricks' flexbox guide][flexbox-guide].

#### Cross-browser support

Use [compass' mixins][compass-mixins] for cross-browser support:

```sass
#header
  @include border-bottom-radius(50px)
```

#### Comments

Comments should on a line of their own (above the target line); not inline.

```sass
.fa-MyComponent
  font-size: 12pt
  // This country has big borders. <-- good comment
  border: 100px
  margin: 0 // but no margins. <-- bad comment
```


### Component & Widget styling: Naming conventions

This naming convention relies on _structured class names_ and _meaningful hyphens_ (i.e., not
using hyphens merely to separate words). This helps to work around the current
limits of applying CSS to the DOM (i.e., the lack of style encapsulation), and
to better communicate the relationships between classes.

Migration to this architecture will take time and should be done whenever styles of compontents are worked on.

#### Credit

The following styles are heavily based on, and an adaption of the excellent [SUIT CSS naming conventions][suitcss-naming-conventions].

### Components/Widgets

Syntax: `[<namespace>-]<ComponentName>[--modifierName|-descendentName]`

This has several benefits when reading and writing HTML and CSS/SASS:

* It helps to distinguish between the classes for the root of the component,
  descendent elements, and modifications.
* It keeps the specificity of selectors low.
* It helps to decouple presentation semantics from document semantics.

Each component should be defined in their own SASS file.

#### Namespace

Custom components should be prefixed with the `fa` namespace. We want to avoid the potential for collisions between libraries and custom components by prefixing all components with a namespace.

```sass
.fa-Button
  /* … */
.fa-Tabs
  /* … */
```

This makes it clear, when reading the HTML, which components are part of your
library.

<a name="ComponentName"></a>
#### ComponentName

The component's name must be written in pascal case. Nothing else in the
HTML/CSS uses pascal case.

```sass
.fa-MyComponent
  /* … */
```

```html
<article class="fa-MyComponent">
  …
</article>
```

<a name="ComponentName--modifierName"></a>
#### ComponentName--modifierName

A component modifier is a class that modifies the presentation of the base
component in some form (for a certain configuration of the component).
Modifier names must be written in camel case and be separated from the
component name by two hyphens. The class should be included in the HTML _in
addition_ to the base component class.

```sass
/* Core button */
.fa-Button
  /* … */
/* Cancel and Default buttons often differentiate themselves visually from normal buttons. This would be a perfect usage of --modifierName */
/* Default button style */
.fa-Button--default
  /* … */
/* Cancel button style */
.fa-Button--cancel
  /* … */

/* Core sidebar button */
.fa-SidebarButton
  /* … */
/* Sidebar primary action button style */
.fa-SidebarButton--primary
  /* … */
/* Sidebar secondary action button style */
.fa-SidebarButton--secondary
  /* … */
```

```html
<button class="fa-Button fa-Button--default" type="button">…</button>

<div class="fa-Sidebar">
  <button class="fa-Button fa-Button--sidebarPrimary" type="button">
    Save
  </button>
  <button class="fa-Button fa-Button--sidebarSecondary" type="button">
    Print
  </button>
</div>
```

<a name="ComponentName-descendentName"></a>
### ComponentName-descendentName

A component descendent is a class that is attached to a descendent node of a
component. It's responsible for applying presentation directly to the
descendent on behalf of a particular component. Descendent names must be
written in camel case.

```html
<article class="fa-Tweet">
  <header class="fa-Tweet-header">
    <img class="fa-Tweet-avatar" src="{{src}}" alt="{{alt}}">
    …
  </header>
  <div class="fa-Tweet-bodyText">
    …
  </div>
</article>
```

<a name="is-stateOfComponent"></a>
### ComponentName.is-stateOfComponent

Use `is-stateOfComponent` to reflect changes to a component's state, e.g. `is-expanded` or `is-loadingData`. The state name must be camel case. **Never style these classes directly; they should always be used as an adjoining class.**

This means that the same state names can be used in multiple contexts, but
every component must define its own styles for the state (as they are scoped to
the component).

```sass
.fa-Tweet
  /* … */
.fa-Tweet.is-expanded
  /* … */
```

```html
<article class="fa-Tweet is-expanded">
  …
</article>
```


## Tools and configuration

### Generic

- [Jshint file][jshint-file]

### Sublime Text

- Package Control [installation][package-control-install]
- Basic [Sublime Text User settings][sublime-settings]
- Sublime Text [Package Control settings][sublime-package-control] with some nice packages to have.

## ChangeLog

#### 2015.08.13

- Added link to icon list

#### 2015.05.22

- Added CSS/SASS naming convention for components
- Added SASS flexbox recomendations

#### 2015.05.19

- Add css/sass section

#### 2015.05.13

- Add more general tips
- Add performance gotchas
- Add tools & settings

#### 2015.05.08

- Initial Draft

<!-- links -->

[google-v8-optimizations]:https://developers.google.com/v8/design
[memory-management-keynote]:https://speakerdeck.com/addyosmani/javascript-memory-management-masterclass
[memory-management-video]:https://www.youtube.com/watch?v=LaxbdIyBkL0
[steve-kwan-best-practices]:https://github.com/stevekwan/best-practices/blob/master/javascript/best-practices.md
[javascript-the-good-parts]:http://javascript.crockford.com/
[learning-javascript-design-patterns]:http://addyosmani.com/resources/essentialjsdesignpatterns/book/
[jsdoc3]:https://github.com/jsdoc3/jsdoc
[compass-mixins]:http://compass-style.org/index/mixins/
[suitcss-naming-conventions]:https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md
[compass]:http://compass-style.org/
[sass]:http://sass-lang.com/
[flexbox-guide]:https://css-tricks.com/snippets/css/a-guide-to-flexbox/
[flexbox-mixins]:http://compass-style.org/reference/compass/css3/flexbox/#mixin-flexbox

[jshint-file]:{{ site.baseurl }}{{ post.url }}/files/jshintrc
[package-control-install]:https://packagecontrol.io/installation
[sublime-settings]:{{ site.baseurl }}{{ post.url }}/files/Preferences.sublime-settings
[sublime-package-control]:{{ site.baseurl }}{{ post.url }}/files/PackageControl.sublime-settings
[js-better-parts]:https://www.youtube.com/watch?v=PSGEjv3Tqo0
