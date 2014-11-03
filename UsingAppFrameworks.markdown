% Using Web Application Frameworks
% Richard Clark

# Single-Page Applications

## A trivial SPA

[Demo here](./examples/1.1-trivial-spa.html)

## A trivial SPA (HTML)

```
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

```
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
3. Model-View-Controller
4. Routing and History

## Multiple Screens in a Page

```
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

## Revised SPA (HTML)

```
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

## Revised SPA (script)

```
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

## Thank you!

[richard.clark@kaazing.com](mailto:richard.clark@kaazing.com)
Github/Twitter: @rdclark
