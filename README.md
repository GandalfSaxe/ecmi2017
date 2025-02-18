# ECMI 2017: Snowplow Route Optimization

[ECMI Modelling Week 2017](http://www.mafy.lut.fi/ECMIMW2017/) was a mathematics modeling week in Lappeenranta, Finland on July 9th - July 16th 2017. There were 9 teams, one for each of the [problems](http://www.mafy.lut.fi/ECMIMW2017/index.php?page=problems).

Project Lambda was about optimization of snow plowing operations in Skinnarila area near Lappeenranta. We wanted to minimize the distance travelled by the snow plowing vehicles, and if possible, prioritize roads differently.


**Table of Contents**

- [Introduction](#introduction)
- [1. Problem and results](#1-problem-and-results)
	- [Presentation](#presentation)
	- [Report](#report)
	- [Animations](#animations)
		- [Optimal solver animation](#optimal-solver-animation)
		- [Penalty scout animation](#penalty-scout-animation)
- [2. How to use code](#2-how-to-use-code)
	- [Obtain graph of road network](#obtain-graph-of-road-network)
	- [Obtain solutions for efficient routes](#obtain-solutions-for-efficient-routes)
		- [Find optimal route using classical algorithm](#find-optimal-route-using-classical-algorithm)
		- [Find optimal route using stochastic (penalty scout) algorithm](#find-optimal-route-using-stochastic-penalty-scout-algorithm)
	- [Convert list of nodes into list of coordinates](#convert-list-of-nodes-into-list-of-coordinates)
	- [Visualize solutions using Google Maps API](#visualize-solutions-using-google-maps-api)
	- [Add red motion lines to the route animation](#add-red-motion-lines-to-the-route-animation)
- [The Team](#the-team)

# Introduction
The problem turned out to be a variation of the [Chinese Postman Problem](https://en.wikipedia.org/wiki/Route_inspection_problem) (CPP) (a.k.a route inspection problem) on a graph having road intersections as nodes and intersection distances as edges. With this basic model, the problem basically had three parts:

1. **Data acquisition**
2. **Solution**
3. **Visualizing solution**

And in summary we found:

1. **Data acquisition:** We opted for the Google Maps API to create our graph, despite the free API-key limitations, since it was very easy to work with. We ended up making a graph of the limited area of priority 1 roads, in a very manual fashion.
2. **Solution**: We basically worked on two solutions: 1. Optimal solution of basic CPP problem using by finding optimal pairings of odd-degree vertices (aided by [Blossom algorithm](https://en.wikipedia.org/wiki/Blossom_algorithm)) in order to restructure the graph into an Eulerian graph, for which the CPP can then solved in polynomial time, and 2. A stochastic algorithm (a.k.a. "penalty scout") with a cost function that penalize undesirable moves such as traveling the same edge multiple times, making needless U-turns etc.
3. **Visualizing solution**: We used the [Google Maps Directions API](https://developers.google.com/maps/documentation/directions/) to obtain coordinates for routes between read intersections and [Google Maps Javascript API](https://developers.google.com/maps/documentation/javascript/) to animate a symbol on the maps (luckily Google had provided a some [nice sample code](https://developers.google.com/maps/documentation/javascript/examples/overlay-symbol-animate)). This could be done after the solution was converted from a list of nodes to a list of (latitude, longitude) coordinates, using the Google Maps Directions API

# 1. Problem and results
The total length of all the edges in our (one-way) graph is 24.2355 km.
However, since it's not Eulerian, this distance is not achievable and the snowplow would have to travel a longer distance. We found the theoretical lower bound of a solution to be 30.5275 km, which is exactly what both algorithms found.

## Presentation
[Link to presentation (PDF)](https://github.com/GandalfSaxe/ecmi2017/blob/master/presentation/presentation.pdf)  
[Link to presentation (Overleaf, read-only)](https://www.overleaf.com/read/ywjqndnytwkz)

## Report
[Link to report (PDF)](https://github.com/GandalfSaxe/ecmi2017/blob/master/report/report.pdf)  
[Link to report (Overleaf, read-only)](https://www.overleaf.com/read/xfgsdmqmwjvf)

## Animations

### Optimal solver animation
![Alberithm solution](https://github.com/GandalfSaxe/ecmi2017/blob/master/map-plotting/animation/animation-videos/final-animation-videos/alberithm.gif?raw=true)  
*Total travel distance: 30.5275 km. Single-driver, priority 1 streets only.*

### Penalty scout animation
![Carlgorithm solution](https://github.com/GandalfSaxe/ecmi2017/blob/master/map-plotting/animation/animation-videos/final-animation-videos/carlgorithm.gif?raw=true)  
*Total travel distance: 30.5275 km. Single-driver, priority 1 streets only.*


# 2. How to use code
Due to the time constraints, the following is a series of not-so-pretty, but functional workflows of obtaining road graph data, obtaining route solutions and visualizing them using Google Maps API.

## Obtain graph of road network
During the week, we tried to find a way to automatically generate a graph of a road network. The naive idea was following several steps:
1. Take a list of street names
2. Find all pairs of streets, which share an intersection and find the coordinates of this intersection
3. Enumerate all intersections
4. Identify for each intersection the neighbor intersections and find the distance from each intersection to its neighbors
5. Use the values from step 4 to create a distance matrix
6. With the distance matrix its easy to create a graph

The thing about the workflow just mentioned is, that (not only) Lappeenranta has several intersections where a street crosses itself. Maybe also more than once which makes it difficult to identify all intersections just using the street list. It is also very hard to automatically identify neighbors for each intersection. But this part is essential, because if neighbors are missing, then our graph is not covering every street, but if we identify to many neighbors our graph covers some streets twice or even more.

Also for all streets in Lappeenranta we would had to make around 50k Google Maps API requests only for finding all intersections, which exceeds the limit of 2500 free requests.

## Obtain solutions for efficient routes
We arrived at the following two methods:
* Solution 1: the classical algorithm (Eulerian-augmented graph + blossom) algorithm
* Solution 2: the stochastic (penalty scout) algorithm

See presentation and report for in-depth descriptions.

### Find optimal route using classical algorithm
All functions for the classical algorithm are in the `chinese_postmal_algorithm.py` file.
1. Load graph from the `example_graph.py` script or create your own using networkx library.
2. Call the `solve_chinese_postman_problem.py` function with the graph as first argument.
3. Specify start/ending node with the "start" and "end" keyword arguments.
	3a. If both are not specified (set to "None") random ones will be used.
	3b. If start is specified but "end" is set to None, the output path will be closed.
4. If the input is a multigraph (for roadmaps with two-way streets) set "is_multigraph" keyword argument to True.
5. Set "matching_algorithm" to efficient for Blossom algorithm, or "original" for brute force (not recommended).
6. Function will return the total distance traveled and a path that solves the chinese postman problem.
7. If there is an error and the solution is wrong or the distance is not optimal, a message will be shown on screen.

### Find optimal route using stochastic (penalty scout) algorithm
All code you need for finding a solution with our Penalty Scout Algorithm is within the `Penalty_Scout_Algorithm.py` file.
1. Create graph with the `manual_map_one` function, which is calling a distance matrix from a CSV file. (If needed, add attributes such as visits within the function loop)
2. Within the `valuevertex` function you can change which characteristics increase the value (our penalty).
3. Within the `nextpoint` function, the `borders` variable is representing probabilities for each element. Change several entries, if lower or higher probabilities are wanted.
4. Choose how many iterations you want to try and call the `start_penalty_scout` function. It will return the best candidate of all N attempts and the iteration number. And it will save a CSV-file containing all improvements and paths.
5. Best path found will be last entry in the saved `penaltyscoutloglength.csv` file.

## Convert list of nodes into list of coordinates
1. Open `map-plotting/google_maps_plotting2.Rmd` (using RStudio).
2. Install R googleway package if you haven't: `install.packages(googleway)`.
3. Start by inserting two API keys: 1. Copy your [Google Maps Directions API Key](https://developers.google.com/maps/documentation/directions/) into the variable `key` (used to find coordinates of roads between intersections) and the [Maps JavaScript API Key](https://developers.google.com/maps/documentation/javascript/) into the variable `map_key` (used to draw coordinates on a map).
4. Insert the route solution as a list of nodes in the variable `routeNodes`. For example: `routeNodes <- c(0,1,2,3,4,3,5,3,6)`.
5. Run the whole script in `google_maps_plotting2.Rmd`. The whole route has now been written into file `route.csv` as a (latitude, longitude) coordinate list, in the same directory.
6. Run `route-conversion.py`. It's a very simple 16-line python script that converts the (lat,lon) list in `route.csv` into a list format in file `route-lat-lon.txt`, which is needed in the final script.

## Visualize solutions using Google Maps API
1. Open file `map-plotting/animation/polyline-animation_googledev.html`. At the bottom of the script, insert your Maps JavaScript API key into the string variable `src="https://maps.googleapis.com/maps/api/js?key=INSERT-MAPS-JS-API-KEY-HERE&callback=initMap">`. 
2.  Paste the contents of into the file `route-lat-lon.txt` into variable `var lineCoordinates`.
2. Open `polyline-animation_googledev.html` in a browser. Enjoy :)

As a final step, we then screen recorded the animation, trimmed, cropped, recompressed it and added a red trail (see next section)

## Add red motion lines to the route animation
1. Run script `map-plotting/animation/animation-videos/red-motion-lines.py` on video file.

Special thanks to my friend [jakejhansen](https://github.com/jakejhansen) for this script.

*Warning: may require a somewhat cumbersome installation of `OpenCV` in python.*


# The Team
[Albert Miguel López](https://github.com/amiguello)  
[Carl Assmann](https://github.com/carlassmann)  
[Edyta Kabat](https://github.com/edyta-kabat)  
[Gandalf Saxe](https://github.com/GandalfSaxe)  
[Matthew Geleta](https://github.com/MatthewGeleta)  
[Sara Battiston](https://www.facebook.com/BlackkRoseImmortal)

