---
layout: post
title: "Network Visualization"
author: "Curtis Hampton"
date: "November 4th, 2017"
categories: blog
---
<script src="//code.jquery.com/jquery.js"></script>
<style>

.node {
  stroke: #fff;
  stroke-width: 1.5px;
}

.link {
  stroke: #999;
  stroke-opacity: .6;
}

</style>

This is my own personal network. 

<div id='d3div'></div>

This is a Force-Directed layout, and you can click-and-drag the nodes around.

## The Code

The visualization above was taken from [this](http://bl.ocks.org/mbostock/4062045) entry of the [D3 gallery](https://github.com/mbostock/d3/wiki/Gallery). First, I create a `style` tag for the node and link attributes of the plot.

{% highlight html %}
<style>

.node {
  stroke: #fff;
  stroke-width: 1.5px;
}

.link {
  stroke: #999;
  stroke-opacity: .6;
}

</style>
{% endhighlight %}

Then I create a `div` tag to place the D3-created SVG. You can assign it any id you want, here I chose "d3div".

{% highlight html %}
<div id='d3div'></div>
{% endhighlight %}

Finally the actual Javascript to create the SVG. I wanted the SVG to resize based on the width of the parent container. This way it should work properly when viewed on mobile or on a skinny web broswer window. To do this I changed the width variable to grab the width of the d3div tag. When you resize the window you'll have to refresh your browser to get the SVG to change. There almost certainly exists a more clever approach, but I didn't take the time to work it out.

{% highlight html %}

<script src="https://d3js.org/d3.v4.min.js"></script>
<script>

var svg = d3.select("svg"),
    width = +svg.attr("width"),
    height = +svg.attr("height");

var color = d3.scaleOrdinal(d3.schemeCategory20);

var simulation = d3.forceSimulation()
    .force("link", d3.forceLink().id(function(d) { return d.id; }))
    .force("charge", d3.forceManyBody().strength(-1500))
    .force("center", d3.forceCenter(width / 2, height / 2));


d3.json("curtis.json", function(error, graph) {
  if (error) throw error;

  var link = svg.append("g")
      .attr("class", "links")
    .selectAll("line")
    .data(graph.links)
    .enter().append("line")
      .attr("stroke-width", function(d) { return Math.sqrt(d.value); });

  var node = svg.append("g")
      .attr("class", "nodes")
    .selectAll("circle")
    .data(graph.nodes)
    .enter().append("circle")
      .attr("r", 25)
      .attr("fill", function(d) { return color(d.group); })
      .call(d3.drag()
          .on("start", dragstarted)
          .on("drag", dragged)
          .on("end", dragended));

  node.append("title")
      .text(function(d) { return d.id; });

  simulation
      .nodes(graph.nodes)
      .on("tick", ticked);

  simulation.force("link")
      .links(graph.links);

  function ticked() {
    link
        .attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; });

    node
        .attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });
  }
});

function dragstarted(d) {
  if (!d3.event.active) simulation.alphaTarget(0.3).restart();
  d.fx = d.x;
  d.fy = d.y;
}

function dragged(d) {
  d.fx = d3.event.x;
  d.fy = d3.event.y;
}

function dragended(d) {
  if (!d3.event.active) simulation.alphaTarget(0);
  d.fx = null;
  d.fy = null;
}

</script>
