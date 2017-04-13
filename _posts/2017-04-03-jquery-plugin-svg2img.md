---
title: jQuery Plugin svg2img
subtitle: Convert inline SVGs to standalone image files without loosing styles
tags: [project, jquery, plugin, svg]
lang: en
---

SVG based charts and visualizations are often used these days. But what if you want to use your browser inline SVGs, generated with some sort of JavaScript and fancy styled with CSS, offline - maybe in a presentation, send by email or printed out large scaled?

You can create a screenshot, of course. But does this look nice when it comes to scaling? You can try to copy the HTML code of the SVG and wrap it into a correct SVG container, but then you will loose in the most cases the stylings, as normally not all CSS definitions are directly attached to the SVG elements. You can also use a browser extension like [this one for Google Chrome][1], but then you depend on the browser and the extension.

There is now a declarative way with the help of a jQuery plugin, which is 100% client based:

```js
jQuery('#example-graph').svg2img();
```

There are default options - you can redeclare it:

```js
jQuery.fn.svg2img.defaults = {
    format: "svg", // svg, png, gif, jpeg or bmp - multiple formats possible: "svg,png"
    jpegQuality: 1, // between 0 and 1 (0% and 100% JPEG quality)
    debug: false // write debug information to console
};
```

You can also set the options at runtime - try it out in your browser console directly here on the page:

```js
$('#example-graph').svg2img({format:"svg,png", debug:true});
```

Alternative you can use the links under the network chart.

The plugin checks if your selector is a SVG element. If this is not the case, it searches under each selector element for all SVGs and exports it. The name(s) of the file(s) are automatically derived from the element ID (or parent element ID) or set to `export`. The current date and time is appended to the file name. For our example graph on this page the name will be something like `example-graph-20170403-154653.svg`.

You can download all SVGs from one page by setting the selector to the document, body or svg - as you like it. There is no need for a `each()` call, `svg2img` works on the whole selection and is chainable:

```js
$("svg").svg2img().css("border","1px solid red");
```

If you want to use anchors to provide export links then it is recommended to prevent the default action like so:

```html
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img();">SVG</a>
```

Otherwise Firefox and IE failing to save. Like often, Chrome works without any extra attention and for the IE I had to implement some "special features". One last thing: Safari has currently (as of this writing) problems with the underlaying `savAs()` implementation and tries to open the images in a new tab with or without success. See also this [issue on GitHub][2].

The project is hosted on [GitHub][3] and MIT licensed.

Happy SVG exporting  
Ottmar

[1]: https://chrome.google.com/webstore/detail/export-svg-with-style/dkjdcaddoplepioppogpckelchefhddi
[2]: https://github.com/eligrey/FileSaver.js/issues/267
[3]: https://github.com/ogobrecht/jquery-plugin-svg2img

<div id="example-graph"></div><!--the graph container-->
Download
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img();">SVG</a> |
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img({format:'png'});">PNG</a> |
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img({format:'gif'});">GIF</a> |
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img({format:'jpeg'});">JPEG</a> |
<a href="" onclick="event.preventDefault(); $('#example-graph').svg2img({format:'bmp'});">BMP</a>

<link  href="/assets/d3.js/d3-force-2.0.2.css" rel="stylesheet" type="text/css">
<script src="/assets/d3.js/d3-3.5.6.min.js"></script>
<script src="/assets/d3.js/d3-force-2.0.2.min.js"></script>
<script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
<script src="/assets/svg2img/svg2img.min.js"></script>
<script>
window.onload = function (){
  window.example = netGobrechtsD3Force('example-graph')
    .debug(true) //to enable the customization wizard
    //.lassoMode(true)
    .zoomMode(true)
    .useDomParentWidth(true) //for responsive layout
    .start(); //sample data is provided, when called without data
}
</script>
