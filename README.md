# Extracting coastline vectors from *planet.osm* 
This document discusses various methods to extract, from the *Open Street Map* dataset, a set of vectors representing the coastline and some of the challenges related to each method.

# The *planet.osm* file 
This section provides some basic technical info on the way Open Street Map publishes data.

# What is the "coastline"
Well there's an entire wiki page on that. <https://wiki.openstreetmap.org/wiki/Coastline>  
Basically, it depends. Empirically, there is no consistency among OSM contributors.

## "natural"="coastline"
<https://wiki.openstreetmap.org/wiki/Tag:natural%3Dcoastline>  
This is special key that is only applied to *ways*. This tag is almost always applied correctly and consistently. This, combined with its usage sematics, makes it the only tag that (almost) defines a winding direction. (More on this below.)  
(Sidebar: When applied incorrectly this tag can cause bad rendering artifacts. Since the most active users of OSM polygons are people rending base maps, errors get corrected or reverted quickly.)
### Water must be on the right
From the wiki linked above:
> The direction the ways are drawn is very important! They must be drawn so that the land is on the left side and water on the right side...  

In practice, this is half right. The *natural=coastline* pair guarantees water on the right, but __do not__ assume that land is on the left. A *way*, or some subset of its nodes, may also be labeled as the boundary of a water polygon. This is common for river mouths. In this case, water is on the left (and the right). This is necessary since ways labeled as coastline are expected to form a closed ring around continents or islands.  
Labeling the *way* as the exterior of a water polygon does not guarantee that the left side is water. River-banks may be labeled as coastline; i.e. the river is effectively inside the ocean. A river bank bordering land may (or may not) be labeled exactly the same as a river mouth bordering the ocean. Features like *estuaries* that can be internal to other water polygons can further complicate determining whether there is indeed land on the left side. (Yes, none of this makes sense.)

## Water Polygons
