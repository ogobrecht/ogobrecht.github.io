---
title: D3.js - Data Driven Documents
subtitle: Folien für DOAG APEX connect 2015, Düsseldorf
lang: de
layout: slides
data-separator-notes: "^Anmerkung:"
---
<link  href="/assets/d3.js/d3-force-2.0.2.css" rel="stylesheet" type="text/css">
<script src="/assets/d3.js/d3-3.5.6.min.js"></script>
<script src="/assets/d3.js/d3-force-2.0.2.min.js"></script>
<script>
Reveal.addEventListener( 'ready', function( event ) {

  //Beispiel 3
  window.data = [
    {"id":1, "r":14, "x":80,  "y":15, "color":"red"},
    {"id":2, "r":14, "x":100, "y":15, "color":"green"},
    {"id":3, "r":14, "x":120, "y":15, "color":"blue"}
  ];
  window.svg = d3.select('svg#Beispiel_3');
  window.circle = window.svg.selectAll('circle')
    .data(data, function(d){ return d.id; });
  window.circle.exit().remove();
  window.circle.enter().append('circle');
  window.circle
    .attr('r', function(d) { return d.r; })
    .attr('cx', function(d) { return d.x; })
    .attr('cy', function(d) { return d.y; })
    .attr('fill', function(d) { return d.color; });

  //Beispiel 4
  window.nodes = [
    {"name":"King","dept":10},{"name":"Blake","dept":30},
    {"name":"Clark","dept":10},{"name":"Jones","dept":20},
    {"name":"Scott","dept":20},{"name":"Ford","dept":20},
    {"name":"Smith","dept":20},{"name":"Allen","dept":30},
    {"name":"Ward","dept":30},{"name":"Martin","dept":30},
    {"name":"Turner","dept":30},{"name":"Adams","dept":20},
    {"name":"James","dept":30},{"name":"Miller","dept":10}
  ];
  window.links = [
    {"source":1,"target":0},{"source":2,"target":0},
    {"source":3,"target":0},{"source":7,"target":1},
    {"source":8,"target":1},{"source":9,"target":1},
    {"source":10,"target":1},{"source":12,"target":1},
    {"source":13,"target":2},{"source":4,"target":3},
    {"source":5,"target":3},{"source":6,"target":5},
    {"source":11,"target":4}
  ];
  window.width = 600, height = 400;
  window.svg_4 = d3.select("svg#Beispiel_4")
    .attr("width",width).attr("height", height);
  window.color = d3.scale.category10();
  window.force = d3.layout.force().size([width,height]);
  window.link = svg_4.selectAll("line").data(window.links)
    .enter().append("line").style('stroke','#999');
  window.node = svg_4.selectAll("circle").data(window.nodes)
    .enter().append("circle")
      .attr("r", 5)
      .style("fill", function(d){ return color(d.dept); })
      .call(force.drag);
  window.force.on("tick", function(){
    window.link.attr("x1", function(d){ return d.source.x; })
        .attr("y1", function(d){ return d.source.y; })
        .attr("x2", function(d){ return d.target.x; })
        .attr("y2", function(d){ return d.target.y; });
    window.node.attr("cx", function(d){ return d.x; })
        .attr("cy", function(d){ return d.y; });
  });
  window.force.nodes(window.nodes).links(window.links).start();

  //Beispiel 5
  window.demo_emp = netGobrechtsD3Force('demo_emp')
    .width(800)
    .height(500)
    .colorScheme('color10')
    .showTooltips(false)
    .start();

  //Beispiel Anhang
  window.my_chart = function () {
    var conf = { "width": 600, "height": 400 };
    function chart(){/*create chart with conf*/}
    chart.render = function(){
      chart();
      return chart;
    };
    chart.width = function(value) {
      if (!arguments.length) return conf.width;
      conf.width = value;
      return chart;
    };
    return chart;
  }

});
</script>