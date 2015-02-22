---
layout: post
title: "Out of the shadows"
date: 2015-02-22 04:18
comments: true
categories: [JavaScript, CSS] 
---

![Shadow puppet](/images/shadow/shadow-puppet.jpg)

**Disclaimer** - This article discusses topics which are not cemented yet, may change and likely won't work without browser shims / hacks.

So we have all likely heard of web components by now; they are usable in most JavaScript libraries and are the UI kit that is expected of any GUI related software.

However one of the least discussed features of web components is the [Shadow DOM](http://www.w3.org/TR/shadow-dom/) and even less discussed is how it can be styled with scoped styles.

Shadow DOM is the way in which developers will be able to hide away the minor details of website implementation, this means that template authors won't need to worry as much about perfecting their markup and concentrate on the building block they are trying to make. A great example of this is select boxes native in browsers, content authors don't need to worry about all the complexities of the buttons and functionality. The author can then just give some basic styling to the select box.

So I hear you cry, why is this different to normal components that give us the ability to define our own web language and reuse. Well I think it solves the issue of specificity - by defining the boundaries of reusable objects, you gain the ability to isolate styles when necessary.

#### Before DOM render

![Before render](/images/shadow/before-render.png)

Elements are nested under the Shadow host n1.

#### After DOM render

![After render](/images/shadow/after-render.png)

Once rendered, elements that are not a Shadow route are nested under the DOM insertion point `<content>`

```
<my-element><!-- Shadow host -->
  <"shadow-tree"><!-- A non element -->
    <div class="title">Some text</div>
    <!-- CSS inside a shadow DOM is treated as scoped -->
    <style>
      //my-element won't match anything here, use :host instead
      my-element {
        color: red;
      }
      //:host selector here will match my-element
      :host {
        color: blue;
      }
    </style>
  </>
</my-element>

<style>
  //Matches .title element
  my-element::shadow div {
    color: red;
  }
  //Applies to the content div elements
  //not ones residing in the shadow DOM .title would not match
  my-element div {
    color: red;
  }  
</style>
```

### So why does it matter to me?

- Shadow DOM provides a encapsulation layer that means multiple teams can work on the same HTML documents without worries of colliding.
- Flexibility still remains to "select through the shadows" so you can override the default appearance of a component.
- Content authors can concentrate on styling usage rather than implementation.
- Component developers can concentrate on building the most reusable components possible.
- No amount of namespacing gives the isolation that Shadow DOM gives.
  - Nested HTML that is namespaced still has specificity issues, where as isolation gives easier container selection.

Chrome debugger even helps show the DOM tree:

![Debugger output](/images/shadow/debugger.png)

### Shadow DOM concepts

- [**Shadow host**](http://w3c.github.io/webcomponents/spec/shadow/#dfn-shadow-host) - is the container of one or many **Shadow trees**
  - Elements within the **Shadow host** are detached until they match a DOM insertion point
  - The last **Shadow tree** inserted into the **Shadow host** is the only attached Node to the host.
    - Older **Shadow trees** can be inserted into the newest shadow tree with a **Shadow insertion point**
- [**Shadow insertion point**](http://w3c.github.io/webcomponents/spec/shadow/#dfn-insertion-point) When a host has multiple **Shadow trees** the trees can be reinserted into the newest tree if it has an insertion point.
  - `<shadow>` is the current only implementation of a **Shadow insertion point**
- [**DOM insertion point**](http://w3c.github.io/webcomponents/spec/shadow/#insertion-points) - At the moment this is a `<content>` tag within the shadow DOM however other implementations may happen
  - An insertion point has the elements placed into it inside the *Shadow tree*
  - Insertion points has no representation in the DOM, it is just a container for elements.
  - Insertion points match the elements in the *Shadow host*
- [**Shadow trees**](http://w3c.github.io/webcomponents/spec/shadow/#shadow-trees) - A tree of elements which are abstracted away from the normal DOM tree
  - **Shadow trees** detach all elements from the **Shadow host**

### A simpler implementation

However we can vastly simplify this for the most common usage with custom elements;

- A custom element is given a **Shadow tree** (Making the custom element a **shadow host**)
- A template of DOM nodes is cloned and inserted into the **Shadow tree**
- Elements that are contained in the custom element are either detached from the DOM
  - Or if the template has a `<content>` tag the elements are inserted within that.

Assuming we have some code building the custom elements (see final example), the following code:
```
<template id="mycustomelement">
  <div>
    <content></content>
  </div>
</template>

<my-custom-element>
  <b>Hello</b>
</my-custom-element>
```

Will result in the following DOM tree:
```
<my-custom-element>
  <div>
    <b>Hello</b>
  </div>
</my-custom-element>
```

## Scoping CSS

Styles are assumed to be scoped to the **Shadow Root** however needs [clarification](http://discourse.specifiction.org/t/shadow-dom-selection/766) as the notes have been removed from the specification.

[CSS Scoping specification](http://drafts.csswg.org/css-scoping/) defines ways in which we can utilise CSS to get over those age old specificity woes. 

The spec brings along some new friends which will really be useful for working with shadow DOM:

### [::shadow](http://drafts.csswg.org/css-scoping/#shadow-pseudoelement)
`::shadow` can be used to override the components styles within the shadow DOM.

### [>>>](http://drafts.csswg.org/css-scoping/#deep-combinator)

`>>>` deep shadow selector, through multiple layers of shadow DOM, however doesn't select through `<content>`

### [::content](http://drafts.csswg.org/css-scoping/#content-combinator)
`::content` is used to select the elements residing in the DOM insertion point.

### [:host()](http://drafts.csswg.org/css-scoping/#selectordef-host-function)
Used from within a Shadow DOM to select the Shadow host that matches the selector within the brackets.
```
:host(.big) // From the styles within a shadow root this matches host elements that have an ancestor which match '.big'
```

### [:host](http://drafts.csswg.org/css-scoping/#selectordef-host)
`:host` is used for selecting the shadow host element from within the shadow DOM itself.

### [:host-context()](http://drafts.csswg.org/css-scoping/#selectordef-host-context)
`:host-context` is used to select the parents of the host element from within the shadow context.


### [:unresolved](http://w3c.github.io/webcomponents/spec/custom/#unresolved-element-pseudoclass) <small>(part of the web component specification)</small>

`:unresolved` can be used to target elements that have not yet been registered by the JavaScript. As all new web components are not native in the browser, until the elements are defined then the browsers considers web-component tags as 'unresolved' when the element is defined then the flag is removed from those elements. This means that the CSS author has full control of the behaviour of an element until the element is defined.
For example the author could pick to hide the elements or show a loading spinner.


```css
my-custom-component:unresolved {
  display: none;
}
```

## Example
A simple function to define new elements with a shadow DOM.
```javascript
/**
 * Create callback
 * @callback createElementCallback
 * @param {element} shadow the shadow dom element
 * @param {scope} this the scope of the custom element
 */

/**
 * Generates a new component
 * @param {string} elementName Name of JavaScript Class for the element
 * @param {string} tagName Name of the tag produced (Must contain at least one '-')
 * @param {string} templateSelector Selector to template for the contents of the shadow DOM
 * @param {createElementCallback} postCreateCallback Callback triggered on construction of the element
 */
function generateComponent(elementName, tagName, templateSelector, postCreateCallback) {

  //Create a custom element that extends from the base HTMLElement
  var elementPrototype = Object.create(HTMLElement.prototype);

  //On creation of an element this code triggers
  //Defines a shadow root within our custom element
  //Imports in the matching template
  //Triggers the callback for any further customisation
  elementPrototype.createdCallback = function() {
    var shadow = this.createShadowRoot();
    var template = document.querySelector(templateSelector);
    var clone = document.importNode(template.content, true);
    shadow.appendChild(clone);
    if (postCreateCallback) { postCreateCallback(shadow, this); }
  };

  //Register the element name to the window
  window[elementName] = document.registerElement(tagName, {prototype: elementPrototype});
}
```

Define some elements
```javascript
function slugify(slug) {
  return slug.toLowerCase().replace(/\s/g,'-');
}

generateComponent('MyBlockElement', 'my-block', '#myblocktemplate', function (shadow, scope) {
  var headingDOM = shadow.querySelector('my-heading');
  var heading = scope.getAttribute('heading');
  var icon = scope.getAttribute('heading-icon');
  if (headingDOM) {
    if (heading) {
      headingDOM.innerText = heading;
    } else {
      headingDOM.style.display = 'none';
    }
    if (icon) {
      headingDOM.setAttribute('icon', icon);
    }
  }
});

generateComponent('MyHeadingElement', 'my-heading', '#myheadingtemplate', function (shadow, scope) {
  var icon = scope.getAttribute('icon');
  var iconDOM = shadow.querySelector('i');
  var linkDOM = shadow.querySelector('a');
  if (icon && iconDOM) {
    iconDOM.classList.add(icon);
  }
  if (linkDOM && scope.innerText !== '') {
      linkDOM.setAttribute('href', '#' + slugify(scope.innerText));
  }
});
```

Defiine the component templates:
```
<template id="myheadingtemplate">
  <div>
    <i class="icon"></i>
    <h1>
      <content></content>
    </h1>
    <a href="#" >#</a>
  </div>
  <style>
      i.icon:before {font-size:30px;}
      i.user:before {content: '☻';}
      i.pentagon:before {content: '⬟';}
      h1 {display: inline-block;}
  </style>
</template>

<template id="myblocktemplate">
  <div>
    <my-heading></my-heading>
    <content></content>
  </div>
  <style>
     div {
       border: 1px solid #000;
       padding: 5%;
       margin-bottom: 5%;
     }
  </style>
</template>
```

Our actual template using the elements:
```
<my-block>
  <my-block heading-icon="user" heading="My important heading" >
    <div>Lorem ipsum</div>
  </my-block>
    
  <my-block heading-icon="pentagon" heading="My second heading" >
    Lorem ipsum
  </my-block>
</my-block>
```

The same effect could be achieved with the same CSS outside the templates:
```css
my-heading::shadow div {
 border: 1px solid #000;
 padding: 5%;
 margin-bottom: 5%;
}
my-block::shadow i.icon:before {font-size:30px;}
my-block::shadow i.user:before {content: '☻';}
my-block::shadow i.pentagon:before {content: '⬟';}
my-block::shadow h1 {display: inline-block;}
```
[See this on CodePen](http://codepen.io/anon/pen/dPeYGN)

Which is lengthier however there are a few solutions to this:

- Use LESS or Sass
- Use HTML imports to push the CSS into the element whilst still maintaining a separate CSS file
  - Drawback is [Mozilla won't be implementing import just yet](https://hacks.mozilla.org/2014/12/mozilla-and-web-components/)
- Use a shim to compile together the CSS, JS and HTML into a package similar to how other frameworks do for components now
- Shim [nested CSS](http://tabatkins.github.io/specs/css-nesting/#the-nested-block)

## Further reading
- [Pollyfills](https://github.com/webcomponents/webcomponentsjs)
- [Styling with polymer](https://www.polymer-project.org/articles/styling-elements.html)
- [Good practices with web components](https://github.com/webcomponents/webcomponents.github.io/blob/site/src/documents/articles/web-components-best-practices.html.md)
- [Polymer in Ember-cli](https://github.com/inigo-llc/ember-polymer)
  - [Derived from this gist](https://gist.github.com/dnegstad/eee660aa32efd906082c)
- [Shadow DOM CSS cheat sheet](http://robdodson.me/shadow-dom-css-cheat-sheet/) - also has examples using polymer

[Picture credit to Double-M](https://www.flickr.com/photos/double-m2/3945354161/)
