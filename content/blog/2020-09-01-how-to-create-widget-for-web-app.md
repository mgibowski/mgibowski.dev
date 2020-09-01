+++
title = "How to create widget for your web app"
[taxonomies]
tags = [ "JavaScript" ]
+++

Let's say you have a web application and you would like to enable other developers to include portions of your app as interactive widgets on their websites.

An example of such a use case could be a Twitter timeline. Twitter provides a widget for including timelines on any website. To generate necessary code you would use the [Publish](https://publish.twitter.com) tool. Output from this tool looks like this:

```HTML
<a class="twitter-timeline" href="https://twitter.com/mgibowski?ref_src=twsrc%5Etfw">Tweets by mgibowski</a>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
```

So we have a snippet of code consisting of only two `HTML` tags: `a` and `script`. This short snippet is easy to paste into any website and it generates a full-featured widget that fetches data from remote source in real time. Let's see how widgets like this work under the hood.

## Overall architecture

If your widget is very simple and only puts static content into the target website - there is not much to it, just injecting some HTML code. In fact, this is how Twitter's quote widget works.

However, in more complex scenarios (like Twitter timeline) - with interactivity, rendering data from remote APIs or including custom styling, it's best to prepare your widget as a normal web page, serve it directly from your server and include in the target website inside of an `iframe`. To keep it easy for inclusion, one needs to create a short script that replaces a placeholder `HTML` tag for your widget with an `iframe`.

## Embedded content

The widget itself will be just a fragment of your website, so you need to prepare a view for it that displays exactly what you want. It is common and practical to set a fixed height of the widget and make it reactive in terms of width. In this way it will present itself well on all supported devices and `iframe` will not display any sliders.

## Embedding script

JavaScript embedding script is where most of the mechanism described in this post happens.

Let's invent an imaginary widget code:

```HTML
<div class="my-widget" data-arg="ArgValue"></div>
<script async src="https://example.com/widgets.js" charset="utf-8"></script>
```

In the above example the `div` element serves as a placeholder for the widget and `widgets.js` script is responsible for replacing this placeholder with a real widget - fragment of your website rendered inside of an `iframe`. As you may imagine, the `async` parameter of the `script` tag prevents parser-blocking - so the browser will load, parse and evaluate this script in parallel without interrupting parsing the target document.

The algorithm of the `widgets.js` script is:

1. Defer execution until when `DOM` is ready
2. Query `DOM` for all widget placeholders
3. Create `iframe` and replace each placeholder
4. Optionally pass arguments from placeholders to `iframes`

## Embedding script source code

For the sake of this blog post I created a [repository](https://github.com/mgibowski/widget-embedder-example) with an example Embedder. Repository is a simple Webpack[^1] project configured to fit our needs, so there is no code-splitting and the output JS file is called `widgets.js`.

Most of logic resides in `Embedder.js`. The main JavaScript file `index.js` imports that file and executes it's logic once the HTML document is fully loaded:

```Javascript
import Embedder from './Embedder';

document.addEventListener("DOMContentLoaded", () => {
  const embedder = new Embedder();
  embedder.createWidgets();
});
```

`Embedder.createWidgets()` method looks for all widget placeholders on the target website and replaces them with `iframes` (you could have multiple widgets on a single page):

```Javascript
  createWidgets() {
    document.querySelectorAll('.my-widget').forEach((element) => {
      this._createWidget({
        element,
        height: '320px',
        customArg: element.getAttribute('data-arg'),
      });
    });
  }
```

`Embedder._createWidget` method includes the `_` prefix to make it _"private by convention"_. Its body is very small in order to separate concerns.

Let's take a look at it:

```Javascript
  _createWidget(options) {
    const url = process.env.NODE_ENV == 'development'
      ? `http://localhost:8080/widget?customArg=${options.customArg}`
      : `https://example.com/widget?customArg=${options.customArg}`;
    const iframe = this._createIframe(url, options);
    options.element.parentNode.replaceChild(iframe, options.element);
  }
```

Firstly, it generates the URL of the website fragment serving as a widget. You may notice the different URL for `development` environment. I assume you would be developing the web app serving your widget locally and you might want it to be served from localhost when running Embedder in `development` mode.

Then, there is a method invocation for creating an `iframe` and a separate call to replace the placeholder with this `iframe`.

Finally, `Embedder._createIframe()` looks like this:

```Javascript
  _createIframe(url, options) {
    const iframe = document.createElement('iframe');
    iframe.setAttribute('name', 'My Embedded widget');
    iframe.setAttribute('title', 'My Embedded widget');
    iframe.setAttribute('src', url);
    iframe.setAttribute('border', 'none');
    iframe.setAttribute('loading', 'lazy');
    iframe.setAttribute('width', '100%');
    iframe.setAttribute('frameborder', '0');
    iframe.setAttribute('height', options.height);

    return iframe;
  }
```

So that's it. That's basically how interactive widgets work.
There is, however, one more thing worth explaining.

## Parametrizing embedded content with arguments

Our imaginary example widget code contained a custom argument `data-arg="ArgValue"`. It is possible you would need something like this too. Maybe you have different widgets and one `widgets.js` script for injecting all of them based on some html parameter. Or maybe your widgets are specific for each user and you need to somehow pass the username to the widget.

There are two solutions for this depending on the use case.

The first one, contained in the above source code, passes a custom argument as a URL parameter to the website serving the widget.

Another option would be to utilize [`Window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) mechanism and communicate to your widget web page directly via JavaScript. This may be necessary when you need to pass some sensitive information that shouldn't be part of an URL.

In such case previous `Embedder._createWidget()` would look like this:

```Javascript
  _createWidget(options) {
    const url = process.env.NODE_ENV == 'development'
      ? 'http://localhost:8080/widget'
      : 'https://example.com/widget';
    const iframe = this._createIframe(url, options);
     // 👇 Here goes the main change
    iframe.onload = () => {
      const args = { customArg: options.customArg };
      iframe.contentWindow.postMessage({name: 'init', args}, url);
    }
    options.element.parentNode.replaceChild(iframe, options.element);
  }
```

Handling such a message in the widget code is left as an exercise for the reader 😉.

Thanks for visiting,
Michał.

_If you spot any mistakes in this post, please let me know via [GitHub](https://github.com/mgibowski/mgibowski.dev/pull/1/files)._

[^1]: As of 2020 [Webpack](https://webpack.js.org/) is the de-facto standard for bundling JavaScript projects.
