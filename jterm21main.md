# MiMiGIS

Visit the MiMiGIS GitHub repository [here](https://github.com/GIS4DEV/MiMiGIS)!

During Winter Term ("J-Term") 2021, I worked with my advisor, Assistant Professor of Geography Joseph Holler, to build the MiMiGIS ("Middlebury Minimal GIS") plugin for QGIS, a prominent open-source desktop GIS. (For more background, see Prof. Holler's blog post about the project [here](https://www.josephholler.com/a-minimal-gis-plugin-for-qgis/).) This plugin delivers a set of easy-to-grasp tools that align with fundamental human geography concepts, designed for students in Middlebury’s introductory GIS course. Although we're starting with just a few tools and resources, the plan is to expand the plugin's functionality over time.

The goal of such a “Minimal GIS” concept is, as Marsh, Golledge and Battersby ([2007](https://www.tandfonline.com/doi/abs/10.1111/j.1467-8306.2007.00578.x)) propose, to focus on development of spatial thinking and comprehension of geographic concepts while lessening the impediment of complicated and sometimes non-intuitive tools. Over the course of the past few semesters, Prof. Holler had already created a couple of models for the introductory GIS class using the QGIS Graphic Modeler: `Group By` and `Direction and Distance`.

`Group By` allows the user to group features in a layer by values in one or more fields ("group fields") and to calculate summary statistics for any number of fields ("summary fields"). If the input layer has geometry information, the user can also choose to dissolve geometries based on the group fields. This tool is extremely helpful because QGIS lacks a Dissolve tool with functionality comparable to that of the [ArcGIS Dissolve tool](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/dissolve.htm). Specifically, there is no dissolve tool in QGIS that allows the user to select multiple group fields, multiple summary fields, and the user's choice of summary statistics.

`Direction and Distance` allows the user to calculate the distance and azimuth from an origin point to all features in an input layer. This tool is particularly useful for teaching concepts of urban structure because it allows students to examine spatial relationships between the central business district of a city and the surrounding census tracts or block groups.

Another of Prof. Holler's innovations was to customize a set of Mapbox Maki icons and National Park Service (NPS) map icons (in the form of SVGs) to allow the QGIS user to change their color. The icons come automatically with black fill color, but Prof. Holler replaced that with the QGIS fill color parameter for easily customizable SVG symbology. The user just had to download Prof. Holler's icon folder and add a path to that location under "SVG Paths" in their QGIS settings.

My task for J-Term was to package these tools into a QGIS plugin that anyone can download from the official [QGIS plugins repository](https://plugins.qgis.org/plugins/). First, I needed to convert the `Group By` and `Direction and Distance` models into Python scripts, making a few minor revisions along the way. Then I needed to create a Processing Provider plugin and add those algorithms. Finally, I needed to add functionality into the plugin to automatically install the SVGs when the plugin is enabled.

Although I had [experience writing Python algorithms for QGIS](https://github.com/majacannavo), I had never built a plugin before, and I had a lot to learn. Follow my journey through the weekly blog posts linked below, and take a look at the repository linked at the top of this page!

[Week 1: Getting Started](https://majacannavo.github.io/jterm21week1)

[Week 2: Getting `Group By` Up and Running](https://majacannavo.github.io/jterm21week2)

[Week 3: Finishing the Plugin](https://majacannavo.github.io/jterm21week3)
