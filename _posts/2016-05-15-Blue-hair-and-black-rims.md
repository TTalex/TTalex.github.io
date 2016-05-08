---
layout: single
title: Blue hair and black rims
header:
 image: Blue-hair-and-black-rims/header.jpg
 teaser: Blue-hair-and-black-rims/header.jpg
categories: [Code]
Tags: [Javascript, Jquery, html, svg filters, css, photoshop, images]
excerpt: Dynamically changing colors of a part of a picture, using jquery and svg filters
published: false
---
Hey Senskers !

# The story
A little while ago, while I was fiending my favorite sub-reddits as I often did in my (now missed) extensive spare time, I ran into one of the most common questions of /r/picrequest. For those unfamiliar with this sub-reddit, people come in and ask for photo edits to a community of photoshop "experts" osciliating between having fun and not realizing that they should be paid for their work. Anyway, it was one of the classical "please change the color of this" type of request.

<img src="Blue-hair-and-black-rims/request.jpg"></img>

What poor /u/Kuri619 ignored back then was how easy this is to do for a photoshop amateur and not only could the color be changed to black, but it could be done to any color ... within the software itself.

<img src="Blue-hair-and-black-rims/photoshop.gif"></img>

What if I could reproduce this behavior within a web page, for all Kuris out there to use ? The first thing I delivered was an html snippet, very not user friendly, where you had to manually change rgb values out of a matrix ... a mess, but the guy was quite happy with it.

Fast forward a few days, a lady was asking for a similar job, except she wanted to see herself with hair colored blue or red or whatever other color she had in mind at the time.

The need has been identified, let's get building !

# The solution
So we're building a webpage, here's how the final page looks. Not too fancy and straight to the point.

<img src="Blue-hair-and-black-rims/demo.jpg"></img>

The user feeds in two images of similar sizes, one is the image we want to edit, that will act as a background, the other contains only the parts we want to color with a transparent background (only the rims for example).

Using jquery, the link inputs are read when changed, and the images displayed.
```html
<input type="text" style="float:right" id="bgimg" value="http://i.imgur.com/LIKTzdo.jpg"/><br />
<input type="text" style="float:right" id="cutout" value="http://i.imgur.com/T8xJfrX.png"/>
<svg  viewBox="0 0 1 1" style="height:80%; position: fixed; bottom:0;>
    [...]
    <image x="0" y="0" width="100%" height="100%" xlink:href="http://i.imgur.com/LIKTzdo.jpg" id="dbg"/>
    <g filter="url(#color)">
        <image x="0" y="0" width="100%" height="100%" filter="url(#whity)" xlink:href="http://i.imgur.com/T8xJfrX.png" id="dcut"/>
    </g>
</svg>
```
```javascript
$("#bgimg").on("change paste keyup", function(){
$("#dbg").attr("xlink:href", escapeHtml($(this).val()));
});
$("#cutout").on("change paste keyup", function(){
$("#dcut").attr("xlink:href", escapeHtml($(this).val()));
});
```

The first thing that can be noted is that every user input is escaped using the escapeHtml function, this is maily done so that the page doesn't break when missused.

```javascript
function escapeHtml(text){
    var map = {
    '&': '&amp;',
    '<':'&lt;',
	'>':'&gt;',
      '"':'&quot;',
      "'":'&#039;'
      };
      return text.replace(/[&<>"']/g, function(m){ return map[m];});
}
```

The second thing are the usage of svg filters on the cutout image, overlapping the background one. Filters using feColorMatrix are pretty simple to use once you get the gist of it. The four lines define red, green, blue and alpha chanels to apply to each of the image pixels. In order to apply coloration to any image, including very dark ones, the luminosity needs to be boosted, this is what the "whity" filter does. The second filter, "color", leaves the image unchanged for now.
```html
<svg  viewBox="0 0 1 1" style="height:80%; position: fixed; bottom:0;">
    <defs>
        <filter id="whity">
        <feColorMatrix in="SourceGraphic" type="matrix" 
             values="
                 0.9 0.9 0.9 0 0 
                 0.9 0.9 0.9 0 0 
                 0.9 0.9 0.9 0 0
                 0 0 0 1 0
                 "/>
        </filter>
        <filter id="color">
        <feColorMatrix in="SourceGraphic" type="matrix" id="mtrx"
             values="
                 1 0 0 0 0 
                 0 1 0 0 0 
                 0 0 1 0 0
                 0 0 0 1 0
                 "/>

        </filter>
    </defs>
  [...]
</svg>
```

Now that the image modifications are set up, we need a color selector. I opted to go with a premade one, called <a href="https://github.com/bgrins/spectrum">spectrum</a> written in javascript. It was only a matter of placing it on the webpage and connecting to our "color" filter matrix.

Note that only the diagonal values of the matrix are changed in order to affect the channels, filters could be used for way more effect that coloration using the other dimentions.
```html
<div id="flat">
</div>
<script type="text/javascript">
    $("#flat").spectrum({
        flat: true,
        showInput: true,
        showButtons: false,
        showAlpha: true,
        color: "rgba(255,255,255,1)",
        preferredFormat: "rgb",
        move: function(color){
            $("#mtrx").attr("values", (color._r)/250+" 0 0 0 0 0 "+(color._g)/250+" 0 0 0 0 0 "+(color._b)/250+" 0 0 0 0 0 "+color._a+" 0");
        }
    });
</script>
```

# The Code
https://github.com/TTalex/color-changer
# The Sources
* Spectrum for color selection using javascript: <a href="https://github.com/bgrins/spectrum">https://github.com/bgrins/spectrum</a>
* <a href="https://jquery.com/">The infamous JQuery</a>
* <a href="https://docs.webplatform.org/wiki/svg/tutorials/smarter_svg_filters">More info on svg filters</a>
* <a href="https://www.reddit.com/r/picrequests">/r/picrequests on reddit</a>