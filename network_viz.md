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

This is my personal network. 

<div id='d3div'></div>

You can click-and-drag the nodes around.

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

<script src="//d3js.org/d3.v3.min.js"></script>
<script>

var width = $("#d3div").width(),
    height = 500;

var color = d3.scale.category20();

var force = d3.layout.force()
    .charge(-120)
    .linkDistance(30)
    .size([width, height]);

var svg = d3.select("#d3div").append("svg")
    .attr("width", width)
    .attr("height", height);

d3.json("./curtis.json", function(error, graph) {
  if (error) throw error;

  force
      .nodes(graph.nodes)
      .links(graph.links)
      .start();

  var link = svg.selectAll(".link")
      .data(graph.links)
    .enter().append("line")
      .attr("class", "link")
      .style("stroke-width", function(d) { return Math.sqrt(d.value); });

  var node = svg.selectAll(".node")
      .data(graph.nodes)
    .enter().append("circle")
      .attr("class", "node")
      .attr("r", 5)
      .style("fill", function(d) { return color(d.group); })
      .call(force.drag);

  node.append("title")
      .text(function(d) { return d.name; });

  force.on("tick", function() {
    link.attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; });

    node.attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });
  });
});

</script>

## Conclusion

This went off without a hitch. If I understood more of how all of this works I suppose there wouldn't have been any doubt, but it is nonetheless nice to know that you can embed D3 visualizations in Jekyll blog posts. 
