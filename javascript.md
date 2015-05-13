---
layout: page
title: Javascript
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
- Define your CSS at component level instead of page (and put each into its own component file).
- Use compass mixins instead of multiple rules with browser prefixes.
- Don't use `parseOnLoad`, parse small page sections instead.


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

// If you need access to the outer function's `this` context, you need
// to use lang.hitch.
// Note: ideally we'd defer to a function's `bind()` method, but
// phantomjs 1.9.x doesn't support it.
Object.keys(person).forEach(lang.hitch(this, function(prop){
    console.log("Person's " + prop + " is " + person[prop]);
    console.log("Outer function's context:", this.my_parent_property);
});
```

## Tools and configuration

### Generic

- [Jshint file][jshint-file]

### Sublime Text

- Package Control [installation][package-control-install]
- Basic [Sublime Text User settings][sublime-settings]
- Sublime Text [Package Control settings][sublime-package-control] with some nice packages to have.

### ChangeLog

#### 2015.05.08

- Initial Draft

#### 2015.05.13

- Add more general tips
- Add performance gotchas
- Add tools & settings

<!-- links -->

[google-v8-optimizations]:https://developers.google.com/v8/design
[memory-management-keynote]:https://speakerdeck.com/addyosmani/javascript-memory-management-masterclass
[memory-management-video]:https://www.youtube.com/watch?v=LaxbdIyBkL0
[steve-kwan-best-practices]:https://github.com/stevekwan/best-practices/blob/master/javascript/best-practices.md
[javascript-the-good-parts]:http://javascript.crockford.com/
[learning-javascript-design-patterns]:http://addyosmani.com/resources/essentialjsdesignpatterns/book/
[jsdoc3]:https://github.com/jsdoc3/jsdoc

[jshint-file]:{{ site.baseurl }}{{ post.url }}/files/jshintrc
[package-control-install]:https://packagecontrol.io/installation
[sublime-settings]:{{ site.baseurl }}{{ post.url }}/files/Preferences.sublime-settings
[sublime-package-control]:{{ site.baseurl }}{{ post.url }}/files/PackageControl.sublime-settings
[js-better-parts]:https://www.youtube.com/watch?v=PSGEjv3Tqo0
