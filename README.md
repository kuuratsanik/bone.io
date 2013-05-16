
![bone.io](http://cdn.techpines.io/bone-io-github.png)

# What is Bone.io?

Bone.io is a VIO framework for building high performance, highly scalable single page HTML5 apps.

What does VIO mean?

* View   - HTML5 Document 
* Input  - Data entering the browser or server.
* Output - Data leaving the browser or server.


# Getting Started

### In the Browser

To install the browser library, just include "bone.io.js" and the two dependencies in your html:

```html
<script src="/jquery.js"></script>
<script src="/socket.io/socket.io.js"></script>
<script src="/bone.io.js"></script>
```

Bone.io depends on jquery for DOM manipulation, and socket.io for realtime websockets.

Here are a list of the browser components:

* [View](https://github.com/techpines/bone.io#view) - A view is based on a CSS selector and handles DOM events and DOM manipulations.
* [IO](https://github.com/techpines/bone.io#io-inputoutput) - Input/Output handles bi-directional data communication with various endpoints.
* [Router](https://github.com/techpines/bone.io#templates) - A router executes routes based on client side url changes.
* [Templates](https://github.com/techpines/bone.io#templates) - Use any templating system you like and plug into the bone.io templating infrastructure.

### On the Server

You can install the library with npm:

```bash
npm install bone.io
```

On the server you can just require it:

```js
var bone = require('bone.io');
```

Currently, the only server side component is for IO:

* [IO](http://bone.io) - Input/Output handles bi-directional data communication with connected devices.

## View

![bone.io](http://cdn.techpines.io/bone.io-view-architecture-github.png)

A view in bone.io is based on a `selector`.  They take care of DOM events and changes.

```js
bone.view.DataRow = bone.view('tr.data-row', {

    events: {
        'click .icon': 'open',
        'click .button.edit': 'edit',
        'click .button.delete': 'remove'
    },

    remove: function(event) {
        this.$el.remove();
    },

    edit: function(data) {
        ...
    }

});
```

Similar to backbone.js, views have a few handy properties attached to `this`:

* `el` - The dom element for the current action.
* `$el` - The jquery element for the current action.
* `$` - Short hand for `this.$el.find`, because scoping is good.
* `data` - Store and retrieve data on the element shortcut for `$this.$el.data`.

# IO (Input/Output)

IO is a crucial part of any realtime app, and for bone.io the following diagram illustrates the basic architecture:

![bone.io](http://cdn.techpines.io/bone.io-io-architecture-github.png)

### In the Browser

```js
bone.io.Search = bone.io('search', {
    adapter: 'socket.io',
    options: {
        socket: io.connect()
    },

    outbound: {
        middleware: [
            bone.io.middleware.session
        ]
        shortcuts: ['search'],

    inbound: {
        middleware: [
            bone.io.middleware.session
        ]
        results: function(data, context) {
            ...
        }
    }
});
```

The `adapter` tells bone.io what type of adapter is being used.  Currently only socket.io is supported.  The options object gives options for the adapter.  The `actions` give the names of functions that can be called on the IO object to make server calls.  For the socket.io adapter by default, these will be simple `socket.emit` calls.

You can also use middleware.  Middleware should define two functions `input` and `output`.  To be run for incoming data and outgoing data respectively.


### On the Server

Here's an example using `express`:

```js
var app = require('express')(),
    server = require('http').createServer(app),
    io = require('socket.io').listen(server);

bone.io.Search = bone.io('search', {
    adapter: 'socket.io',
    options: {
        sockets: io.sockets
    },

    outbound: {
        middleware: [
            bone.io.middleware.session
        ]
        shortcuts: ['results'],

    inbound: {
        middleware: [
            bone.io.middleware.session
        ]
        search: function(data, context) {
            ...
        }
    }
});
```

# Router

![bone.io](http://cdn.techpines.io/bone.io-router-architecture.png)

The router in bone.io is very similar to Backbone.js:

```js
bone.Router = bone.router({

    routes: {
        "help":                 "help",    // #help
        "search/:query":        "search",  // #search/bones
        "search/:query/p:page": "search"   // #search/bones/p7
    },

    middleware: [
        bone.router.middleware.session,
        bone.router.middleware.authenticate({
            redirect: '/login'
        })
    ],

    help: function() {
        ...
    },

    search: function(query, page) {
        ...
    }
});
```

The main difference is that the router supports the concept of middleware.  There are two router middleware classes.

* bone.router.middleware.session
* bone.router.middleware.authenticate

# Templates

To set templates for bone.io, you need to do the following:

```js
bone.templates = {
    layout: function() { return '<div id="content"></div><div id="sidebar"></div>'},
    table: function() { return '<table></table>'},
    list: function(data) { return '<ul><li>Bone.io</li><li>'+data+'</ul>'}
};
```

The templating language is up to you, but it must be compiled to javascript functions.

Then for mounting different parts of the page you can use the `mount` command:

```js
bone.mount('body', 'layout');
bone.mount('#content', 'table');
bone.mount('#sidebar', 'list', {data: 'hello', refresh: true});
```

Your mounts should cascade, which is that your highest level DOM nodes need to be mounted first.  In the example above, the `layout` template is attached to the `body` first, before the sub templates.

Typically, you should do this within your routes.  Mounts are not really intended for dynamic data, they are intended to be the static skeleton of a page.

Mounts are based on a single DOM element.  They remove existing DOM elements using jquery `remove` before doing their append.  They also will not render twice by default, this allows you to mount say, your navbar in every route, but then have it not rerender every time a route is called.

## Logging

By default, bone.io will log activity to the console.  Here are the log messages:

```
Inbound:   [eventSpace:action] data         # Data coming into the browser
Outbound:  [eventSpace:action] data         # Data going out of the browser
View:      [selector:action]   data         # DOM manipulation action
Interface: [selector:event]    eventTarget  # DOM events
Route:     [regex:url]                      # URL Route Change
```

_Warning_: You should never run the logger in production, it causes memory leaks.  It is strictly for development purposes.  In production, you need to turn it off:

```js
bone.log = undefined;
```

## License

©2013 Brad Carleton, Tech Pines LLC and available under the [MIT license](http://www.opensource.org/licenses/mit-license.php):

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in   the Software without restriction, including without limitation the rights to    use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies   of the Software, and to permit persons to whom the Software is furnished to do  so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all  copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR      IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,        FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE     AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER          LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,   OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE   SOFTWARE.
