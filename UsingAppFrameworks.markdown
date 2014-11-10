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

## Three things

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

## Practice (1)

Identify the screens in this page. What are your options for showing and hiding?

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

## Practice (2)

Explain what this code is doing

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

## Practice (3)

> 1. Run [1.1-trivial-spa.html](./examples/1.1-trivial-spa.html)
> 2. Implement a "back to the first" page link

# Templating

## "Logic-less" style

> + Uses named placeholders
    - [dust.js](http://akdubya.github.io/dustjs/): `Hello, my name is {name}`
    - [Mustache JS](https://github.com/janl/mustache.js): `Hello, my name is {{name.first}}`
+ Special constructs for lists: `{#names} Name: {first} Last:{last}   {/names}`

## With embedded logic

JSP/ERB style
    - [underscore.js](http://documentcloud.github.io/underscore/#template): `Hello, my name is <%- name %>`

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

> 1. Create a simple "hello world" app using an [underscore.js](http://documentcloud.github.io/underscore/#template) template.
> 2. Merge some data into the template and add to the page.
> 3. Create an array and render all the elements.
> 4. Render nested objects.
> 5. (Extra) Try with a logic-less library such as [dust.js](http://akdubya.github.io/dustjs/)

# Routing and History

## The challenge with URLs

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
    - `window.onhashchange = function() {
    - `// use window.location.hash...`
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

## Practice

> 1. Add history tracking to your simple two-page SPA (a fragment ID for each screen)
> 2. Can you implement bookmarks with the user's name?

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

## MVC in Backbone

> + Model-View-Controller
    - Model: The information to be displayed
    - View: How it's displayed (templates)
    - Controller: The code that links these

## The Model

+ Subclass of Backbone.Model
    - `var Stock = Backbone.Model.extend({ ...})`
    - "Frequently representing a row of data on your server" per [annotated source](http://backbonejs.org/docs/backbone.html)
- Emits `change` event on change
- Paired with a View for rendering

## Example model

```javascript
var Stock = Backbone.Model.extend({
  // typical data: {ticker: "AAPL", company: "Apple", price: 123.45}
  idAttribute: 'ticker',

  lastPrice: null,

  initialize: function () {
      this.attributes.change = '';
      this.lastPrice = null;
      this.on('change', this.onChange, this);
  },

  onChange: function () {
      var price = this.attributes.price;
      if (this.lastPrice) this.attributes.change = price - this.lastPrice;
      this.lastPrice = price;
  }

});
```

## Collections hold multiple models

```javascript
var StockCollection = Backbone.Collection.extend({

  // Reference to this collection's model.
  model: Stock,

  // Sort by name
  comparator: 'company'

});
```

## Views render Models & Collections

+ Subclass of Backbone.View
    - `var StockView = Backbone.View.extend({ ... })`
    - Listen for events from model (`change`) or collection (`add`, `remove`, `change`)

## HTML to contain view

``` html
<div id="stockapp">
    <header>
        <h1>Stock Ticker using JMS and Backbone</h1>
    </header>
    <table id="stocks">
        <thead>
        <tr>
            <th>Company</th>
            <th>Symbol</th>
            <th>Price</th>
            <th>Change</th>
        </tr>
        </thead>
        <tbody>
        <!-- Stocks will go here -->
        </tbody>
    </table>
</div>
```

## View template (underscore)

```html
<script type="text/template" id="stock-template">
  <td><%- company %></td>
  <td><%- symbol %></td>
  <td>$<%= price.toFixed(2) %></td>
  <td>$<%= change.toFixed(2) %></td>
</script>
```

## View code

```javascript
// The DOM element for a Stock...
var StockView = Backbone.View.extend({
  //... is a table row.
  tagName: 'tr',

  className: 'stock',

  // Cache the template function for a single item.
  template: _.template($('#stock-template').html()),

  // The StockView listens for changes to its model, re-rendering.
  initialize: function () {
      this.listenTo(this.model, 'change', this.render);
  },

  // Rebuild the table row.
  render: function () {
      this.$el.html(this.template(this.model.toJSON()));
      return this;
  }
});
```

## Rendering the collection

Wrap the table in a template

```html
<script type="text/template" id="portfolio-template">
  <table>
      <thead>
      <tr>
          <th>Company</th>
          <th>Symbol</th>
          <th>Price</th>
          <th>Change</th>
      </tr>
      </thead>
      <tbody>
      <!-- Stocks will go here -->
      </tbody>
  </table>
</script>
```

## Rendering the collection

```javascript
var StockCollectionView = Backbone.View.extend({

  tagName: "table",

  // Cache the template function for a single item.
  template: _.template($('#portfolio-template').html()),

  subviews: [],

  initialize: function () {
      this.subviews = [];
      this.$el.html(this.template());
      this.collection.on('add', this.added, this);
  },

  added: function (stock) {
      var index = this.collection.models.indexOf(stock);
      var subview = new StockView({model: stock});
      var html = subview.render().$el;
      // Patch the DOM directly to avoid removing and rebuilding the whole table
      if (index == this.subviews.length) {
          this.$el.find('tbody').append(html);
      } else {
          this.subviews[index].$el.before(html);
      }
      this.subviews.splice(index, 0, subview);
  },

  render: function () {
      return this;
  }
});
```

## The Controller

- Not a separate class
- Implemented as event listeners in the view (typically the application view)

## App view code

```javascript
var AppView = Backbone.View.extend({
  el: $("#stockapp"), // bind to existing in HTML

  initialize: function () {
      var stockTable = this.$el.find('#stocks');
      var newTable = PortfolioView.render().el;
      stockTable.html(newTable);

      Portfolio.on('add remove', this.render, this);
  },

  render: function () {
      PortfolioView.render();
      return this;
  }
});

var App = new AppView;
```

## Extra: Adding real-time data

+ Backbone assumes a REST model
    - User actions trigger a GET/PUT/POST
    - e.g. adding a TODO item
+ Add a data source:
    - Listen for incoming messages (Comet, WebSockets)
    - Emit an event for each
    - Controller code parses data, merges into model

## Real-time data via Kaazing JMS

```javascript
// Router for JMS messages
var JMSHandler = function (url) {
    this.url = url;
    this.connection = null;
    this.session = null;
}

_.extend(JMSHandler.prototype, Backbone.Events, {
    connection: null,
    session: null,

    connect: function () {
        var factory = new JmsConnectionFactory(this.url);
        var self = this;
        var future = factory.createConnection(function () {
            try {
                var connection = self.connection = future.getValue();
                var session = self.session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
                self.trigger('sessionReady', session);
                connection.start(function () {
                  this.trigger('connectionStarted', connection, session);
                });
            } catch (e) {
                alert(e.message);
            }
        });
    }
});

```

## Processing the data

In AppView

```javascript
  initialize: function () {
      var stockTable = this.$el.find('#stocks');
      var newTable = PortfolioView.render().el;
      stockTable.html(newTable);

      Portfolio.on('add remove', this.render, this);
      this.jms = new JMSHandler("ws://localhost:8000/jms");
      this.jms.on('sessionReady', this.onSessionReady, this);
      this.jms.connect();
  },

  onSessionReady: function (session) {
      this.topic = session.createTopic("/topic/stock");
      this.consumer = session.createConsumer(this.topic);
      this.consumer.setMessageListener(this.onStockMessage);
  },

  onStockMessage: function (message) {
      var text = message.getText();
      var attrs = Stock.prototype.parse(text);
      Portfolio.add(attrs, {merge: true});
  },

```

## Parsing the message

In Stock class

```javascript
  // The usual parse function is handed a JSON response and merely returns it.
  // We're getting a colon-delimited string.
  parse: function (text) {
      var a = text.split(':');
      return { company: (a[0]), ticker: (a[1]), price: (parseFloat(a[2])) };
  },

```

## Not discussed

- Saving via `Backbone.sync`
- Routing via `Backbone.Router`
- History management via `Backbone.History`

## Practice

> 1. Implement "Hello" SPA using Backbone
> 2. Create a page with a button. Show the number of clicks
> 3. [Getnames.org](http://www.geonames.org) implements a search page that returns JSON. Write your own front-end for it.
    - Hint: The [Neighborhood](https://github.com/rdclark/neighhborhood) sample app implements this search using jQuery's AJAX support. See [js/geo_CORS.js](https://github.com/rdclark/neighhborhood/blob/master/js/geo_CORS.js)

# Extra: Javascript Promises

## Promises simplify asynchronous code

Before:
```javascript
step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // Do something with value4
            });
        });
    });
});
```

## With promises (using Q)

Using the [Q](https://github.com/kriskowal/q) library

```javascript
Q.fcall(promisedStep1)
.then(promisedStep2)
.then(promisedStep3)
.then(promisedStep4)
.then(function (value4) {
    // Do something with value4
})
.catch(function (error) {
    // Handle any error from all above steps
})
.done();
```

## Networking without promises

```javascript
var ws = new WebSocket("ws://echo.websocket.org");
ws.onopen = function() {
  ws.send("I hear an echo");
}

ws.onerror = function(err) {
  console.log(err.data);
  ws.close();
}

ws.onmessage = function(evt) {
  console.log(evt.data);
}

function send(message) {
  ws.send(message):
}

```

## Easier networking with promises

I would like to do this:

```javascript
  open("ws://echo.websocket.org")
    .then(send("hello, world"))
    .then(read)
    .then(print)
    .catch(function(error) {
       console.log("ERROR " + error)
     })
    .finally(close);

```

## Wrapping WebSocket in promises

```javascript
function open(wsurl) {
  return new Q.Promise(resolve, reject) {
    var ws = new WebSocket(wsurl);

    ws.onopen = function() {
      resolve(ws);
    }

    ws.onerror = function(err) {
      reject(new Error(err.data));
    }
}
```
Usage:
```javascript
  open("ws://echo.websocket.org").then(...
```

## Waiting for an event

```javascript
function read(ws) {
  return new Q.Promise(resolve) {

    ws.onmessage = function(evt) {
      resolve(evt.data);
    }
  }
}
```

## Utility functions

```javascript
function close(ws) { ws.close() };

function send(message) return { function(ws) { ws.send(message); return ws }};

function print(message) {
  console.log(message);
}
```

## There's a problem

```javascript
  open("ws://echo.websocket.org") // returns ws
    .then(send("hello, world"))   // consumes ws, returns ws
    .then(read)                   // consumes ws, returns string/binary
    .then(print)                  // consumes string/binary, returns nothing
    .catch(function(error) {
       console.log("ERROR " + error)
     })
    .finally(close);              // consumes ws...which may not be available!

```

## One way to resolve this

```javascript
  var socket;
  open("ws://echo.websocket.org")
    .then(function(ws) { socket = ws }) // returns socket as a side-effect
    .then(send("hello, world"))   		// and send uses that
    .then(read)							// so does read
    .then(print)
    .catch(function(error) {
       console.log("ERROR " + error)
     })
    .finally(function() {
       if (socket) socket.close();
     });

```

## When to use promises?

- Talking to external services (e.g. via AJAX)
- Local operations that take time (e.g. image loading)

## Image loading example

```javascript
function getImage(url) {

  return new Q.Promise(resolve, reject) {
    var image = new Image(url);
    image.onload = function() {
      resolve(image);
    }
    image.onerror = function(err) {
      reject(err);
    }
  }
}
```

```javascript
  getImage("./button.png").then(function(image) { ... });
```

## Other promise tricks

> 1. `promise.all([array of functions])` acts like `map`, runs all concurrently.
> 2. Serial actions:

```javascript
  var messages = ["Hello", "goodbye"];
  messages.reduce(
    function(ws, msg) {
      ws.send(message); return ws
    },
    open("ws://echo.websocket.org")
  )
```

# Thank you!

## Making contact

[richard.clark@kaazing.com](mailto:richard.clark@kaazing.com)
Github/Twitter: @rdclark
