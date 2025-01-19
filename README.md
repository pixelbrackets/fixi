<h1>&#x1F6B2; fixi</h1> 

fixi is an experimental, minimalist implementation
of [generalized hypermedia controls](https://dl.acm.org/doi/fullHtml/10.1145/3648188.3675127):

```html
<button fx-action="/content" fx-method="get" fx-target="#output" fx-swap="innerHTML">
    Get Content
</button>
<output id="output"></output>
```

This fixi-powered button will issue an HTTP `GET` request to the `/content` [relative URL](https://www.w3.org/TR/WD-html40-970917/htmlweb.html#h-5.1.2) 
and insert the HTML response to that request inside the output tag below it.

Philosophically, fixi is [scheme](https://scheme.org/) to [htmx](https://htmx.org)'s 
[common lisp](https://lisp-lang.org/): it is designed to be as simple as possible while still being useful for real 
world projects.

This means it does not have many of the features found in htmx, including:

* [request queueing & synchronization](https://htmx.org/attributes/hx-sync/)
* [extended selector support](https://htmx.org/docs/#extended-css-selectors)
* [extended event support](https://htmx.org/docs/#special-events)
* [attribute inheritance](https://htmx.org/docs/#inheritance)
* [request indicators](https://htmx.org/docs/#indicators)
* [CSS transitions](https://htmx.org/docs/#css_transitions)
* [history support](https://htmx.org/docs/#history)

fixi takes advantage of some modern JavaScript features not used by htmx:

* [`async` functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* the [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
* the use of [`MutationObserver`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) for monitoring when new content is added
* The [View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API) (used by htmx, but the sole mechanism for transitions in fixi)

## Minimalism

fixi an experiment in [software minimalism](https://ia601608.us.archive.org/8/items/pdfy-PeRDID4QHBNfcH7s/LeanSoftware_text.pdf).

A hard constraint on the project is that the _unminified_, _uncompressed_ size must be less than that of
the minified & compressed version of (the excellent) [preact](https://bundlephobia.com/package/preact) (currently 4.6Kb).

The current uncompressed size is `3578` bytes and the gzip'd size is `1369` bytes as determined by:

```bash
ls -l fixi.js | awk  '{print "raw:", $5}'; gzip -k fixi.js; ls -l fixi.js.gz | awk  '{print "gzipped:", $5}'; rm fixi.js.gz 
```

Another goal is that users should be able to [debug](https://developer.chrome.com/docs/devtools/javascript/) 
fixi easily, since it is small enough to use unminified.

The code style is [dense](fixi.js), but the statements are structured for debugging.

Like a fixed-gear bike fixi has very few moving parts:

* No dependencies (including test and development)
* No JavaScript API (beyond the [events](#events))
* No `fixi.min.js` file
* No `package.json`
* No build step

The fixi project consists of three files:

* [`fixi.js`](fixi.js), the code for the library
* [`test.html`](test.html), the test suite for the library
* This [`README.md`](README.md), which is the documentation

[`test.html`](test.html) is a stand-alone HTML file that implements its own visual testing infrastructure, mocking for 
`fetch()`, etc. and that can be opened using the `file:` protocol for easy testing.

## Installing

fixi is intended to be [vendored](https://macwright.com/2021/03/11/vendor-by-default), that is copied, into your 
project:

```bash
curl https://raw.githubusercontent.com/bigskysoftware/fixi/refs/tags/0.0.8/fixi.js >> fixi-0.0.8.js
```

The SHA256 of v0.0.8 is 

`TAG1J/4z7yGaN/rREdX+eVC7/7cgyvrVD/oI7bknPUQ=`

generated by the following command line script:

```bash
cat fixi.js | openssl sha256 -binary | openssl base64
```

You can also use the JSDelivr CDN for local development or testing:

```html

<script src="https://cdn.jsdelivr.net/gh/bigskysoftware/fixi@0.0.8/fixi.js"
        integrity="sha256-z0z+TAG1J/4z7yGaN/rREdX+eVC7/7cgyvrVD/oI7bknPUQ="></script>
```

fixi is not distributed via [NPM](https://www.npmjs.com/).

## API

The fixi api consists of six [attributes](#attributes) & nine [events](#events).

### Attributes

<table>
<thead>
<tr>
  <th>attribute</th>
  <th>description</th>
  <th>example</th>
</tr>
</thead>
<tbody>
<tr>
  <td>
    <code>fx&#8209;action</code>
  </td>
  <td>
    The URL to which an HTTP request will be issued, required
  </td>
  <td>
    <code>fx&#8209;action='/demo'</code>
  </td>
</tr>
<tr>
<td><code>fx&#8209;method</code></td>
<td>The HTTP Method that will be used for the request (case&#8209;insensitive), defaults to <code>GET</code></td>
<td><code>fx&#8209;method=&#39;DELETE&#39;</code></td>
</tr>
<tr>
<td><code>fx&#8209;target</code></td>
<td>A CSS selector specifying where to place the response HTML in the DOM, defaults to the current element</td>
<td><code>fx&#8209;target=&#39;#a&#8209;div&#39;</code></td>
</tr>
<tr>
<td><code>fx&#8209;swap</code></td>
<td>A string specifying how the content should be swapped into the DOM, one of <code>innerHTML</code>, <code>outerHTML</code>, <code>beforestart</code>, <code>afterstart</code>, <code>beforeend</code> or <code>afterend</code>.  <code>outerHTML</code> is the default.</td>
<td><code>fx&#8209;swap=&#39;innerHTML&#39;</code></td>
</tr>
<tr>
<td><code>fx&#8209;trigger</code></td>
<td>The event that will trigger a request.  Defaults to <code>submit</code> for <code>form</code> elements, <code>change</code> for <code>input</code>&#8209;like elements & <code>click</code> for all other elements</td>
<td><code>fx&#8209;trigger=&#39;click&#39;</code></td>
</tr>
<tr>
<td><code>fx&#8209;ignore</code></td>
<td>Any element with this attribute on it or on a parent will not be processed for <code>fx&#8209;*</code> attributes</td>
<td></td>
</tr>
</tbody>
</table>

#### Mechanism

fixi works in a straight-forward manner & I encourage you to look at [the source](fixi.js) as you read through this. The 
three components of fixi are:

* [Processing](#processing) elements in the DOM (or added to the DOM)
* Issuing HTTP [requests](#requests) in response to events
* [Swapping](#swapping) new HTML content into the DOM

##### Processing

The main entry point is found at the bottom of the file: on the `DOMContentLoaded` event fixi does two things:

* It establishes a MutationObserver to watch for newly added content with fixi-powered elements
* It processes any existing fixi-powered elements

fixi-powered elements are elements with the `fx-action` attribute on them.

When fixi finds one it will establish an event listener on that element that will dispatch an AJAX request via `fetch()` to
the URL specified by `fx-action`.

fixi will ignore any elements that have the `fx-ignore` attribute on them or on a parent.

The event that will trigger the request is determined by the `fx-trigger` attribute.

If that attribute is not present, the trigger defaults to:

* `submit` for `form` elements
* `change` for `input`, `select` & `textarea` elements
* `click` for everything else.

fixi will store the generated event listener in the `__fixi` property on the element.  To clear the behavior
on an element you may use the following code:

```js
  elt.removeEventListener(elt.__fixi.evt, elt.__fixi)
```

##### Requests

When a request is triggered, the [HTTP method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) of the request 
will be determined by the `fx-method` attribute.  If this attribute is not present, it will default to `GET`.  This 
attribute is case-insensitive.

fixi sends the [request header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) `FX-Request`, with the value 
`true`.  You can add or remove headers using the `evt.detail.cfg.headers` object, see the [`fx:config`](#fxconfig) event 
below.

If an element is within a form element or has a `form` attribute, the values of that form will be included with the
request.  Otherwise, if the element has a `name`, it's `name` & `value` will be sent with the request.   You can add or 
remove values using the `evt.detail.cfg.form` `FormData` object in the [`fx:config`](#fxconfig) event.

`GET` & `DELETE` requests will include values via query parameters, other request types will submit them as a form
encoded body.

Before a request is sent, the aforementioned [`fx:config`](#fxconfig) event is triggered, which can be used to configure
aspects of the request. If `preventDefault()` is invoked in this event, the request will not be sent.
The `evt.detail.cfg.drop` property will be set to `true` if there is an existing outstanding request associated with 
the element and, if it is not set to `false` in an event handler, the request will be dropped.

In the [`fx:config`](#fxconfig) event you can also set the property `evt.detail.cfg.confirm`, to a no-argument function.
This function can return a Promise and can be used to asynchronously confirm that the request should be issued:

```js
function showAsynConfirmDialog() {
    //... a Promise-based confirmation dialog...
}

document.addEventListener("fx:config", (evt) => {
    evt.detail.cfg.confirm = showAsynConfirmDialog;
})
```

Note that confirmation will only occur if the [`fx:config`](#fxconfig) event is not canceled and the request is not
dropped.

After the configuration step and the confirmation, if any, the [`fx:before`](#fxbefore) event will be triggered,
and then a `fetch()` will be issued.

When fixi receives a response it triggers the [`fx:after`](#fxafter) event.

In this event (and all following events) there are two more properties available on `evt.detail.cfg`:

* `response` the fetch [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) object
* `text` the text of the response

These can be inspected, and the `text` value can be changed if you want to transform it in some way.

If an error occurs the [`fx:error`](#fxerror) event will be triggered instead of `fx:after`.

The [`fx:finally`](#fxfinally) event will be triggered regardles of if an error occurs or not.

##### Swapping

fixi then swaps the response text into the DOM using the mechanism specified by `fx-swap`, targeting the element specified
by `fx-target`.  If the `fx-swap` attribute is not present, fixi will use `outerHTML`.

If the `fx-target` attribute is not present, it will target the element making the request.  

The swap mechanism and target can be changed in the request-related fixi events.

You can implement a custom swapping mechanism by setting a function into the `evt.detail.cfg.swap` property in one of
the request related events.  This function should take two arguments: the target element and the text to swap.

By default, swapping will occur in a [View Transition](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)
if they are available.  If you don't want this to occur, you can set the `evt.detail.cfg.transition` property to false
in one of the request-related events.

Finally, when the swap and any associated View Transitions have completed, the `fx:swapped` event will be triggered on 
the element.

#### Example

Here is an example using all the attributes available in fixi:

```html

<button fx-action="/demo" fx-method="GET" 
        fx-target="#output" fx-swap="innerHTML" 
        fx-trigger="click">
    Get Content
</button>
<output id="output" fx-ignore>--</output>
```

In this example, the button will issue a `GET` request to `/demo` and put the resulting HTML into the `innerHTML` of the
output element with the id `output`.

Because the `output` element is marked as `fx-ignore`, any `fx-action` attributes in the new content will be ignored.

### Events

fixi fires the following events, broken into two categories:

<table>
<thead>
<tr>
  <th>category</th>
  <th>event</th>
  <th>description</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="3"><a href="#initialization-events">initialization</a></td>
<td>
<a href="#fxinit"><code>fx:init</code></a>
</td>
<td>
triggered on elements that have a <code>fx-action</code> attribute and are about to be initialized by fixi
</td>
</tr>
<tr>
<td>
<a href="#fxinited"><code>fx:inited</code></a>
</td>
<td>
triggered on elements have been initialized by fixi (does <b>not</b> bubble)
</td>
</tr>
<tr>
<td>
<a href="#fxprocess"><code>fx:process</code></a>
</td>
<td>
triggered on new content added to the DOM that is not filtered via the <code>fx-ignore</code> attribute
</td>
</tr>
<tr>
<td rowspan="6"><a href="#fetch-events">fetch</a></td>
<td>
<a href="#fxconfig"><code>fx:config</code></a>
</td>
<td>
triggered on an element immediately when a request has been triggered, allowing users to configure the request
</td>
</tr>
<tr>
<td>
<a href="#fxbefore"><code>fx:before</code></a>
</td>
<td>
triggered on an element just before a <code>fetch()</code> request is made
</td>
</tr>
<tr>
<td>
<a href="#fxafter"><code>fx:after</code></a>
</td>
<td>
triggered on an element just after a <code>fetch()</code> request finishes normally but before content is swapped
</td>
</tr>
<tr>
<td>
<a href="#fxerror"><code>fx:error</code></a>
</td>
<td>
triggered on an element if an exception occurs during a <code>fetch()</code>
</td>
</tr>
<tr>
<td>
<a href="#fxfinally"><code>fx:finally</code></a>
</td>
<td>
triggered on an element after a request no matter what
</td>
</tr>
<tr>
<td>
<a href="#fxswapped"><code>fx:swapped</code></a>
</td>
<td>
triggered after the swap and any associated 
<a href="https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API">View Transition</a> has completed
</td>
</tr>
</tbody>
</table>

#### Initialization Events

##### `fx:init`

The `fx:init` event is triggered when fixi is processing a node with an `fx-action` attribute. One property
is available in `evt.detail`:

* `options` - An [Options Object](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#options)
              that will be passed to `addEventListener()`

If this event is cancelled via `preventDefault()`, the element will not be initialized by fixi.

##### `fx:inited`

The `fx:inited` event is triggered when fixi finished processing a node with an `fx-action` attribute.

Unlike other fixi events, this event does not bubble.

##### `fx:process`

The `fx:process` event is triggered when fixi is sees new content added to the DOM that is not marked as `fx-ignore` or
the child of a node marked as `fx-ignore`. 

If it is cancelled via `preventDefault()`, neither the element nor any of its children will not be initialized by fixi.

#### Fetch Events

fixi uses the [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) to issue HTTP requests. It
triggers events around this call that allow users to configure the request.

##### `fx:config`

The first event triggered is `fx:config`. This event can be used to configure the arguments passed to `fetch()` via the
fixi config object, which can be found at `evt.detail.cfg`.

This config object has the following properties:

* `trigger` - The event that triggered the request
* `method` - The HTTP Method that is going to be used
* `action` - The URL that the request is going to be issued to
* `headers` - An Object of name/value pairs to be sent as HTTP Request Headers
* `target` - The target element that will be swapped when the response is processed
* `swap` - The mechanism by which the element will be swapped
* `body` - The body of the request, if present, a FormData object that holds the data of the form associated with the
* `drop` - Whether this request will be dropped, defaults to `true` if a request is already in flight
* `transition` - Whether to use the View Transition API for swaps, defaults to `true`
* `preventTriggerDefault` - A boolean (defaults to true) that, if true, will call `preventDefault()` on the triggering
  event
* `signal` - The AbortSignal of the related AbortController for the request
* `abort()` - A function that can be invoked to abort the pending fetch request

Mutating the `method`, etc. properties of the `cfg` object will change the behavior of the request dynamically. Note
that the `cfg` object is passed to `fetch()` as the second argument of type `RequestInit`, so any properties you want
to set on the `RequestInit` may be set on the `cfg` object (e.g. `credentials`).

Another property available on the `detail` of this event is `requests`, which will be an array of any existing
outstanding requests for the element.

fixi does not implement request queuing like htmx does, but you can implement a simple
"replace existing requests in flight" rule with the following JavaScript:

```js
document.addEventListener("fx:config", (evt) => {
    evt.detail.cfg.drop = false;  // allow this request to be issued even if there are other requests
    evt.detail.requests.forEach((cfg) => cfg.abort()); // abort all existing requests
})
```

If you call `preventDefault()` on this event, no request will be issued.

##### `fx:before`

The  `fx:before` event is triggered just before a `fetch()` is issues. The config will again be available in the
`evt.detail.cfg` property, but after any confirmation is done.  The requests will also be available in 
`evt.detail.requests` and will include the current request.

If you call `preventDefault()` on this event, no request will be issued.

##### `fx:after`

The  `fx:after` event is triggered after a `fetch()` successfully completes. The config will again be available in the
`evt.detail.cfg` property, and will have two additional properties:

* `response` - The response object from the `fetch()` call
* `text` - The text of the response

At this point you may still mutate the `swap`, etc. attributes to affect swapping, and you may mutate the `text` if you
want to modify it is some way before it is swapped.

Calling `preventDefault()` on this event will prevent swapping from occurring.

##### `fx:error`

The  `fx:error` event is triggered when a network error occurs. In this case the `cfg.txt` will be set to a blank
string, and the `evt.detail.cfg` object is available for modification.

Calling `preventDefault()` on this event will prevent swapping from occurring. Note that `AbortError`s will also prevent
swapping.

##### `fx:finally`

The  `fx:finally` event is triggered regardless if an error occurs or not and can be used to clean up after a request.
Again the `evt.detail.cfg` object is available for modification.

##### `fx:swapped`

The  `fx:swapped` event is triggered once the swap and any associated View Transitions complete.

## Examples

Because fixi is minimalistic, the user is responsible for implementing many behaviors they want via events. We have
already seen how to abort an existing request that is already in flight.

Here are some additional examples of useful behaviors implemented using events:

### Disabling an Element During A Request

Here is an example that will use attributes to disable an element when a request is in flight:

```js
document.addEventListener("fx:init", (evt)=>{
  if (evt.target.matches("[ext-fx-disable]")){
    var disableSelector = evt.target.getAttribute('ext-fx-disable')
    evt.target.addEventListener('fx:before', ()=>{
      let disableTarget = disableSelector == "" ? evt.target : document.querySelector(disableSelector)
      disableTarget.disabled = true
      evt.target.addEventListener('fx:after', (afterEvt)=>{
        if (afterEvt.target == evt.target){
          disableTarget.disabled = true
        }
      })
    })
  }
})
```
```html
<button fx-action="/demo" ext-fx-disable>
  Demo
</button>
```

### Showing an Indicator During A Request

Here is an example that will use attributes a `fixi-request-in-flight` class to show an indicator of some kind:

```js
document.addEventListener("fx:init", (evt)=>{
  if (evt.target.matches("[ext-fx-indicator]")){
    var disableSelector = evt.target.getAttribute("ext-fx-indicator")
    evt.target.addEventListener("fx:before", ()=>{
      let disableTarget = disableSelector == "" ? evt.target : document.querySelector(disableSelector)
      disableTarget.classList.add("fixi-request-in-flight")
      evt.target.addEventListener("fx:after", (afterEvt)=>{
        if (afterEvt.target == evt.target){
          disableTarget.classList.remove("fixi-request-in-flight")
        }
      })
    })
  }
})
```
```html
<style>
  #indicator {
    display: none;
  }
  #indicator .fixi-request-in-flight {
    display: inline-block;
  }
</style>
<button fx-action="/demo" ext-fx-indicator="#indicator">
    Demo
   <img src="spinner.gif" id="indicator">
</button>
```

This example can be modified to use classes or other mechanisms for showing indicators as well.

### Debouncing A Request

Here is an implementation of the Active Search example from htmx done in fixi, utilizing the config `confirm` feature
to return a promise that resolves to `true` after the number of milliseconds specified by `fx-ext-debounce` if no
other triggers have occurred:

```js
document.addEventListener("fx:init", (evt)=>{
  let elt = evt.target
  if (elt.matches("[ext-fx-debounce]")){
    let latestPromise = null;
    elt.addEventListener("fx:config", (evt)=>{
      evt.detail.drop = false
      evt.detail.cfg.confirm = ()=>{
        let currentPromise = latestPromise = new Promise((resolve) => { 
          setTimeout(()=>{
            resolve(currentPromise === latestPromise)
          }, parseInt(elt.getAttribute("ext-fx-debounce")))
        })
        return currentPromise
      }
    })
  }
})
```
```html
<form action="/search" fx-action="/search" fx-target="#results" fx-swap="innerHTML">
  <input id="search" type="search" fx-action="/search" fx-trigger="input" ext-fx-debounce="200" fx-target="#results" fx-swap="innerHTML"/>
  <button type="submit">
    Search
  </button>
</form>
<table>
  ...
  <tbody id="results">
  ...
  </tbody>
</table>
```

### Lazy Loading

You can implement lazy loading using the [`fx:inited`](#fxinited) event:

```html
<div fx-action="/lazy-content" fx-trigger="fx:inited">
  Content Loading...
</div>
```

### Polling

htmx-style polling can be implemented in the following manner:

```js
document.addEventListener("fx:inited", (evt)=>{ // use fx:inited so the __fixi property is available
  let elt = evt.target
  if (elt.matches("[ext-fx-poll-interval]")){
    // squirrel away in case we want to call clearInterval() later
    elt.__fixi.pollInterval = setInterval(()=>{
      elt.dispatchEvent(new CustomEvent("poll"))
    }, parseInt(elt.getAttribute("ext-fx-poll-interval")))
  }
})
```
```html
<h3>Live News</h3>
<div fx-action="/news" fx-trigger="poll" ext-fx-poll-interval="300">
  ... initial content ...
</div>
```

## LICENCE

```
Zero-Clause BSD
=============

Permission to use, copy, modify, and/or distribute this software for
any purpose with or without fee is hereby granted.

THE SOFTWARE IS PROVIDED “AS IS” AND THE AUTHOR DISCLAIMS ALL
WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES
OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE
FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY
DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN
AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT
OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```

<h2>&#x1f480; <i>Memento mori</i></h2>

```js
/**
* Adding a single line to this file requires great internal reflection
* and thought.  You must ask yourself if your one line addition is so
* important, so critical to the success of the company, that it warrants
* a slowdown for every user on every page load.  Adding a single letter
* here could cost thousands of man hours around the world.
* 
* That is all.
*/
```

-- [A comment](https://www.youtube.com/watch?v=wHlyLEPtL9o&t=1528s) at the beginning of [Primer](https://gist.github.com/makinde/376039)
