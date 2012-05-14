# Synopsis

Transparency is a (client-side) template engine which binds data to DOM with zero configuration. Just call `.render(data)`.

```html
<div id="template">
  <span class="greeting"></span>
  <span class="name"></span>
</div>
```
```js
var hello = {
  greeting: 'Hello',
  name:     'world!'
};

$('#template').render(hello);
```
```html
<div id="template">
  <span class="greeting">Hello</span>
  <span class="name">world!</span>
</div>
```

[![Build Status](https://secure.travis-ci.org/leonidas/transparency.png?branch=master)](http://travis-ci.org/leonidas/transparency)

## Features

* Data binding by convention - No extra markup in the views
* Collection rendering - No loops and partials
* Nested objects and collections - No configuration, just conventions
* Directives - No custom DSL, just functions
* Template caching - No manual template lookup/compilation/rendering
* Fast - In most real-world cases, it's faster than any other template engine or hand-crafted bindings (*)
* Compatible - Tested on IE6+, Chrome and Firefox

(*) Take with a grain of salt, as "real-world performance" isn't that easy to define or measure. Anyway, jsperf.com
should give an idea:
http://jsperf.com/transparency-vs-handlebars-finite-list/3
http://jsperf.com/transparency-vs-handlebars-infinite-list/3
http://jsperf.com/dom-vs-innerhtml-based-templating/366


If interested, see other performance tests at `browser` folder. See also
[Frequently asked questions](https://github.com/leonidas/transparency/wiki/Frequently-Asked-Questions).

## Try it

[Demo website](http://leonidas.github.com/transparency/) with interactive examples.

## Use it

Get the
[compiled and minified version](https://raw.github.com/leonidas/transparency/master/lib/transparency.min.js)
and include it to your application. jQuery is optional, but if you happen to use it, Transparency registers itself
as a plugin.

Node users can also install via NPM:

        npm install transparency

```html
<script src="js/jquery-1.7.1.min.js"></script>
<script src="js/transparency.min.js"></script>
```

For server-side use, see `spec` folder and the awesome [jsdom](https://github.com/tmpvar/jsdom) for the details.

## Examples

### Binding values

Transparency binds JavaScript objects to DOM a element by `id`, `class`,`name` attribute and
`data-bind` attribute. Values are escaped before rendering.

Template:

```html
<div id="container">
  <div id="hello"></div>
  <div class="goodbye"></div>
  <input type="text" name="greeting" />
  <button class="hi-button" data-bind="hi-label"></button>
</div>
```

Javascript:

```js
var hello = {
  hello:      'Hello',
  goodbye:    '<i>Goodbye!</i>',
  greeting:   'Howdy!',
  'hi-label': 'Terve!' // Finnish i18n
};

// with jQuery
$('#container').render(hello);

// ..or without
Transparency.render(document.getElementById('container'), hello);
```

Result:

```html
<div class="container">
  <div id="hello">Hello</div>
  <div class="goodbye">lt;i&gt;Goodbye!&lt;/i&gt;</div>
  <input type="text" name="greeting" value="Howdy!" />
  <button class="hi-button" data-bind="hi-label">Terve!</button>
</div>
```

### Rendering a list of models

Template:

```html
<ul id="activities">
  <li class="activity"></li>
</ul>
```

Javascript:

```js
var activities = [
  {activity: 'Jogging'},
  {activity: 'Gym'},
  {activity: 'Sky Diving'},
];

$('#activities').render(activities);

// or
Transparency.render(document.getElementById('activities'), activities);
```

Result:

```html
<ul id="activities">
  <li class="activity">Jogging</li>
  <li class="activity">Gym</li>
  <li class="activity">Sky Diving</li>
</ul>
```

#### Rendering a list of plain values

With plain values, Transparency can't guess how you would like to bind the data to DOM, so a bit of
help is needed. Directive functions are just for that. Directives are defined as an object of functions.
Keys are matched to dom elements and each model is passed for the functions as `this` parameter, one by one.

Access to the plain values within the directive function is provided through `this.value`. There's a whole
lot more to say about the directives, but that's all we need for now. For further examples, see
section [Directives](https://github.com/leonidas/transparency#directives).

Template:

```html
<div>
  <div class="comments">
    <label>Comments:</label><span class="comment"></span>
  </div>
</div>
```

Javascript:

```js
var comments, directives;

comments = ["That rules", "Great post!"];

// See section 'Directives' for the details
directives = {
  comment: function() {
    return this.value;
  }
};

$('.comments').render(comments, directives);
```

Result:

```html
<div>
  <div class="comments">
    <label>Comments</label><span class="comment">That rules</span>
    <label>Comments</label><span class="comment">Great post!</span>
  </div>
</div>
```

### Nested lists

Template:

```html
<div class="container">
  <h1 class="title"></h1>
  <p class="post"></p>
  <div class="comments">
    <div class="comment">
      <span class="name"></span>
      <span class="text"></span>
    </div>
  </div>
</div>
```

Javascript:

```js
var post = {
  title:    'Hello World',
  post:     'Hi there it is me',
  comments: [ {
      name: 'John',
      text: 'That rules'
    }, {
      name: 'Arnold',
      text: 'Great post!'
    }
  ]
};

$('.container').render(post);
```

Result:

```html
<div class="container">
  <h1 class="title">Hello World</h1>
  <p class="post">Hi there it is me</p>
  <div class="comments">
    <div class="comment">
      <span class="name">John</span>
      <span class="text">That rules</span>
    </div>
    <div class="comment">
      <span class="name">Arnold</span>
      <span class="text">Great post!</span>
    </div>
  </div>
</div>
```

### Nested objects

Template:

```html
<div class="person">
  <div class="firstname"></div>
  <div class="lastname"></div>
  <div class="address">
    <div class="street"></div>
    <div class="zip"><span class="city"></span></div>
  </div>
</div>
```

Javascript:

```js
var person = {
  firstname: 'John',
  lastname:  'Wayne',
  address: {
    street: '4th Street',
    city:   'San Francisco',
    zip:    '94199'
  }
};

$('.person').render(person);
```

Result:

```html
<div class="container">
  <div class="firstname">John</div>
  <div class="lastname">Wayne</div>
  <div class="address">
    <div class="street">4th Street</div>
    <div class="zip">94199<span class="city">San Francisco</span></div>
  </div>
</div>
```

### Directives

Directives are used for manipulating text or html values and setting element attributes.
In addition to having an access to the current data object through `this`, directives also receive
index number and current element as a parameter, which makes it easy to, e.g., add `even` and `odd` classes or
hide elements.

The return value of a directive function can be either string or object. If the return value is string, it is assigned
to the matching elements as text content. If the return value is an object, keys can be either `text`, `html` or any
valid element attribute, e.g., `class`, `src` or `href`. Values are assigned accordingly to the matching elements.

If both `text` and `html` are present, `html` overrides the text content.

Template:

```html
<div class="person">
  <span class="name"></span>
  <a class="email"></a>
</div>
```

Javascript:

```js
person = {
  firstname: 'Jasmine',
  lastname:  'Taylor',
  email:     'jasmine.tailor@example.com'
};

directives =
  name:  function(element, index) { return this.firstname + " " + this.lastname; }
  email: function(element, index) { return {href: "mailto:" + this.email}; }
};

$('.person').render(person, directives);
```

Result:

```html
<div class="person">
  <span class="name">Jasmine Taylor</span>
  <a class="email" href="mailto:jasmine.tailor@example.com">jasmine.tailor@example.com</a>
</div>
```

### Nested directives

Template:

```html
<div class="person">
  <span class="name"></span>
  <span class="email"></span>
  <div class="friends">
    <div class="friend">
      <span class="name"></span>
      <span class="email"></span>
    </div>
  </div>
</div>
```

Javascript:

```js
person = {
  firstname:  'Jasmine',
  lastname:   'Taylor',
  email:      'jasmine.taylor@example.com',
  friends:    [ {
      firstname: 'John',
      lastname:  'Mayer',
      email:     'john.mayer@example.com'
    }, {
      firstname: 'Damien',
      lastname:  'Rice',
      email:     'damien.rice@example.com'
    }
  ]
};

nameDecorator = function() { return {html: "<b>" + this.firstname + " " + this.lastname + "</b>"}; };

directives = {
  name: nameDecorator,
  friends: {
    name: nameDecorator
  }
};

$('.person').render(person, directives);
```

Result:

```html
<div class="person">
  <span class="name"><b>Jasmine Taylor</b></span>
  <span class="email">jasmine.taylor@example.com</span>
  <div class="friends">
    <div class="friend">
      <span class="name"><b>John Mayer</b></span>
      <span class="email">john.mayer@example.com</span>
    </div>
    <div class="friend">
      <span class="name"><b>Damien Rice</b></span>
      <span class="email">damien.rice@example.com</span>
    </div>
  </div>
</div>
```

## Debugging templates, data and Transparency

http://leonidas.github.com/transparency/ is great place to fiddle around with your data and templates.

To enable debug mode, call `.render` with a `{debug: true}` config and open the javascript console.

```javascript
$('container').render(data, {}, {debug: true});
```

## Getting help

* [FAQ](https://github.com/leonidas/transparency/wiki/Frequently-Asked-Questions)
* IRC: [freenode/#transparency.js](http://webchat.freenode.net/)
* Google Groups: transparencyjs@googlegroups.com

## Development environment

You need node.js 0.6.x and npm.

Install dependencies:

    npm install
    npm install -g uglify-js
    npm install -g coffee-script

Run tests

    npm test

Run tests during development for more verbose assertion output

    node_modules/jasmine-node/bin/jasmine-node --coffee --verbose spec

Generate Javascript libs

    cake build

Use [debugger statement to debug spec scripts](http://bytes.goodeggsinc.com/post/11587373922/debugging-jasmine-node-and-coffeescript-specs).

## Contributing

All the following are appreciated, in an asceding order of preference

1. A feature request or a bug report
2. Pull request with a failing unit test
3. Pull request with unit tests and corresponding implementation

In case the contribution is going to change Transparency API, please create a ticket first in order to discuss and
agree on design.

There's [an article](https://github.com/leonidas/codeblog/blob/master/2012/2012-01-13-implementing-semantic-anti-templating-with-jquery.md)
regarding the original design and implementation. It might be worth reading as an introduction.


