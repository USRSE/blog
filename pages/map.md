---
layout: page
title: USRSE Map
description: Where is the Research Software Engineer Community?
permalink: /map/
---

<style>
.page-heading {
    padding-bottom: 10px !important;
}
</style>

Where are the RSEs that participate in this community blog on the map?

<div id="map-container" style="height:800px"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.5.1/dist/leaflet.css"
      integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
      crossorigin=""/>

<script src="https://unpkg.com/leaflet@1.5.1/dist/leaflet.js"
     integrity="sha512-GffPMF3RvMeYyc1LWMHtK8EbPv0iNZ8/oTtHPx9/cc2ILxQ+u905qIwdpULaqDkyBKgOaB57QTMg7ztg8Jm2Og=="
     crossorigin=""></script>

{% include map.html %}

<br/>
<br/>
Map made with [http://leafletjs.com](http://leafletjs.com), and [Open Streetmap](https://osm.org/), and idea from [DE-RSE](https://github.com/DE-RSE/www). Thank you!
