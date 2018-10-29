# Extracting coastline vectors from *planet.osm* 
This document discusses various automated methods to extract, from the *Open Street Map* dataset, a set of vectors representing the coastline and some of the challenges related to each method.

# The *planet.osm* file 
This section provides some basic technical info on the way Open Street Map publishes data.  
The *planet.osm* file and its various extracts consist of 3 types of entities. Each entity has a positive integer id and some associated metadata in the form of key=value pairs.
1. Node - A point with a latitude and longitude, with precision of about 1cm.
2. Way - A sequence of nodes. Basically a linestring. Order is important. May be closed. 
    * e.g. a closed way with natural=water may be a pond.
3. Relations - Usually a set of ways. Make no assumptions about order.  
    * e.g. a relation with type=multipolygon can be used to represent a river, possibly with islands.

Files are usually sorted s.t. nodes are first, then ways, then relations. The author calls this order "the worst one possible."

For a quick sense of scale - just so we know what we're up against - there are currently (Oct-2018) 4.8billion nodes in the OSM dataset. (https://www.openstreetmap.org/stats/data_stats.html) That means we need 33 bits to represent an id. (The id space is dense but not strictly sequential.) Let's round that up to the next register size, say 64 bits. Latitude and Longitude are each encoded as 32 bit integers. So to store a node with its id and location well need 8+4+4=16 bytes. So to store all of the nodes we'll need 4.8&#8226;G&#8226;node * 16&#8226;bytes/node = 76.8&#8226;Gbytes!  
In conclusion, there are already a lot of nodes and this number grows pretty quickly.

# What is the "coastline"
Well there's an entire wiki page on that. <https://wiki.openstreetmap.org/wiki/Coastline>  
Basically, its ambiguous. Empirically, there is no consistency among OSM contributors. 

The coastline as perscibed in the OSM dataset is rather conservative.  
<a href=images/osm_coast.png><img src=images/osm_coast.png width=400> </a>  
For many applications we'll want to augment these vectors with the boundaries of the water reachable from the sea.

## "natural"="coastline"
<https://wiki.openstreetmap.org/wiki/Tag:natural%3Dcoastline>  
This is special key-value pair that is only applied to *ways*. This tag is unqiue in being applied consistently and correctly. This, combined with its water-on-the-right sematics, makes it the only tag that (almost) defines a winding direction. (More on this below.)  
Sidebar: When applied incorrectly this tag can cause bad rendering artifacts. Since the most active users of OSM polygons are people rendering base maps, errors get corrected or reverted quickly.
### Water must be on the right
From the wiki linked above:
> The direction the ways are drawn is very important! They must be drawn so that the land is on the left side and water on the right side...  

In practice, this is half right. The *natural=coastline* pair guarantees water on the right, but __do not__ assume that land is on the left. A *way*, or some subset of its nodes, may also be labeled as the boundary of a water polygon. This is common for river mouths. In this case, water is on the left (and the right). This is necessary since ways labeled as coastline are expected to form a closed ring around continents or islands.  
Labeling the *way* as the exterior of a water polygon does not guarantee that the left side is water. River-banks may be labeled as coastline; i.e. the river is effectively inside the ocean. A river bank bordering land may (or may not) be labeled exactly the same as a river mouth bordering the ocean. Features like *estuaries* that can be internal to other water polygons can further complicate determining whether there is indeed land on the left side. (Yes, none of this makes sense.)

The images below show two different river/sea interfaces. Both have opinionated and diligent stewards. 
<a href=images/riverbank1.png> <img src=images/riverbank1.png width=400></a>
<a href=images/riverbank2.png> <img src=images/riverbank2.png width=400></a>

## Water Polygons
To generate a more complete coastline we'll need to include the water polygons that are reachable from the sea. 
### Polygons
* Polygons are reprsented in the OMS data in 2 ways.
    1. A closed *way*.
    2. A relation labeled type=multipolygon.  

Polygons do not have winding direction. A closed way can be CW or CCW. The order and orientation of *ways* within a multipolygon *relation* should be considered random. This makes assembling polygons more complicated but somewhat simplifies editing. (More on trying to correctly edit polygons later!)  
The only reliable way to find the interior side of a polygon is to load the location of every node and perform a ray-cast or similar. Note: there are *lots* of nodes.
### Reachability
We'll consider a polygon to be reachable from another if the boundary shares at least 2 consecuitve nodes. 
