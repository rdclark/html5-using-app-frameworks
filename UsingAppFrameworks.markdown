% Using Web Application Frameworks
% Richard Clark

# Single-Page Applications

## A trivial SPA

[Demo here](./examples/1.1-trivial-spa.html)

## A trivial SPA (HTML)

```html
<!DOCTYPE html>
<meta charset=utf-8>
<title>A Trivial Single-Page App</title>
<form id=request>
Hi there. What is your name? <input type=text autofocus>
</form>
<div id=response style="display:none" />
<template style="display:none">Welcome, {name}!</template>
<script>
// See next slide
</script>
```

## A Trivial SPA (script)

```javascript
(function() {
  document.forms[0].onsubmit = function (event) {
	var text = event.target.elements[0].value.trim();
	if (text) {
	  var template = document.querySelector('template').innerHTML;
	  var message = template.replace(/{name}/, text);
	  var responsePage = document.querySelector('#response');
	  responsePage.innerHTML = message;
	  responsePage.style.display = 'block';
	  event.target.style.display = 'none';
	}
	event.preventDefault();
  }  
})()
```

# Key Concepts

## Key Concepts

1. Multiple screens in one page
2. Content templates
3. Routing and History

## Multiple Screens in a Page

```html
<!-- Screen 1 -->
<form id=request>
Hi there. What is your name? <input type=text>
</form>

<!-- Screen 2 -->
<div id=response style="display:none" />
```

## Multiple Screens in a Page

- Block-level elements
+ Control visibility via CSS
    - (Often by adding or removing a class)
    - `screen.classList.add('current')`
    - `screen.classList.remove('current')`

# Templating

## Content Templates

> + Two kinds:
    1. Named placeholders:

    `Hello, my name is {name}` (dust.js)
    `Hello, my name is {{name}}` (mustache.js, handlebars.js)

   2. JSP/ERB style:

   `Hello, my name is <%- name %>` -- [underscore.js](http://documentcloud.github.io/underscore/#template)

## How templates work

1. Define template in a hidden HTML element (e.g. &lt;script> or &lt;template>)
2. Load with template library &rarr; compiled function
3. Call function with a JS object

## Templated SPA (HTML)

```html
<!DOCTYPE html>
<meta charset=utf-8>
<title>A Trivial Single-Page App</title>
<style>
  .current  { display: block }
  form, div { display: none }
</style>

<form id=request class="current">
Hi there. What is your name? <input type=text autofocus>
</form>
<div id=response />
<script src="lib/underscore.js"></script>
<script>/* Next slide */</script>

```

## Templated SPA (script)

```javascript
window.onload = function() {
  var compiled = _.template("Welcome, <%- name %>!");
  var responsePage = document.querySelector('#response');

  document.forms[0].onsubmit = function (event) {
    var text = event.target.elements[0].value.trim();
    if (text) {
      responsePage.innerHTML = compiled({name:(text)});
      responsePage.classList.add('current');
      event.target.classList.remove('current');
    }
    event.preventDefault();
  }
}
```

## Practice

1. Create a simple "hello world" app using an underscore template.
2. Merge some data into the template and add to the page.
3. (Extra) Create an array and render all the elements.
4. (Extra) Render nested objects.

# Routing and History

## The problem with URLs in SPAs

+ Unique URLs typically map to unique content
    - Different pages
    - Fragments within a page: `page#named-link`
    - RESTful APIs: `/shop/shoes/mens/dress/` ...
- People want to bookmark
- Search engines want to link

## URLs and SPAs

+ Make URLs map to states in the app (_Routing_)
    - `page?state=1` vs. `page#state1`
- When state changes, update the URL
- Track URL changes to set state

## # vs #!

- Content after `#` is the _fragment identifier_
- [Google proposed](http://googlewebmastercentral.blogspot.com/2009/10/proposal-for-making-ajax-crawlable.html) `#!` to designate bookmarkable URLs

## Updating the URL

- `window.location` or `window.location.hash` (preferred)
- `window.pushState(data, description, url)` (HTML5)
- `window.replaceState(data, description, url)` (HTML5)


## Capturing URL changes

+ hashchange
    - `window.onhashchange = function() { // use window.location.hash...`
    - Works on old browsers, even IE
    - Useful library: [jQuery BBQ](http://benalman.com/projects/jquery-bbq-plugin/)

## The History API

- Added in HTML5
- Adds state objects & descriptions to browser history

## The History API

```javascript
interface History {
  readonly attribute long length;
  readonly attribute any state;
  void go(optional long delta);
  void back();
  void forward();
  void pushState(any data, DOMString title, optional DOMString? url = null);
  void replaceState(any data, DOMString title, optional DOMString? url = null);
};
```

## Tracking history changes

+ popstate
    - Event fired for history forward/back (if using pushState/replacreState)
    - `event.state` = passed-in state object

## History API gotchas

- Some browsers fire popstate on page load, others don't
- Most browsers ignore title string
- Can only set url for http/https pages

## History API example

- [Star Polygons](./examples/1.4.canvas-star-polygon.html)

## History API example

```javascript
function getValueObject() {
  var sides = parseInt(sidesCtl.value);
  var skip = parseInt(skipCtl.value);
  return {sides:(sides), skip:(skip)};
}

document.onchange = function(evt) {
  var values = getValueObject(); // ex: {sides:3, skip:1}
  window.history.pushState(values);
  draw(values);
}

window.onpopstate = function (event) {
  draw(event.state);
}

// First time
window.onload = function() {
  var values = getValueObject();
  window.history.replaceState(values);
  draw(values);
}
```

# SPAs via backbone.js

## backbone.js

+ Basic framework for building SPAs
    - View rendering
    - Routing
    - Events
    - History management
    - Networking (via REST & JSON)
    - Model-View-Controller structure
- Nice [collection of plugins and tools](https://github.com/jashkenas/backbone/wiki/Extensions%2C-Plugins%2C-Resources)
- Good entry point for more complex frameworks (e.g. AngularJS)

## Our first example, in Backbone (HTML)

```html
<!DOCTYPE html>
<meta charset=utf-8>
<title>A Trivial Single-Page App</title>

<main id="helloapp">
  <form id=request>
  Hi there. What is your name? <input id="name" type=text autofocus>
  </form>
  <div id=response />
</main>

<script src="lib/underscore.js"></script>
<script src="lib/zepto.js"></script>
<script src="lib/backbone.js"></script>
<script>/* Next slide */</script>
```
## Our first example, in Backbone (JS)

```javascript
$(function() {

  var AppView = Backbone.View.extend({

    el: $('#helloapp'),

    events: {
      "submit": "sayHello"
    },

    initialize: function() {
      this.form = $('#request');
      this.input = $('#name');
    },

    sayHello: function(event) {
      $('#response').html("Hello, " + this.input.val());
      this.form.hide();
      event.preventDefault();
    },
  });

  // Starts the app
  new AppView;

})
```

## Thank you!

[richard.clark@kaazing.com](mailto:richard.clark@kaazing.com)
Github/Twitter: @rdclark
