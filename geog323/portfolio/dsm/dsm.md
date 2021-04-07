# Spatial Urban Resilience Analysis of Dar es Salaam

## Background and Question

[This week](https://gis4dev.github.io/lessons/06b_UrbanResilience.html), we chose a dimension of urban resilience in Dar es Salaam, Tanzania, to investigate using data from [OpenStreetMap](https://www.openstreetmap.org/) (OSM) and [Ramani Huria](https://ramanihuria.org/en/), a community mapping project dedicated to improving flood resilience. The OSM data were imported into a PostGIS database by [Joseph Holler](https://www.josephholler.com/) using the [OSM2PGSQL tool](https://osm2pgsql.org/), and the Ramani Huria data were accessed through the web feature service (WFS) of the Ramani Huria project [Resilience Academy](https://resilienceacademy.ac.tz/data/).

[Madeleine Tango](https://mtango99.github.io/) and I chose to investigate the following question: "How does environmental risk and vulnerability due to placement of waste sites near water transmission features vary spatially across Dar es Salaam?" This is an important issue because placement of solid waste sites near water transmission features such as rivers, streams, canals, drains and ditches can lead to flooding during rain events if these waste collections block water transmission and egress. Not only can this result in flooding, but it can also lead to increased contact between humans and pathogens, toxins, and other environmental hazards.

In this analysis we identified waste collection sites within 50 meters of water transmission features as potentially dangerous waste sites and calculated the density of dangerous waste sites for each ward in Dar es Salaam to identify spatial distribution of environmental vulnerability.

<br />

## Data Sources

[OpenStreetMap](https://www.openstreetmap.org/) is a public database editable by anyone with access to the internet. For much of the data in Dar es Salaam, local university students are among the main contributors.
- planet_osm_line: a table of line features in Dar es Salaam compiled by [Joseph Holler](https://www.josephholler.com/) from OSM into a database table using [OSM2PGSQL](https://osm2pgsql.org/)

[Resilience Academy](https://resilienceacademy.ac.tz/) is a Dar es Salaam-based program aimed to equip students with the GIS tools necessary to analyze local challenges and urban resilience (e.g. flood risk).
- [wards](https://geonode.resilienceacademy.ac.tz/layers/geonode_data:geonode:dar_es_salaam_administrative_wards): administrative wards in Dar es Salaam
- [wastesites](https://geonode.resilienceacademy.ac.tz/layers/geonode_data:geonode:dar_es_salaam_trash_data): sites of "poorly managed solid waste" in Dar es Salaam

<br />

## Methods

We completed the following analysis using SQL queries within a PostGIS database through the QGIS DB Manager.

1. Select relevant waterway features (drains, ditches, streams, rivers, and canals) from planet_osm_line layer

  create table waterway_lines as
  select osm_id, waterway, way from planet_osm_line
  where waterway = 'drain' or waterway = 'ditch' or waterway = 'stream' or waterway = 'river' or waterway = 'canal';

2. Transform geometry field of our waterway_lines table to EPSG:32737

`create table waterway_lines_geom as
select osm_id, waterway, st_transform(way,32737)::geometry(linestring,32737) as geom
from waterway_lines;`

3. Create 50m buffers around the water features--this will be the area where we look for dangerous waste sites

`create table buffers_50m as
select osm_id, waterway, st_multi(st_buffer(geom, 50))::geometry(multipolygon,32737) as geom from waterway_lines_geom;`

4. Dissolve the buffers

`create table buffers_50m_dissolve as
select st_union(geom)::geometry(multipolygon,32737) as geom
from buffers_50m;`

5. Convert dissolved buffers to singlepart geometries

`create table singlepart_buffers as
select (st_dump(geom)).geom::geometry(polygon,32737) from buffers_50m_dissolve;`

6. Import wastesites with EPSG:32737

7. Select all waste sites that intersect the buffers

`create table wastesites_in_zones as
select wastesites.*
from wastesites inner join singlepart_buffers
on st_intersects(wastesites.geom, singlepart_buffers.geom);`

8. Join ward names to waste sites based on location

`create table wastesites_with_wardnames as
select wastesites_in_zones.*, wards.ward_name
from wastesites_in_zones inner join wards
on st_intersects(wastesites_in_zones.geom, wards.utmgeom);`

9. Group waste sites by ward

`create table countedwastesites_byward as
select count(id) as countwastesites, ward_name
from wastesites_with_wardnames group by ward_name;`

10. Add area field (in km^2) to wards table

`alter table wards add column area_km2 real;
update wards set area_km2 = st_area(utmgeom)/1000000;`

11. Join count of waste sites back to wards table

`create table wards_with_count as
select wards.*, countedwastesites_byward.countwastesites
from wards left join countedwastesites_byward
on wards.ward_name = countedwastesites_byward.ward_name;`

12. Replace null waste site count values with 0

`update wards_with_count
set countwastesites = 0
where countwastesites is null;`


13. Calculate dangerous waste site density

`alter table wards_with_count add column danger_ws_density real;
update wards_with_count set danger_ws_density = countwastesites / area_km2;`

<br />

## Results

An interactive web map of our results is available [here](https://mtango99.github.io/dsm/assets/#11/-6.7925/39.2333).

According to our analysis, the highest density of waste sites near water transmission features occurs in the center of Dar es Salaam (Figure 1). It is unsurprising that waste sites are so clustered here due to the high population density of the area (Figure 2), but the high prevalence of water transmission features greatly increases the level of environmental hazard posed by the waste sites (Figure 1). In the event of rain, it is highly likely that the accumulated solid waste would block water flow and drainage, leading to increased flood risk in addition to pollution and contamination by pathogens, toxins, and other environmental hazards. This is particularly concerning in areas of high population density such as Dar es Salaam's central wards, where pollution from waste sites can reach large numbers of people very easily in the event of rain.

However, the waste site density choropleth map obscures the full story. As Schuurman ([2008](https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1749-8198.2008.00150.x)) argues, the ethnography of spatial data, or the "social practices" (p. 1529) and particular context through which such data are gathered and compiled, is an essential attribute that researchers far too often fail to consider. In this particular situation, although our results are relatively unsurprising since both population and water transmission features seem to be concentrated in the central wards of Dar es Salaam (Figures 1 and 2), there are also clear gaps in data collection. Documented waste sites are entirely absent from the southwestern and southeastern wards of the city (Figure 3). This could be due to a difference in waste disposal practices between those areas and the central-to-northern wards, but another likely
possibility is that waste site locations were recorded only in part of the city. It is difficult to analyze distribution of environmental risk and vulnerability across the entirety of Dar es Salaam if we are missing data for a wide swath of the city. This problem underscores both the benefits and drawbacks of community mapping efforts. Community mapping has the capacity to pool the knowledge and experience of large and diverse groups of people, but the data may be collected and recorded less systematically and comprehensively than in other types of mapping efforts. In order to best interpret any kind of spatial data it is crucial to know the story behind the rows and columns, and this is particularly important in the case of community- or crowd-sourced data.

![Figure 1.](assets/wastesite_density.png)
Figure 1. <br /><br /><br /><br />

![Figure 2.](assets/wastesite_locations.png)
Figure 2. <br /><br /><br /><br />

![Figure 3.](assets/pop_density.png)
Figure 3. <br /><br />

## References
Schuurman, N. 2008. Database Ethnographies Using Social Science Methodologies to Enhance Data Analysis and Interpretation. Geography Compass 2 (5):1529–1548. 10.1111/j.1749-8198.2008.00150.x
