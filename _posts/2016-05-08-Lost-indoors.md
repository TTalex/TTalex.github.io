---
layout: single
title: Lost indoors
header:
 image: Lost-indoors/header.jpg
 teaser: Lost-indoors/header.jpg
categories: [Code]
Tags: [Javascript, html, svg, d3js, angular]
excerpt: Visualization of indoor movement using d3.js and angular.js
published: true
---
Hey Senskers !

# The story

As part of one of my pro-jects (professional project), I needed a way to visualize indoors movements. This is taking part in the very interesting research field of "indoor GPS" or IPS (Indoor positioning system). However, I won't be writing on the way of gathering data in this article, I'll let you guys figure out something with radio waves, magnetic fields, mobile devices or sensors (hint hint). This article is mainly about visualization of a path taken by a person or object in an indoor environment, let's say a house. We're merely trying to display in a fancy way an ordered set of location identifiers.
![image-center](/images/Lost-indoors/intro.jpg){: .align-center}

# The solution

The goal is to make an animated svg image, here's the final result.

![image-center](/images/Lost-indoors/demogif.gif){: .align-center}

### Setting up an angular environment

First of all, let's set up a very simple angular project, with routing and a service reaching a remote API. The file structure should look as follows.

```bash
.
|   index.html
|   
+---css
|       main.css
|       
+---images
|       floorplan.jpg
|       
+---js
|   |   app.js
|   |   
|   +---controllers
|   |       mainController.js
|   |       
|   +---directives
|   |       floormapDirective.js
|   |       
|   \---services
|           mainService.js
|           
\---views
        main.html
```        

The **index.html** file loads up d3, Angular and the routing library. It also sets up a div to hold our views with ng-view.

**Remember** to add Controllers, Services and Directives scripts in here as well.
{: .notice--info}

```html
<!-- index.html -->
<!doctype html>
<html>
  <head>
    <link href="css/main.css" rel="stylesheet" />    
    <script src="http://d3js.org/d3.v3.js"></script>

    <!-- Include the core AngularJS library -->
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>

    <!-- Include the AngularJS routing library -->
    <script src="https://code.angularjs.org/1.2.28/angular-route.min.js"></script>
  </head>
  <body ng-app="myApp">
    <div ng-view></div>
    <!-- Modules -->
    <script src="js/app.js"></script>

    <!-- Controllers -->
    <script src="js/controllers/mainController.js"></script>

    <!-- Services -->
    <script src="js/services/mainService.js"></script>
    
    <!-- Directives -->
    <script src="js/directives/floormapDirective.js"></script>

  </body>
</html>
```

In **js/app.js**, we create our angular application and define routing. In this case, only one route is accessible.

```javascript
// js/app.js
var app = angular.module('myApp', ['ngRoute']);

app.config(function ($routeProvider) { 
  $routeProvider 
    .when('/', { 
      controller: 'mainController', 
      templateUrl: 'views/main.html' 
    })
    .otherwise({ 
      redirectTo: '/' 
    }); 
});
```

Let's define our mainController in **js/controllers/mainController.js** and its associated view template in **views/main.html**.

 * **positionmap** is an hard coded list of coordinates [x, y] defining indoor positions such as rooms.
 * We fetch **data** from our mainService to get the ordered list of positions the path takes
 * We feed information to the view using **$scope.data** containing a list of points to follow
 * The view only displays the list for now

```javascript
// js/controllers/mainController.js
app.controller('mainController', ['$scope', 'mainService', '$routeParams', function($scope, mainService, $routeParams) {
  var positionmap = [ [ 467, 367 ], [ 626, 401 ], [ 616, 323 ], [ 610, 160 ], [ 443, 173 ], [ 351, 67 ], [ 272, 173 ], [ 90, 114 ], [ 90, 114 ], [ 84, 253 ], [ 94, 412 ], [ 303, 323 ], [ 388, 374 ] ];
  mainService.success(function(data) {
    // data is shaped like data = { data: [1, 2] }
    var i, points = [];
    for (i = 0; i < data.data.length; i = i + 1) {
        points.push(positionmap[data.data[i] - 1]);
    }
    $scope.data = points;
  });
}]);
```

```liquid
<!-- views/main.html -->
<div id="main">
<h1>Wow such great house</h1>
{{ data }}
</div>
```

The **js/services/mainService.js** used by our controller consists of a very basic HTTP get request to a remote API (or a gist file in our example).

```javascript
// js/services/mainService.js
app.factory('mainService', ['$http', function($http) {
  return $http.get('https://gist.githubusercontent.com/TTalex/f732532f61c900827078c318a1622f56/raw/f1beca357308ff41224714da280470679c64b519/sample-data.json');
}]);
```

Finaly a **css/main.css** file is there to beutify everything.

```css
/* css/main.css */
#main{
    text-align: center;
}
path{
    stroke-width: 5px;
    stroke: steelblue;
    fill: none;
}

.dot {
  fill: white;
  stroke: steelblue;
  stroke-width: 1.5px;
}
```

### Building the d3.js directive

Now that our angular project is all set up and ready to serve, it's time to build and integrate our d3.js directive.

As [Brian Ford](http://briantford.com/blog/angular-d3) mentions in his blog, building directives from d3.js code isn't a very hard task. It's only a matter of copying and pasting and replacing the original DOM selector.

This code is very much inspired by the [Stroke Dash Interpolation code by Mbostock](http://bl.ocks.org/mbostock/1705868), the creator of D3.js. I highly encourage you Senskers out there to compare both if you want to learn how to adapt any D3js code to angular directives.
{: .notice--info}

First let's create the **js/directives/floormapDirective.js** file. In addition to the code provided by the Stroke Dash Interpolation example, we're also loading an image that will act as a background plan for our floor. Adding the image source as a scope for our directive could be an interesting improvement.

```javascript
// js/directives/floormapDirective.js
app.directive('floormap', function() {
    return {
        restrict: 'E',
        scope: {
          data: '=',
          width: '=',
          height: '='
        },
        link: function (scope, element, attrs) {
            // Initalize height and width of our working space
            var width = scope.width;
            var height = scope.height;
            // Append main svg element under <floormap>
            var svg = d3.select(element[0]).append("svg")
                .attr("width", width)
                .attr("height", height)
            
            // Append the background image
            svg.append("svg:image")
               .attr('width', width)
               .attr('height', height)
               .attr("xlink:href","images/floorplan.jpg")

            // Create a line object, not data bound to it yet
            var line = d3.svg.line()
                .interpolate("cardinal")
            
            // transition & tween Dash are used to animate the path
            function transition(path) {
              path.transition()
                  .duration(7500)
                  .attrTween("stroke-dasharray", tweenDash)
                  .each("end", function() { d3.select(this).call(transition); });
            }
            function tweenDash() {
              var l = this.getTotalLength(),
                  i = d3.interpolateString("0," + l, l + "," + l);
              return function(t) { return i(t); };
            }
            // We watch changes on the data value
            scope.$watch('data', function (newVal, oldVal) {
                // Skip when newVal is not set
                if (!newVal){
                    return
                }
                // Bind newVal data to our svg object
                svg.datum(newVal);
                // Append a dashed path following our line object
                svg.append("path")
                    .style("stroke", "#666")
                    .style("stroke-dasharray", "4,4")
                    .attr("d", line);
                // Append a path following our line object, this is the animated one
                svg.append("path")
                    .attr("d", line)
                    .call(transition);
                
                // Append dots at each data points
                svg.selectAll(".dot")
                    .data(newVal)
                  .enter().append("circle")
                    .attr("class", "dot")
                    .attr("cx", line.x())
                    .attr("cy", line.y())
                    .attr("r", 5);
            });
        }
    };
});
```

Once the directive is created, remember to add the file to our **index.html**.

```html
<!-- index.html -->
[...]
    <!-- Directives -->
    <script src="js/directives/floormapDirective.js"></script>

  </body>
</html>
```

Now we can add the directive to our template file **views/main.html**.

```html
<!-- views/main.html -->
<div id="main">
<h1>Wow such great house</h1>
<floormap data="data" width="720" height="487"></floormap>
</div>
```

TADA ! Everything should now be working fine, animations included.

### Bonus: Calibration

Hang on a minute, are you really expecting us Senskers to manually find out all coordinates defining indoor positions for our positionmap ?!
{: .notice--warning}

That would be insane, wouldn't it ? Let's create a webpage that will help generating this positionmap with simple clicks on an image. We'll call it the calibration page.

First, let's add a route to our **js/app.js** file, right before the otherwise route.

```javascript
// js/app.js
.when('/calibration', {
  controller: 'calibrationController', 
  templateUrl: 'views/calibration.html' 
})
```

Then we create the **js/controllers/calibrationController.js** and its associated **view/calibration.html**. First the user presses a button to start calibrating, then clicks on the image for each position he wants to register. This triggers the input function with an associated [JQuery event](http://api.jquery.com/category/events/event-object/) (thanks to $event) from which we fetch the mouse positions and image offsets. Once done, the user can copy the generated positionmap.

```javascript
// js/controllers/calibrationController.js
app.controller('calibrationController', ['$scope', '$routeParams', function($scope, $routeParams) {
  $scope.result = [];
  $scope.cali_value = -1;
  $scope.calibrate = "Start calibration";
  $scope.toggleCalibrate = function () {
    if ($scope.calibrate === "Start calibration") {
        $scope.calibrate = "Stop calibration";
        $scope.cali_value = 0;
    } else {
        $scope.calibrate = "Start calibration";
        $scope.cali_value = -1;
    }
  }
  $scope.input = function (event) {
    if ($scope.cali_value !== -1){
        $scope.result[$scope.cali_value] = [event.pageX - event.originalTarget.offsetLeft, 
                                            event.pageY - event.originalTarget.offsetTop];
        $scope.cali_value += 1;
    }
  }
}]);
```

```liquid
<!-- views/calibration.html -->
<div>
    <div>
    <img src="images/floorplan.jpg" ng-click="input($event)" height=487 width=720></image>
    </div>
    <div>
        <input style="float: left" type="button" ng-value="calibrate" ng-click="toggleCalibrate()"></input>
        <div ng-show="cali_value !== -1" style="float: left">Calibrating room {{ cali_value }}</div>
        <div style="clear:left">{{result | json}}</div>
    </div>
</div>
```

There we go, our application is completed, it should be noted that there should be more error management throughout the code, especially around the service call. For example, the program fails when a position identifier fetched from our API does not match a positionmap element. However, I'm sure Senskers are smart enough to add and improve on many points, and I would be pleased to hear about them !


Thanks for reading this one !

# The Code
[https://github.com/TTalex/angular-d3js-indoor-map](https://github.com/TTalex/angular-d3js-indoor-map)

# The Sources
* [Indoor positioning system Wiki](https://en.wikipedia.org/wiki/Indoor_positioning_system)
* [Xkcd comics editor](http://cmx.io/)
* [Angularjs](https://angularjs.org/)
* [D3js](https://d3js.org/)
* [D3-floorplan, a project close to this one by Dciarletta](https://dciarletta.github.io/d3-floorplan/)
* [Using d3.js in angular with directives by Brian Ford](http://briantford.com/blog/angular-d3)
* [D3js Stroke Dash Interpolation by Mbostock](http://bl.ocks.org/mbostock/1705868)
* [D3js Line Interpolation by Mbostock](http://bl.ocks.org/mbostock/3310323)
* [Some reminder on svg animation](https://css-tricks.com/svg-line-animation-works/)
* [Jquery events, also used in angular](http://api.jquery.com/category/events/event-object/)