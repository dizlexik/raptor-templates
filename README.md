Raptor Templates
================

Raptor Templates is an extensible, streaming, asynchronous, [high performance](https://github.com/raptorjs3/templating-benchmarks), _HTML-based_ templating language that can be used in Node.js or in the browser. Raptor Templates was founded on the philosophy that an HTML-based templating language is more natural and intuitive for generating HTML.  Because the Raptor Templates compiler understands the structure of the HTML document, the directives in template files are less obtrusive and more powerful. In addition, Raptor Templates allows you to introduce your own custom tags and custom attributes to extend the HTML grammar (much like [Web Components](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)—only you can use it now).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Design Philosophy](#design-philosophy)
- [Sample Code](#sample-code)
- [Installation](#installation)
- [Usage](#usage)
	- [Template Rendering](#template-rendering)
		- [Callback API](#callback-api)
		- [Streaming API](#streaming-api)
		- [Asynchronous Render Context API](#asynchronous-render-context-api)
	- [Browser-side Rendering](#browser-side-rendering)
		- [Using the RaptorJS Optimizer](#using-the-raptorjs-optimizer)
		- [Using Browserify](#using-browserify)
	- [Template Compilation](#template-compilation)
		- [Sample Compiled Template](#sample-compiled-template)
- [Language Guide](#language-guide)
	- [Template Directives Overview](#template-directives-overview)
	- [Text Replacement](#text-replacement)
	- [Expressions](#expressions)
	- [Includes](#includes)
	- [Variables](#variables)
	- [Conditionals](#conditionals)
		- [if...else-if...else](#ifelse-ifelse)
		- [choose…when…otherwise](#choose…when…otherwise)
		- [Shorthand conditionals](#shorthand-conditionals)
	- [Looping](#looping)
		- [for](#for)
			- [Loop Status Variable](#loop-status-variable)
			- [Loop Separator](#loop-separator)
			- [Property Looping](#property-looping)
	- [Macros](#macros)
		- [def](#def)
		- [invoke](#invoke)
	- [Structure Manipulation](#structure-manipulation)
		- [attrs](#attrs)
		- [content](#content)
		- [replace](#replace)
		- [strip](#strip)
	- [Comments](#comments)
	- [Helpers](#helpers)
	- [Custom Tags and Attributes](#custom-tags-and-attributes)
	- [Layout Taglib](#layout-taglib)
- [Custom Taglibs](#custom-taglibs)
	- [Tag Renderer](#tag-renderer)
	- [raptor-taglib.json](#raptor-taglibjson)
		- [Sample Taglib](#sample-taglib)
		- [Defining Tags](#defining-tags)
			- [Defining Attributes](#defining-attributes)
		- [Scanning for Tags](#scanning-for-tags)
		- [Nested Tags](#nested-tags)
	- [Taglib Discovery](#taglib-discovery)
- [FAQ](#faq)
- [Discuss](#discuss)
- [Contributors](#contributors)
- [Contribute](#contribute)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Design Philosophy

* __Readable:__ Templates should be as close to the output HTML as possible to keep templates readable. Cryptic syntax and symbols should be avoided.
* __Simple:__ The number of new concepts should be minimized and complexity should be avoided.
* __Extensible:__ The template engine should be easily extensible at both compile-time and runtime.
* __High Performance:__ Runtime and compiled output should be optimized for low CPU and memory usage and have a small footprint. All expressions should be native JavaScript to avoid runtime interpretation.
* __Not Restrictive:__ Whether or not to go less logic or more logic is up to the developer.
* __Asynchronous and Streaming Output:__ It should be possible to render HTML out-of-order, but the output HTML should be streamed out in the correct order. This minimizes idle time and reduces the time to first byte.
* __Intuitive:__ The templating engine should introduce as few surprises as possible.
* __Browser and Server Compatibility:__ Templates should compile down to JavaScript that can be executed on both the server and the client.
* __Debuggable:__ Compiled JavaScript should be debuggable and readable.
* __Compile-Time Checks:__ Syntax, custom tags and custom attributes should be validated at compile-time.
* __Tools Support:__ Tools should be enabled to offer auto-completion and validation for improved productivity and safety.
* __Modular:__ Runtime and compiled templates should be based on CommonJS modules for improved dependency management. Template dependencies (such as custom tags) should be resolved based on a template's file system path instead of relying on a shared registry.

# Sample Code

A basic template with text replacement, looping and conditionals is shown below:

```html
Hello ${data.name}!

<ul c-if="notEmpty(data.colors)">
    <li style="color: $color" c-for="color in data.colors">
        $color
    </li>
</ul>
<div c-else>
    No colors!
</div>
```

The template can then be rendered as shown in the following sample code:

```javascript
var templatePath = require.resolve('./hello.rhtml');
var template = require('raptor-templates').load(templatePath);

template.render({
        name: 'World',
        colors: ["red", "green", "blue"]
    },
    function(err, output) {
        console.log(output);
    });
```

The output of running the above program will be the following (formatted for readability):
```html
Hello World!

<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

For comparison, given the following input data consisting of an empty array of colors:

```javascript
{
    name: 'World',
    colors: []
}
```

The output would be the following:

```html
Hello World!

<div>No colors!</div>
```

The streaming API can be used to stream the output to an HTTP response stream or any other writable stream. For example, with Express:

```javascript
var template = require('raptor-templates').load(require.resolve('./template.rhtml'));

app.get('/profile', function(req, res) {
    template.stream({
            name: 'Frank'
        })
        .pipe(res);
});
```

Raptor Templates also supports custom tags so you can easily extend the HTML grammar to support things like the following:

```html
Welcome to Raptor Templates!

<ui-tabs>
    <ui-tab label="Home">
        Content for Home
    </ui-tab>
    <ui-tab label="Profile">
        Content for Profile
    </ui-tab>
    <ui-tab label="Messages">
        Content for Messages
    </ui-tab>
</ui-tabs>
```

The above template is a very simple way to generate the much more complicated HTML output shown below:

```html
<div class="tabs">
    <ul class="nav nav-tabs">
        <li class="active">
            <a href="#tab0" data-toggle="tab">Home</a>
        </li>
        <li>
            <a href="#tab1" data-toggle="tab">Profile</a>
        </li>
        <li>
            <a href="#tab2" data-toggle="tab">Messages</a>
        </li>
    </ul>
    <div class="tab-content">
        <div id="tab0" class="tab-pane active">
            Content for Home
        </div>
        <div id="tab1" class="tab-pane">
            Content for Profile
        </div>
        <div id="tab2" class="tab-pane">
            Content for Messages
        </div>
    </div>
</div>
```

The custom tags encapsulate rendering logic and help avoid repeating the same HTML (and potentially the same mistakes).

# Installation

To install the `raptor-templates` module into your project you should use the following command:
```bash
npm install raptor-templates --save
```

To install the optional `rhtmlc` command line interface to compile templates you can use the following command:
```bash
npm install raptor-templates --global
```

# Usage

## Template Rendering

### Callback API
```javascript
var template = require('raptor-templates').load('template.rhtml');

template.render({
        name: 'Frank',
        count: 30
    },
    function(err, output) {
        if (err) {
            console.error('Rendering failed');
            return;
        }

        console.log('Output HTML: ' + output);
    });
```

### Streaming API
```javascript
var template = require('raptor-templates').load('template.rhtml');
var out = require('fs').createWriteStream('index.html', 'utf8');

// Render the template to 'index.html'
template.stream({
        name: 'Frank',
        count: 30
    })
    .pipe(out);
```


### Asynchronous Render Context API

```javascript
var raptorTemplates = require('raptor-templates');
var template = raptorTemplates.load('template.rhtml');

var out = require('fs').createWriteStream('index.html', 'utf8');

var context = raptorTemplates.createContext(out);

// Render the first chunk asynchronously (after 1s delay):
var asyncContext = context.beginAsync();
setTimeout(function() {
    asyncContext.write('BEGIN ');
    asyncContext.end();
}, 1000);

// Render the template to the existing render context:
template.render({
        name: 'World'
    },
    context);

// Write the last chunk synchronously:
context.write(' END');

// End the rendering context
context.end();
```

Despite rendering the first chunk asynchronously, the above program will stream out the output in the correct order to `index.html`:

```html
BEGIN Hello World! END
```

For more details, please see the documentation for the [raptor-render-context](https://github.com/raptorjs3/raptor-render-context) module.

## Browser-side Rendering

Given the following module code that will be used to render a template on the client-side:

_run.js_:
```javascript

var templatePath = require.resolve('./hello.rhtml');
var template = require('raptor-templates').load(templatePath);

templatePath.render({
        name: 'John'
    },
    function(err, output) {
        document.body.innerHTML = output;
    });
```

You can then bundle up the above program for running in the browser using either [raptor-optimizer](https://github.com/raptorjs3/raptor-optimizer) (recommended) or [browserify](https://github.com/substack/node-browserify).


### Using the RaptorJS Optimizer

The `raptor-optimizer` CLI can be used to generate resource bundles that includes all application modules and all referenced Raptor Template files using a command similar to the following:
```bash
# First install the raptor-optimizer
npm install raptor-optimizer --global

raptor-optimizer --main run.js --name my-page
```

This will produce a JSON file named `build/my-page.html.json` that contains the HTML markup that should be used to include the required JavaScript and CSS resources that resulted from the page optimization. 

Alternatively, you can inject the HTML markup into a static HTML file using the following command:

```bash
raptor-optimizer --main run.js --name my-page --inject-into my-page.html
```


### Using Browserify

The `rhtmlify` transform for browserify must be enabled in order to automatically compile and include referenced Raptor Template files.

```bash
# Install the rhtmlify plugin from npm:
npm install rhtmlify --save

# Build the browser bundle:
browserify -t rhtmlify run.js > browser.js
```


## Template Compilation

The Raptor Templates compiler produces a Node.js-compatible, CommonJS module as output. This output format has the advantage that compiled template modules can benefit from a context-aware module loader and templates can easily be transported to work in the browser using the [RaptorJS Optimizer](https://github.com/raptorjs3/raptor-optimizer) or [Browserify](https://github.com/substack/node-browserify).

The `raptor-templates` module will automatically compile templates loaded by your application on the server, but you can also choose to precompile all templates. This can be helpful as a build or test step to catch errors early.

You can either use the command line interface or the JavaScript API to compile a Raptor Template file. To use the CLI you must first install the `raptor-templates` module globally using the following command:
```bash
npm install raptor-templates --global
```

You can then compile single templates using the following command:
```bash
rhtmlc hello.rhtml
```

This will produce a file named `hello.rhtml.js` next to the original file.

You can also recursively compile all templates in the current directory (the `node_modules` and `.*` directories will be ignored by default)

```bash
rhtmlc .
```

You can also specify multiple directories or files
```bash
rhtmlc foo/ bar/ template.rhtml
```

To delete all of the generated `*.rhtml.js` files you can add the `--clean` argument. For example:
```bash
rhtmlc . --clean
```


Alternatively, you can use the JavaScript API to compile a module as shown in the following sample code:
```javascript
require('raptor-templates/compiler').compileFile(path, function(err, src) {
    // Do something with the compiled output 
});
```

### Sample Compiled Template
```javascript
module.exports = function create(__helpers) {
  var empty = __helpers.e,
      notEmpty = __helpers.ne,
      escapeXml = __helpers.x,
      forEach = __helpers.f,
      escapeXmlAttr = __helpers.xa;

  return function render(data, context) {
    context.w('Hello ' +
      escapeXml(data.name) +
      '! ');

    if (notEmpty(data.colors)) {
      context.w('<ul>');

      forEach(data.colors, function(color) {
        context.w('<li style="color: ' +
          escapeXmlAttr(color) +
          '">' +
          escapeXml(color) +
          '</li>');
      });

      context.w('</ul>');
    }
    else {
      context.w('<div>No colors!</div>');
    }
  };
}
```

The compiled output is designed to be both extremely readable and minifiable. The minified code is shown below:


```javascript
module.exports=function(a){var d=a.ne,c=a.x,e=a.f,f=a.xa;return function(a,b){b.w("Hello "+c(a.name)+"! ");d(a.colors)?(b.w("<ul>"),e(a.colors,function(a){b.w('<li style="color: '+f(a)+'">'+c(a)+"</li>")}),b.w("</ul>")):b.w("<div>No colors!</div>")}};
```

_File size: 190 bytes gzipped (251 bytes uncompressed)_


# Language Guide

## Template Directives Overview

Almost all of the Raptor templating directives can be used as either an attribute or as an element. For example:

_Applying directives using attributes:_
```html
<!-- Colors available -->
<ul c-if="notEmpty(colors)">
    <li c-for="color in colors">
        $color
    </li>
</ul>
 
<!-- No colors available -->
<div c-if="empty(colors)">
    No colors!
</div>
```

_Applying directives using elements:_
```html
<!-- Colors available -->
<c-if test="notEmpty(colors)">
    <ul>
        <c-for each="color in colors">
            <li>
                $color
            </li>
        </c-for>
    </ul>
</c-if>
  
<!-- No colors available -->
<c-if test="empty(colors)">
    <div>
        No colors!
    </div>
</c-if>
```

The disadvantage of using elements to control structural logic is that they change the nesting of the elements which can impact readability. For this reason it is often more suitable to apply directives as attributes.

## Text Replacement

Dynamic text is supported using either `$<variable-reference>` or `${<javascript-expression}`.

Examples:
```html
Hello $data.name!
Hello ${data.name}!
Hello ${data.name.toUpperCase()}!
```

By default, all special HTML characters will be escaped in dynamic text to prevent Cross-site Scripting (XSS) Attacks. To disable HTML escaping, you can use `$!` as shown in the following sample code:

```html
Hello $!{data.name}! <!-- Do not escape -->
```

If necessary, you can escape `$` using a forward slash to have it be treated as text instead of a placeholder token:

```html
Test: \${hello}
<!-- Rendered Ouptut: 
Test: ${hello}
-->
```

## Expressions

Wherever expressions are allowed, they are treated as JavaScript expressions and copied out to the compiled template verbatim. However, you can choose to use alternate versions of the following JavaScript operators:

JavaScript Operator | Raptor Equivalent
------------------- | -----------------
`&&` 	            | `and`
<code>&#124;&#124;</code>                | `or`
`===`               | `eq`
`!==`               | `ne`
`<`                 | `lt`
`>`                 | `gt`
`<=`                | `le`
`>=`                | `ge`

For example, both of the following are valid and equivalent:

```html
<div c-if="searchResults.length > 100">
    Show More
</div>
```

```html
<div c-if="searchResults.length gt 100">
    Show More
</div>
```

## Includes

Other Raptor Template files can be included using the `<c-include>` tag and a relative path. For example:

```html
<c-include template="./greeting.rhtml" name="Frank" count="30"/>
```

## Variables

Input data passed to a template is made available using a special `data` variable. It's possible to declare your own variables as shown in the following sample code:

```html
<c-var name="name" value="data.name.toUpperCase()" />
```

## Conditionals

### if...else-if...else

Any element or fragment of HTML can be made conditional using the `c-if`, `c-else-if` or `c-else` directive.

_Applied as attributes:_
```html
<!--Simple if-->
<div c-if="someCondition">
    Hello World
</div>

<!--Complex if-->
<div c-if="test === 'a'">
    A
</div>
<div c-else-if="test === 'b'">
    B
</div>
<div c-else-if="test === 'c'">
    C
</div>
<div c-else>
    Something else
</div>
```

_Applied as elements:_
```html
<!-- Colors available -->
<!--Simple if-->
<c-if test="someCondition">
    <div>
        Hello World
    </div>
</c-if>

<!--Complex if-->
<c-if test="test === 'a'">
    <div>
        A
    </div>
</c-if>
<c-else-if test="test === 'b'">
    <div>
        B
    </div>
</c-else-if>
<c-else-if test="test === 'c'">
    <div>
        C
    </div>
</c-else-if>
<c-else>
    <div>
        Something else
    </div>
</c-else>
```

### choose…when…otherwise

The `c-choose` directive, in combination with the directives `c-when` and `c-otherwise` provides advanced conditional processing for rendering one of several alternatives. The first matching `c-when` branch is rendered, or, if no `c-when` branch matches, the `c-otherwise` branch is rendered.

_Applied as an attribute:_
```html
<c-choose>
    <c-when test="myVar === 'A'">
        <div>A</div>
    </c-when>
    <c-when test="myVar === 'B'">
        <div>B</div>
    </c-when>
    <c-otherwise>
        <div>Something else</div>
    </c-otherwise>
<c-choose>
```

_Applied as an element:_
```html
<c-choose>
    <div c-when="myVar === 'A'">
        A
    </div>
    <div c-when="myVar === 'B'">
        B
    </div>
    <div c-otherwise="">
        Something else
    </div>
<c-choose>
```

### Shorthand conditionals

Shorthand conditionals allow for conditional values inside attributes or wherever expressions are allowed. Shorthand conditionals are of the following form:
`{?<expression>;<true-template>[;<false-template>]}`

For example:

```html
<div class="{?active;tab-active}">Hello</div>
```
With a value of `true` for `active`, the output would be the following:

```html
<div class="tab-active">Hello</div>
```

With a value of `false` for `active`, the output would be the following:

```html
<div>Hello</div>
```

_NOTE: If the expression inside an attribute evaluates to `null` or an empty string then the attribute is not included in the output._

As shown in the previous example, the "else" block for shorthand conditionals is optional. The usage of an else block is shown below:

```html
<div class="{?active;tab-active;tab-inactive}">Hello</div>
```

With a value of `false` for `active`, the output would be the following:

```html
<div class="tab-inactive">Hello</div>
```

## Looping

### for

Any element can be repeated for every item in an array using the `c-for` directive. The directive can be applied as an element or as an attribute.

_Applied as an attribute:_

```html
<ul>
    <li c-for="item in items">${item}</li>
</ul>
```

_Applied as an element:_

```html
<ul>
    <c-for each="item in items">
        <li>${item}</li>
    </c-for>
</ul>
```


Given the following value for items:

```javascript
["red", "green", "blue"]
```

The output would be the following:

```html
<ul>
    <li>red</li>
    <li>green</li>
    <li>blue</li>
</ul>
```

#### Loop Status Variable

The `c-for` directive also supports a loop status variable in case you need to know the current loop index. For example:

```html
<ul>
    <li c-for="color in colors; status-var=loop">
        ${loop.getIndex()+1}) $color
        <c-if test="loop.isFirst()"> - FIRST</c-if>
        <c-if test="loop.isLast()"> - LAST</c-if>
    </li>
</ul>
```

#### Loop Separator

```html
<c-for each="color in colors" separator=", ">$color</c-for>
    
<div>
    <span c-for="color in colors; separator=', '" style="color: $color">$color</span>
</div>
```

#### Property Looping

```html
<ul>
    <li c-for="(name,value) in settings">
        <b>$name</b>:
        $value
    </li>
</ul>
```

## Macros

Parameterized macros allow for reusable fragments within an HTML template. A macro can be defined using the `<c-def>` directive.

### def

The `<c-def>` directive can be used to define a reusable function within a template.

```html
<c-def function="greeting(name, count)">
    Hello $name! You have $count new messages.
</c-def>
```

The above macro can then be invoked as part of any expression. Alternatively, the [`<c-invoke>`](#invoke) directive can be used invoke a macro function using named attributes. The following sample template shows how to use macro functions inside expressions:

```html
<c-def function="greeting(name, count)">
    Hello $name! You have $count new messages.
</c-def>

<p>
    ${greeting("John", 10)}
</p>
<p>
    ${greeting("Frank", 20)}
</p>
```

### invoke

The `<c-invoke>` directive can be used to invoke a function defined using the `<c-def>` directive or a function that is part of the input to a template. The `<c-invoke>` directive allows arguments to be passed using element attributes, but that format is only supported for functions that were previously defined using the `<c-def>` directive.

```html
<c-def function="greeting(name, count)">
    Hello ${name}! You have ${count} new messages.
</c-def>
 
<c-invoke function="greeting" name="John" count="${10}"/>
<c-invoke function="greeting('Frank', 20)"/> 
```

The output for the above template would be the following:

```html
<p>
    Hello John! You have 10 new messages.
</p>
<p>
    Hello Frank! You have 20 new messages.
</p>
```

_NOTE:_ By default, the arguments will be of type "string" when using `<c-invoke>.` However, argument attributes support JavaScript expressions which allow for other types of arguments. Example:
```html
count="10" <!-- string argument -->
count="${10}"  <!-- number argument -->
```


## Structure Manipulation

### attrs

The `c-attrs` attribute allows attributes to be dynamically added to an element at runtime. The value of the c-attrs attribute should be an expression that resolves to an object with properties that correspond to the dynamic attributes. For example:

```html
<div c-attrs="myAttrs">
    Hello World!
</div>
```

Given the following value for the `myAttrs` variable:

```javascript
{style: "background-color: #FF0000;", "class": "my-div"}
```

The output would then be the following:

```html
<div style="background-color: #FF0000;" class="my-div">
    Hello World!
</div>
```

### content

This directive replaces any nested content with the result of evaluating the expression:

```html
<ul>
    <li c-content="myExpr">Hello</li>
</ul>
```

Given a value of `"Bye!"` for the value of `myExpr`, the output of the above template would be the following:

```html
<ul>
    <li>Bye!</li>
</ul>
```

### replace

This directive replaces the element itself with the result of evaluating the expression:

```html
<div>
    <span c-replace="myExpr">Hello</span>
</div>
```

Given a value of "Bye!" for the value of "myExpr", the output of the above template would be the following:

```html
<div>
    Bye!
</div>
```

### strip

This directive conditionally strips the top-level element from the output. If the expression provided as the attribute value evaluates to true then the element is stripped from the output:

```html
<div>
    <span c-strip="true"><b>Hello</b></span>
</div>
```

_Output:_

```html
<div>
    <b>Hello</b>
</div>
```

## Comments

Standard HTML comments can be used to add comments to your template. The HTML comments will not show up in the rendered HTML.

Example comments:

```html
<!-- This is a comment that will not be rendered -->
<h1>Hello</h1>
```

If you would like for your HTML comment to show up in the final output then you can use the custom `html-comment` tag:
```html
<html-comment>This is a comment that *will* be rendered</html-comment>
<h1>Hello</h1>
```

Output:

```html
<!--This is a comment that *will* be rendered-->
<h1>Hello</h1>
```

## Helpers

Since Raptor Template files compile into CommonJS modules, any Node.js module can be "imported" into a template for use as a helper module. For example, given the following helper module:

_src/util.js_:
```javascript
exports.reverse = function(str) {
    var out = "";
    for (var i=str.length-1; i>=0; i--) {
        out += str.charAt(i); 
    }
    return out;
};
```

The above module can then be imported into a template as shown in the following sample template:

_src/template.rhtml_:
```html
<c-require module="./util" var="util" />

<div>${util.reverse('reverse test')}</div>
```

The `c-require` directive is just short-hand for the following:
```html
<c-var name="util" value="require('./util')" />
<div>${util.reverse('reverse test')}</div>
```

## Custom Tags and Attributes

Raptor Templates supports extending the language with custom tags and attributes. A custom tag or a custom attribute __must have at least one dash__ to indicate that is not part of the standard HTML grammar.

Below illustrates how to use a simple custom tag:

```html
<div>
    <my-hello name="World"/>
</div>
```

The output of the above template might be the following:

```html
<div>
    Hello World!
</div>
```

For information on how to use and create taglibs, please see the [Custom Taglibs](#custom-taglibs) section below.

## Layout Taglib

Raptor Templates provides a `layout` taglib to support separating out layout from content. The usage of of the `layout` taglib is shown in the sample code below:
   
_default-layout.rhtml:_

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><layout-placeholder name="title"/></title>
</head>
<body>
    <h1 c-if="data.showHeader !== false">
        <layout-placeholder name="title"/>
    </h1>
    <p>
        <layout-placeholder name="body"/>
    </p>
    <div>
        <layout-placeholder name="footer">
            Default Footer
        </layout-placeholder>
    </div>
</body>
</html>
```

_Usage of `default-layout.rhtml`:_

```html
<layout-use template="./default-layout.rhtml" show-header="$true">
    <layout-put into="title">My Page</layout-put>
    <layout-put into="body">BODY CONTENT</layout-put>
</layout-use>
```

# Custom Taglibs


## Tag Renderer

Every tag should be mapped to a "renderer". A renderer is just a function that takes two arguments (`input` and `context`). The `input` argument is an arbitrary object that contains the input data for the renderer. The `context` argument is an [asynchronous rendering context](https://github.com/raptorjs3/raptor-render-context) that wraps an output stream. Output can be produced using `context.write(someString)` There is no class hierarchy or tie-ins to Raptor Templates when implementing a tag renderer. A simple tag renderer is shown below:

```javascript
module.exports = function(input, context) {
    context.write('Hello ' + input.name + '!');
}
```

If, and only if, a tag has nested content, then a special `invokeBody` method will be added to the `input` object. If a renderer wants to render the nested body content then it must call the `invokeBody` method. For example:

```javascript
module.exports = function(input, context) {
    context.write('BEFORE BODY');
    if (input.invokeBody) {
        input.invokeBody();
    }
    context.write('AFTER BODY');
}
```

A tag renderer should be mapped to a custom tag by creating a `raptor-taglib.json` as shown in the next few sections.

## raptor-taglib.json

### Sample Taglib

```json
{
    "tags": {
        "my-hello": {
            "renderer": "./hello-renderer",
            "attributes": {
                "name": "string"
            }
        }
    }
}
```

### Defining Tags

Tags can be defined by adding a `"tags"` property to your `raptor-taglib.json`:

```json
{
    "tags": {
        "my-hello": {
            "renderer": "./hello-renderer",
            "attributes": {
                "name": "string"
            }
        },
        "my-foo": {
            "renderer": "./foo-renderer",
            "attributes": {
                "*": "string"
            }
        }
    }
}
```

Every tag should be associated with a renderer. When a custom tag is used in a template, the renderer will be invoked at render time to produce the HTML/output.

#### Defining Attributes

If you provide attributes then the Raptor Templates compiler will do validation to make sure only the supported attributes are provided. A wildcard attribute (`"*"`) allows any attribute to be passed in. Below are sample attribute definitions:

_Multiple attributes:_
```javascript
"attributes": {
    "message": "string",     // String
    "my-data": "expression", // JavaScript expression
    "*": "string"            // Everything else will be added to a special "*" property
}
```

### Scanning for Tags

Raptor Templates supports a directory scanner to make it easier to maintain a taglib by introducing a few conventions:

* One tag per directory
* All tag directories should be direct children of a parent directory
* Every tag directory must contain a `renderer.js` that is used as the tag renderer
* Each tag directory may contain a `raptor-tag.json` file or the tag definition can be embedded into `renderer.js`

With this approach, `raptor-taglib.json` will be much simpler:

```json
{
    "tags-dir": "./components"
}
```

Given the following directory structure:

* __components/__
    * __my-hello/__
        * renderer.js
    * __my-foo/__
        * renderer.js
    * __my-bar/__
        * renderer.js
        * raptor-tag.json
* raptor-taglib.json

The following three tags will be exported:

* `<my-hello>`
* `<my-foo>`
* `<my-bar>`

Directory scanning only supports one tag per directory and it will only look at directories one level deep. The tag definition can be embedded into the `renderer.js` file or it can be put into a separate `raptor-tag.json`. For example:

_In `renderer.js`:_

```javascript
exports.tag = {
    "attributes": {
        "name": "string"
    }
}
```

_In `raptor-tag.json`:_

```javascript
{
    "attributes": {
        "name": "string"
    }
}
```

_NOTE: It is not necessary to declare the `renderer` since the scanner will automatically use `renderer.js` as the renderer._

### Nested Tags

It is often necessary for tags to have a parent/child or ancestor/descendent relationship. For example:

```html
<ui-tabs>
    <ui-tab label="Overview"></ui-tab>
    <ui-tab label="Language Guide"></ui-tab>
    <ui-tab label="JavaScript API"></ui-tab>
</ui-tabs>
```

Raptor Templates supports this by leveraging JavaScript closures in the compiled output. A tag can introduce scoped variables that are available to nested tags. This is shown in the sample `raptor-taglib.json` below:

```json
{
    "tags": {
        "ui-tabs": {
            "renderer": "./tabs-tag",
            "var": "tabs"
        },
        "ui-tab": {
            "renderer": "./tab-tag",
            "import-var": {
                "tabs": "tabs"
            },
            "attributes": {
                "title": "string"
            }
        }
    }
}
```

In the above example, the `<ui-tabs>` tag will introduce a scoped variable named `tabs` that is then automatically imported by the nested `<ui-tab>` tags. When the nested `<ui-tab>` tags render they can use the scoped variable to communicate with the renderer for the `<ui-tabs>` tag.

The complete code for this example is shown below:

_components/tabs/renderer.js:_

```javascript
var templatePath = require.resolve('./template.rhtml');
var template = require('raptor-templates').load(templatePath);

module.exports = function render(input, context) {
    var nestedTabs = [];  
    
    // Invoke the body function to discover nested <ui-tab> tags
    input.invokeBody({ // Invoke the body with the scoped "tabs" variable
        addTab: function(tab) {
            tab.id = tab.id || ("tab" + tabs.length);
            nestedTabs.push(tab);
        }
    });
    
    // Now render the markup for the tabs:
    template.render({
        tabs: nestedTabs
    }, context);
};
```

_components/tab/renderer.js:_

```javascript
module.exports = function render(input, context) {
    // Register with parent but don't render anything
    input.tabs.addTab(input); 
};
```

_components/tabs/template.rhtml:_

```html
<div class="tabs">
    <ul class="nav nav-tabs">
        <li class="tab" c-for="tab in data.tabs">
            <a href="#${tab.id}" data-toggle="tab">
                ${tab.title}
            </a>
        </li>
    </ul>
    <div class="tab-content">
        <div id="${tab.id}" class="tab-pane" c-for="tab in data.tabs">
            <c-invoke function="tab.invokeBody()"/>
        </div>
    </div>
</div>
```


## Taglib Discovery

Given a template file, the `raptor-templates` module will automatically discover all taglibs by searching relative to the template file. The taglib discoverer will search up and also look into `node_modules` to discover applicable taglibs. 

As an example, given a template at path `/my-project/src/pages/login/template.rhtml`, the search path will be the following:

1. `/my-project/src/pages/login/raptor-taglib.json`
2. `/my-project/src/pages/login/node_modules/*/raptor-taglib.json`
3. `/my-project/src/pages/raptor-taglib.json`
4. `/my-project/src/pages/node_modules/*/raptor-taglib.json`
5. `/my-project/src/raptor-taglib.json`
6. `/my-project/src/node_modules/*/raptor-taglib.json`
7. `/my-project/raptor-taglib.json`
8. `/my-project/node_modules/*/raptor-taglib.json`

# FAQ

__Question:__ _Is Raptor Templates ready for production use?_

__Answer__: Yes, Raptor Templates has been battle-tested at [eBay](http://www.ebay.com/) and other companies for well over a year and has been designed with high performance, scalability, security and stability in mind. However, the latest version of Raptor Templates is still being marked as beta as we nail down the final feature set as part of the [RaptorJS 3 initiative](https://github.com/raptorjs/raptorjs/wiki/RaptorJS-3-Plan).

<hr>

__Question:__ _Can templates be compiled on the client?_

__Answer__: Possibly, but it is not recommended and it will likely not work in older browsers. The compiler is optimized to produce small, high performance compiled templates, but the compiler itself is not small and it comes bundled with some heavyweight modules such as a [JavaScript HTML parser](https://github.com/fb55/htmlparser2). In short, always compile your templates on the server. The [RaptorJS Optimizer](https://github.com/raptorjs3/raptor-optimizer) is recommended for including compiled templates as part of a web page.

<hr>

__Question:__ _Which web browsers are supported?_
__Answer__: The runtime for template rendering is supported in all web browsers. If you find an issue please report a bug.

<hr>

__Question:__ _How can Raptor Templates be used with Express?_

__Answer__: The recommended way to use Raptor Templates with Express is to bypass the Express view engine and instead directly pipe the rendering output stream to the response stream as shown in the following code:

```javascript
var raptorTemplates = require('raptor-templates');
var templatePath = require.resolve('./template.rhtml');

app.get('/profile', function(req, res) {
    raptorTemplates
        .stream(templatePath, {
            name: 'Frank'
        })
        .pipe(res); 
});
```

With this approach, you can benefit from streaming and there is no middleman (less complexity).

Alternatively, you can use the independent [view-engine](https://github.com/patrick-steele-idem/view-engine) module to provide an abstraction that allows pluggable view engines and provides full support for streaming. This is shown in the following sample code:

```javascript
var template = require('view-engine').load(require.resolve('./template.rhtml'));

app.get('/profile', function(req, res) {
    template.stream({
            name: 'Frank'
        })
        .pipe(res);
});
```

<hr>

__Question:__ _I heard Raptor Templates is XML-based. What is that about?_

__Answer__: Raptor Templates started out using an XML parser. This required that templates be well-formed XML (a major source of problems). This is no longer the case, as the compiler has been updated to use the awesome [htmlparser2](https://github.com/fb55/htmlparser2) module by [Felix Boehm](https://github.com/fb55). Also, XML namespaces are no longer used and all taglibs are now defined using simple JSON. If you are coming from the old XML-based version of Raptor Templates, please see the [Migration Guide](migration.md).

<hr>

__Question:__ _What is the recommended directory structure for templates and "partials"_

__Answer__: Your templates should be organized just like all other JavaScript modules. You should put your templates right next to the code that refers to them. That is, do not create a separate "templates" directory. For a sample Express app that uses Raptor Templates, please see [raptorjs-express-app](https://github.com/raptorjs3/samples/tree/master/raptorjs-express-app).

<hr>

__Question:__ _How is Raptor Templates related to [RaptorJS](http://raptorjs.org)?_

__Answer__: Raptor Templates is one of the modules that is part of the RaptorJS toolkit. It used to be a submodule, but now it has been split out into its own top-level Node.js module (for history, please see the [RaptorJS 3 Plan](https://github.com/raptorjs/raptorjs/wiki/RaptorJS-3-Plan) page).

# Discuss

Please post questions or comments on the [RaptorJS Google Groups Discussion Forum](http://groups.google.com/group/raptorjs).

# Contributors

* [Patrick Steele-Idem](https://github.com/patrick-steele-idem) (Twitter: [@psteeleidem](http://twitter.com/psteeleidem))
* [Phillip Gates-Idem](https://github.com/philidem/) (Twitter: [@philidem](https://twitter.com/philidem))

# Contribute

Pull Requests welcome. Please submit Github issues for any feature enhancements, bugs or documentation problems.

# License

Apache License v2.0
