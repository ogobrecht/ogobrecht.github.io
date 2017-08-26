---
title: D3.js Force-Directed Network Chart
subtitle: A JavaScript standalone implementation and Oracle APEX region type plugin
tags: [project, oracle, apex, plugin, d3js, chart, svg]
lang: en
last_modified: 2017-08-12
---
_My first Oracle APEX plugin and open source project :-)_

<div id="example-graph"></div><!--the graph container-->

Everything started some months ago when I stumbled over this fascinating [force-directed graph][1]. I immediately had the idea to use this technology to visualize data models in APEX. My experience is, that direct after I documented a data model, this documentation is out of date. I wanted to have a live visualization of the data models. After the first implementation it turned out, that the resulting graph is also useful for other purposes and I decided to build up an APEX plugin for easier integration.

If you use the bare JavaScript files you can run it on every HTML page:

```html
<div id="example"></div><!--the graph container-->
<link  href="/assets/d3.js/d3-force-2.1.0beta1.css" rel="stylesheet" type="text/css">
<script src="/assets/d3.js/d3-3.5.6.min.js"></script>
<script src="/assets/d3.js/d3-force-2.1.0beta1.min.js"></script>
<script>
window.onload = function (){
  window.example = netGobrechtsD3Force('example')
    .debug(true) //to enable the customization wizard
    .lassoMode(true)
    //.zoomMode(true)
    .useDomParentWidth(true) //for responsive layout
    .wrapLabels(true)
    .preventLabelOverlappingOnForceEnd(true)
    .labelPlacementIterations(1000)
    .start(); //sample data is provided, when called without data
}
</script>
```

I found myself often fiddling around with the parameters of the physical simulation - change config, reload page, not amused, change config...

Really annoying - even I know the implementation behind. So, the question was: What can I do for an easy configuration. My answer was after a while of thinking and trying out: A customization wizard with a live preview of the changes. The link to enter the wizard is shown when the debug mode is switched on or in APEX when the developer bar is shown - try it out in the example graph above.

You can find the sources and more informations on [GitHub][2] and a demo app on [apex.oracle.com][3].


[1]: https://bl.ocks.org/mbostock/4062045
[2]: https://github.com/ogobrecht/d3-force-apex-plugin
[3]: https://apex.oracle.com/pls/apex/f?p=18290:1

<link  href="/assets/d3.js/d3-force-2.1.0beta1.css" rel="stylesheet" type="text/css">
<script src="/assets/d3.js/d3-3.5.6.min.js"></script>
<script src="/assets/d3.js/d3-force-2.1.0beta1.min.js"></script>
<script>
window.onload = function (){
  window.example = netGobrechtsD3Force('example-graph')
    .debug(true) //to enable the customization wizard
    .lassoMode(true)
    //.zoomMode(true)
    .useDomParentWidth(true) //for responsive layout
    .wrapLabels(true)
    .preventLabelOverlappingOnForceEnd(true)
    .labelPlacementIterations(1000)
    .start(); //sample data is provided, when called without data
}
</script>
