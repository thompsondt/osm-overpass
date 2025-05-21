# osm-overpass
A Cookbook of Overpass Queries for OpenStreetMap

> [!NOTE]
> **Pardon My Spaghetti**: I'm publishing a handfull of examples to kickoff this cookbook. These come directly from my personal notes.

## For days focused on curb tagging, query for all known and suspected curbs

```
[out:xml][timeout:90][bbox:{{bbox}}];
// find suspected untagged curbs
way({{bbox}})[highway=footway][footway!=crossing];
node(w)->.all_footway_nodes;

way({{bbox}})[footway=crossing];
node(w)->.crossing_nodes;

// the "intersection" of both sets of nodes
node.all_footway_nodes.crossing_nodes->.intersection_nodes;


node.intersection_nodes["barrier"!="kerb"]->.suspected_curbs;

// find known curbs
node({{bbox}})[barrier=kerb]->.known_curbs;

(
  .suspected_curbs;
  .known_curbs;
);
(._;>;);
out meta;
```

## Finding Active Transportation Infrastructure (ATI) near walk generators

```
[out:xml][timeout:90][bbox:{{bbox}}];

// determine the set of streets to focus on
(
  nw["amenity"~"^(school|kindergarden|college|university|bench)?$"];
  nw["shop"~"^(supermarket|convenience|grocery)?$"];
  nw["highway"~"^(bus_stop)?$"];
  nw["leisure"~"^(park|pitch)?$"];
  nw["landuse"~"^(greenery)?$"];

)->.focusArea;

// determine the set ATI near the focus area
(
  nw(around.focusArea:10)["highway"~"^(sidewalk|path|crossing|footway|cycleway)"];
)->.nearbyObjects;

// determine the combined set of the focus area with nearby ATI
(
  .focusArea;
  .nearbyObjects;
  nw(around.nearbyObjects:20)["highway"~"^(sidewalk|path|crossing|footway|cycleway)"];
);
(._;>;);
out meta;
```

## Extend sidewalks and crossings by downloading all active transportation infrastructure and nearby roads

```
[out:xml][timeout:90][bbox:{{bbox}}];

// determine the set of ATI nodes and ways
(
  nw["highway"~"^(sidewalk|path|crossing|footway|cycleway)"];
)->.ati;

// determine the set of roads and highways
(
  nw[highway];
)->.roads;

// determine the set of roads near ATI
(
  nw.roads(around.ati:20);
)->.roadsNearATI;

// determine the set of ATI + roads near ATI
(
  .ati;
  .roadsNearATI;
);
(._;>;);
out meta;
```

## ATI near major roads and bus routes

```overpass
[out:xml][timeout:90][bbox:{{bbox}}];

// determine the set of streets to focus on
(
  nw["highway"~"^(trunk|primary|secondary)"];
  rel[type=route][route=bus];
)->.focusArea;

// determine the set ATI
(
  nw["highway"~"^(sidewalk|path|crossing|footway|cycleway)"];
  nw["amenity"~"^(school|kindergarden|college|university|bench)?$"];
  nw["shop"~"^(supermarket|convenience|grocery)?$"];
  nw[shop];
  nw["highway"~"^(bus_stop)?$"];
  nw["leisure"~"^(park|pitch)?$"];
  nw["landuse"~"^(greenery)?$"];
)->.ATI;

// determine the combined set of the focus area with nearby ATI
(
  .focusArea;
  nw(around.focusArea:100).ATI;
  nw(around.focusArea:100)[highway];
  nw(around.focusArea:100)[railway];
);
(._;>;);
out meta;
```
