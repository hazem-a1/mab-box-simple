# mab-box-simple
how to get started with map box api


### in this tutorial we will use map box with geojson data

<https://www.mapbox.com/>

which will gave you 50 k request for free just make account and get key

then make html and add this tags

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
   <script src="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js"></script>
    <link href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" rel="stylesheet" />
</head>
<body>
    
</body>
</html>
```

#### if you want to use it in spa you can find SDK for your chosen framework

##### now we are 2 steps away

add dev with id to target style

```html
<div id="map"> </div>
```

add style to the head tag

```css
 <style>
    body {
      margin: 0;
      padding: 0;
    }

    html,
    #map {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
    }
  </style>
```

#### pro tip: if you want the map be part of the screen to fit author component just assign only the height property and add container to control the style

##### now add script tag with your access token

```javascript
mapboxgl.accessToken = 'YOUR_ACCESS_TOKEN';
var map = new mapboxgl.Map({
    container: 'map', // div name
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [35, 35], // set coords to be in center of the screen
    zoom: 5, // zoom lv 
    interactive: false, // moveable mab or no
});
```

> ##### what if you wanna add a custom  image on top of the map
>
> you have access to map.on("load", cb) 
>
> which take a callback function  then you can call map.loadImage() 

```javascript
map.on("load", function () {
    map.loadImage(
         "https://docs.mapbox.com/mapbox-gl-js/assets/custom_marker.png",// image uri here
        async function (error, image) {
            if (error) {
                console.log(error)
            }
            this.map.addImage('logo', image);
        })

})
```

#### now we successfully added an external resource and have access to it inside map.on("load")

### but yet now image added to the map we have named it "logo"

### there is author function call addLayer

##### lets say we need to place this logo img on a coords  35, 35

```javascript
 map.addLayer({
    'id': 'logo', // unique id 
    'type': 'symbol', // type off layer
    'source': {
        'type': 'geojson', // this way we can deal with geojson data
        'data': {
            'type': 'FeatureCollection',
          //this array would contain a list of point to list the
          // symbol on the map
            'features': [{
                'type': 'Feature',
                'geometry': {
                    'type': 'Point',
                    'coordinates': [35, 35] // coods
                }
            }]
        }
    },
    'layout': { // data type
        'icon-image': 'logo',
      // you can resize icon like .5 or 1.25
        'icon-size': 1
    }
});
```

##### at this example  we loaded image from external api and added like geojson data  with the same approach we can load geojson to the map using fetch()

from 

<https://earthquake.usgs.gov>

###### this api have query parms as following

###### format , the expected data format

###### latitude , number of the latitude in the coords

###### longitude,  number of the longitude in the coords

###### maxradius  radius to bring back all earthquake points

```javascript
let arr = []

await fetch(
    "https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&latitude=35&longitude=35&maxradius=10"
    )
    .then((res) => res.json())
    .then((result) =>
        arr = result.features)
    .catch((e) => console.log(e));
```

now  need to add this data to the map  first add source then add Layer just like we added the image before

```javascript
map.addSource('points', {
  // name it unique to this item so we are going
  // to reference it later
  // before was logo remember 😎
    'type': 'geojson',
    'data': {
        'type': 'FeatureCollection',
        'features': arr
    }
});
```

#### to understand how the geojson data structured lets look at a sample

```json
[
  {
    "type": "Feature",
    "properties": {
      "mag": 4.4,
      "place": "21 km NNW of Basse-Pointe, Martinique",
      "time": 1613091689074,
      "updated": 1613093329040,
      "tz": null,
      "url": "https://earthquake.usgs.gov/earthquakes/eventpage/us6000dguu",
      "detail": "https://earthquake.usgs.gov/fdsnws/event/1/query?eventid=us6000dguu&format=geojson",
      "felt": null,
      "cdi": null,
      "mmi": null,
      "alert": null,
      "status": "reviewed",
      "tsunami": 0,
      "sig": 298,
      "net": "us",
      "code": "6000dguu",
      "ids": ",us6000dguu,",
      "sources": ",us,",
      "types": ",origin,phase-data,",
      "nst": null,
      "dmin": 0.2,
      "rms": 0.56,
      "gap": 105,
      "magType": "mb",
      "type": "earthquake",
      "title": "M 4.4 - 21 km NNW of Basse-Pointe, Martinique"
    },
    "geometry": { "type": "Point", "coordinates": [-61.2133, 15.0367, 143.66] },
    "id": "us6000dguu"
  }
]
```

##### same like what we structure for our own image but this time with props about the earthquake  what we want here  geometry key contain the coordinates for the earthquake

#### now lets add this data on the map

```javascript
 map.addLayer({
      id: "circles", // UNIQUE ID
      type: "circle",  // type of the item will added on top of the map
      source: "points", // our pre  source  
      paint: {
          "circle-radius": {
              // make the circles larger as the user zooms from 12 to 22
              property: "mag",
              stops: [
                  [0, 4],
                  [5, 10],
              ],
          },
          "circle-color": {
              property: "mag", // mag is a property from api show earthquake power
              stops: [
                  [0, "white"], // give it color based on power
                  [3, "yellow"],
                  [5, "red"],
              ],
          },
          "circle-stroke-width": 2.2,  // the outer line of the circle
          "circle-stroke-color": "#fff", // line of the border line
      },
      filter: ["==", "$type", "Point"], // filter out any data not point
      // from the api you could receive a line or author shape
  });
```

you can find a complete example here 

<https://github.com/hazem-a1/mab-box-simple>

you should have a output like that 



![sample image for earthquake map](https://github.com/hazem-a1/mab-box-simple/blob/main/sample.png "output")



### todo next try using event with on click to show more info on the screen
