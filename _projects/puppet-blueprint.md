---
layout: page
title: Puppet Blueprints
summary: Dashboard showing architecture of your puppet managed servers and their dependencies
permalink: /projects/puppet-blueprints/
icons: 
  - code
  - sitemap
  - dashboard
---


Many larger organisations use configuration managers such as [Puppet](http://puppetlabs.com/) and [Chef](https://www.chef.io/chef/) to orchestrate their server environment from code, allowing for many tens, hundreds and even thousands of servers to be managed by only a handful of engineers.


# Scale Vs [Cognitive Load](http://en.wikipedia.org/wiki/Cognitive_load)

As increasing number of servers are provisioned by fewer numbers of staff, it becomes ever harder to understand how these servers actually fit into the bigger picture. While a given machine can be inspected for the services it runs, eg HAProxy, Tomcat, Postgresql and so forth, it is less apparent how those services are used to deliver value to the team, nor how they fit in the dependency tree of larger service oriented platforms.



## Monitoring

In a simple model if Machine A goes offline the typical questions to ask are who needs to know, and how quickly, what service breaks, and ultimatly how does this affect our customers? Quickly followed up by 'how do we fix it' and if you're lucky 'how do we stop it happening in the future'.

These questions are often answered by simply sending a message to whomever is 'on-call' and then relying on their knowledge of the platform to being able to recover from any arbitrary failure in the platform, usually at 4am from a laptop at home.




believe the questions you should be asking are; 'Where are the choke-points in our architecture?' in other words, which server(s) have the highest service impact if they were to fail. 


# Puppet Blueprints

My suggested approach to helping resolve these issues is to provide a dashboard view of the server environment that is always up-to-date, and represents not only the servers in your estate, but what services are operating on those servers, and how they interact with eachother.

## Lofty Goals

The questions I'd like to be able answer are 

1. What servers are used to deliver <Product X> ?
2. Where do we make use of <Software Y>?
3. What is the impact of restarting <Hardware Z>
4. How is <Switch A> involved in delivering content for <Product X>?

I'm specifically not going to try and answer questions that can be solved by existing tools like [Foreman](theforeman.org/learn_more.html) or [Puppet Explorer](https://github.com/spotify/puppetexplorer) which are capable of returning the state of machines, but not necessarily their relationships to other parts of your system.


## Initial concept

* Use [PuppetDB](https://docs.puppetlabs.com/puppetdb/latest/) to provide access to core Puppet state 
* Using [Exported Resources](https://docs.puppetlabs.com/puppet/latest/reference/lang_exported.html) also expand upon relationships between services
    * VM Hosts indicate the guests they're running
    * Guests announce the services they deliver
    * Services announce their dependencies on other systems and the product they're delivering
* Collate the data into a graph database (eg Neo4J / TitanDB ) maintaining relationships between services
* Create dashboard that takes the Node data and lets users explore the relationships
    * Intitial import into Neo4J may be sufficient as a starting point with some cypher queries


## Example

A quick concept diagram showing a badly formed relationship between a HA proxy load balancer to three web servers each of which hit a specifi DB server which are in a replicated cluster (circular).

<div id='example-graph'>
</div>

<style>

.link {
  fill: none;
  stroke: #666;
  stroke-width: 1.5px;
}

#datastore {
  fill: green;
}

#balance {
  stroke-width: 1.5px;
}

.link.datastore {
  stroke: green;
}

.link.replication {
  stroke-dasharray: 0,2 1;
}

circle {
  fill: #ccc;
  stroke: #333;
  stroke-width: 1.5px;
}

text {
  font: 10px sans-serif;
  pointer-events: none;
  text-shadow: 0 1px 0 #fff, 1px 0 0 #fff, 0 -1px 0 #fff, -1px 0 0 #fff;
}

</style>
<script src="http://d3js.org/d3.v3.min.js"></script>
<script>

// http://blog.thomsonreuters.com/index.php/mobile-patent-suits-graphic-of-the-day/
var links = [


  {source: "db1.cluster.dc1", target: "db2.cluster.dc1", type: "replication"},
  {source: "db2.cluster.dc1", target: "db3.cluster.dc1", type: "replication"},
  {source: "db3.cluster.dc1", target: "db1.cluster.dc1", type: "replication"},

  {source: "web1.blog.dc1", target: "db1.cluster.dc1", type: "datastore"},
	{source: "web2.blog.dc1", target: "db2.cluster.dc1", type: "datastore"},
	{source: "web3.blog.dc1", target: "db3.cluster.dc1", type: "datastore"},

	{source: "haproxy.blog.dc1", target: "web1.blog.dc1", type: "balance"},
	{source: "haproxy.blog.dc1", target: "web2.blog.dc1", type: "balance"},
	{source: "haproxy.blog.dc1", target: "web3.blog.dc1", type: "balance"},



];

var nodes = {};

// Compute the distinct nodes from the links.
links.forEach(function(link) {
  link.source = nodes[link.source] || (nodes[link.source] = {name: link.source});
  link.target = nodes[link.target] || (nodes[link.target] = {name: link.target});
});

var width = 300,
    height = 300;

var force = d3.layout.force()
    .nodes(d3.values(nodes))
    .links(links)
    .size([width, height])
    .linkDistance(60)
    .charge(-300)
    .on("tick", tick)
    .start();

var svg = d3.select("#example-graph").append("svg")
    .attr("width", width)
    .attr("height", height);

// Per-type markers, as they don't inherit styles.
svg.append("defs").selectAll("marker")
    .data(["suit", "licensing", "resolved"])
  .enter().append("marker")
    .attr("id", function(d) { return d; })
    .attr("viewBox", "0 -5 10 10")
    .attr("refX", 15)
    .attr("refY", -1.5)
    .attr("markerWidth", 6)
    .attr("markerHeight", 6)
    .attr("orient", "auto")
  .append("path")
    .attr("d", "M0,-5L10,0L0,5");

var path = svg.append("g").selectAll("path")
    .data(force.links())
  .enter().append("path")
    .attr("class", function(d) { return "link " + d.type; })
    .attr("marker-end", function(d) { return "url(#" + d.type + ")"; });

var circle = svg.append("g").selectAll("circle")
    .data(force.nodes())
  .enter().append("circle")
    .attr("r", 6)
    .call(force.drag);

var text = svg.append("g").selectAll("text")
    .data(force.nodes())
  .enter().append("text")
    .attr("x", 8)
    .attr("y", ".31em")
    .text(function(d) { return d.name; });

// Use elliptical arc path segments to doubly-encode directionality.
function tick() {
  path.attr("d", linkArc);
  circle.attr("transform", transform);
  text.attr("transform", transform);
}

function linkArc(d) {
  var dx = d.target.x - d.source.x,
      dy = d.target.y - d.source.y,
      dr = Math.sqrt(dx * dx + dy * dy);
  return "M" + d.source.x + "," + d.source.y + "A" + dr + "," + dr + " 0 0,1 " + d.target.x + "," + d.target.y;
}

function transform(d) {
  return "translate(" + d.x + "," + d.y + ")";
}

</script>
