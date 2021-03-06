<!DOCTYPE html>
<html>

<head>
    <title></title>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <link href='https://api.mapbox.com/mapbox-assembly/v0.24.0/assembly.min.css' rel='stylesheet'>
    <script src='https://api.mapbox.com/mapbox-assembly/v0.24.0/assembly.js'></script>
    <script src='https://api.mapbox.com/mapbox-gl-js/v2.0.1/mapbox-gl.js'></script>
  <link href='https://api.mapbox.com/mapbox-gl-js/v2.0.1/mapbox-gl.css' rel='stylesheet' />
    <script
        src='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-geocoder/v4.2.0/mapbox-gl-geocoder.min.js'></script>
    <link rel='stylesheet'
        href='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-geocoder/v4.2.0/mapbox-gl-geocoder.css'
        type='text/css' />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/geojson/0.5.0/geojson.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@turf/turf@5/turf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/lodash@4.17.15/lodash.min.js"></script>
    <script src='https://npmcdn.com/csv2geojson@latest/csv2geojson.js'></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
    <link rel='stylesheet' href="style.css" />
</head>

<body>
    <div class='flex-parent viewport-full relative scroll-hidden'>
        <div class='flex-child w-full w360-ml absolute static-ml left bottom'>
            <div class='flex-parent flex-parent--column viewport-third bg-white'>
                <div class='flex-child flex-child--grow'>
                    <div id="sidebarA"
                        class="flex-parent flex-parent--column-ml flex-parent--center-main theme py12 px12 ">
                        <h3 id='title' class='txt-l-ml txt-m txt-bold mb6 mr0-ml mr24 align-center block'>
                        </h3>
                        <p id='description' class='txt-s py12 none block-ml'>
                        </p>
                        <div class="flex-parent flex-parent--center-main relative-ml absolute right top mt0-ml mt6">
                         
                        </div>


                    </div>
                    <div id="listings" class="flex-child viewport-twothirds py12 px12 listings scroll-auto"></div>

                </div>
            </div>
        </div>
        <div class='flex-child flex-child--grow w-auto viewport-full-ml viewport-twothirds' id='map'>

        </div>

        <div id='modal'
            class='absolute top right bottom left scroll-auto hide-visually flex-parent flex-parent--center-main mt120-ml'>
            <div class='pt36'>
                <div class='flex-child bg-white round relative scroll-auto'>
                   
              

                    </div>

                </div>
            </div>
        </div>

    </div>

    </div>
</body>





<script>
const config = {
  style: "mapbox://styles/mapbox/satellite-v9",
  accessToken:
    "pk.eyJ1Ijoic29wbGFuIiwiYSI6ImNrcmp6bHN4djBwdGcyd3J2YWtieGtuYWkifQ.gD2YzPaZzcj2VYFIPEkhLg",
  CSV: "https://docs.google.com/spreadsheets/d/1xX1Rx-8kCfaJ9oPpnsyhZVURfNhbqlK0Xj0iVxRyJS0/gviz/tq?tqx=out:csv&sheet=Sheet1",
  center: [-120.234, 47.398],
  zoom: 1,
  sideBarInfo: ["Building", "Architect", "Year"],
  popupInfo: ["Building"],
  };
  </script>

<script>

/* eslint-disable strict */
mapboxgl.accessToken = config.accessToken;
const columnHeaders = config.sideBarInfo;

let geojsonData = {};
const filteredGeojson = {
  type: "FeatureCollection",
  features: [],
};

const map = new mapboxgl.Map({
  container: "map",
  style: config.style,
  center: config.center,
  zoom: config.zoom,
  transformRequest: transformRequest,
});

function flyToLocation(currentFeature) {
  map.flyTo({
    center: currentFeature,
    zoom: 11,
  });
}

function createPopup(currentFeature) {
  const popups = document.getElementsByClassName("mapboxgl-popup");
  /** Check if there is already a popup on the map and if so, remove it */
  if (popups[0]) popups[0].remove();
  const popup = new mapboxgl.Popup({ closeOnClick: true })
    .setLngLat(currentFeature.geometry.coordinates)
    .setHTML("<h3>" + currentFeature.properties[config.popupInfo] + "</h3>")
    .addTo(map);
}

function buildLocationList(locationData) {
  /* Add a new listing section to the sidebar. */
  const listings = document.getElementById("listings");
  listings.innerHTML = "";
  locationData.features.forEach(function (location, i) {
    const prop = location.properties;

    const listing = listings.appendChild(document.createElement("div"));
    /* Assign a unique `id` to the listing. */
    listing.id = "listing-" + prop.id;

    /* Assign the `item` class to each listing for styling. */
    listing.className = "item";

    /* Add the link to the individual listing created above. */
    const link = listing.appendChild(document.createElement("button"));
    link.className = "title";
    link.id = "link-" + prop.id;
    link.innerHTML =
      '<p style="line-height: 1.25">' + prop[columnHeaders[0]] + "</p>";

    /* Add details to the individual listing. */
    const details = listing.appendChild(document.createElement("div"));
    details.className = "content";

    for (let i = 1; i < columnHeaders.length; i++) {
      const div = document.createElement("div");
      div.innerText += prop[columnHeaders[i]];
      div.className;
      details.appendChild(div);
    }

    link.addEventListener("click", function () {
      const clickedListing = location.geometry.coordinates;
      flyToLocation(clickedListing);
      createPopup(location);

      const activeItem = document.getElementsByClassName("active");
      if (activeItem[0]) {
        activeItem[0].classList.remove("active");
      }
      this.parentNode.classList.add("active");

      const divList = document.querySelectorAll(".content");
      const divCount = divList.length;
      for (i = 0; i < divCount; i++) {
        divList[i].style.maxHeight = null;
      }

      for (let i = 0; i < geojsonData.features.length; i++) {
        this.parentNode.classList.remove("active");
        this.classList.toggle("active");
        const content = this.nextElementSibling;
        if (content.style.maxHeight) {
          content.style.maxHeight = null;
        } else {
          content.style.maxHeight = content.scrollHeight + "px";
        }
      }
    });
  });
}


const geocoder = new MapboxGeocoder({
  accessToken: mapboxgl.accessToken, // Set the access token
  mapboxgl: mapboxgl, // Set the mapbox-gl instance
  marker: true, // Use the geocoder's default marker style
  zoom: 11,
});

function sortByDistance(selectedPoint) {
  const options = { units: "miles" };
  if (filteredGeojson.features.length > 0) {
    var data = filteredGeojson;
  } else {
    var data = geojsonData;
  }
  data.features.forEach(function (data) {
    Object.defineProperty(data.properties, "distance", {
      value: turf.distance(selectedPoint, data.geometry, options),
      writable: true,
      enumerable: true,
      configurable: true,
    });
  });

  data.features.sort(function (a, b) {
    if (a.properties.distance > b.properties.distance) {
      return 1;
    }
    if (a.properties.distance < b.properties.distance) {
      return -1;
    }
    return 0; // a must be equal to b
  });
  const listings = document.getElementById("listings");
  while (listings.firstChild) {
    listings.removeChild(listings.firstChild);
  }
  buildLocationList(data);
}

geocoder.on("result", function (ev) {
  const searchResult = ev.result.geometry;
  sortByDistance(searchResult);
});

map.on("load", function () {
  map.addControl(geocoder, "top-right");

  // csv2geojson - following the Sheet Mapper tutorial https://www.mapbox.com/impact-tools/sheet-mapper
  console.log("loaded");
  $(document).ready(function () {
    console.log("ready");
    $.ajax({
      type: "GET",
      url: config.CSV,
      dataType: "text",
      success: function (csvData) {
        makeGeoJSON(csvData);
      },
      error: function (request, status, error) {
        console.log(request);
        console.log(status);
        console.log(error);
      },
    });
  });

  function makeGeoJSON(csvData) {
    csv2geojson.csv2geojson(
      csvData,
      {
        latfield: "Latitude",
        lonfield: "Longitude",
        delimiter: ",",
      },
      function (err, data) {
        data.features.forEach(function (data, i) {
          data.properties.id = i;
        });

        geojsonData = data;
        // Add the the layer to the map
        map.addLayer({
          id: "locationData",
          type: "circle",
          source: {
            type: "geojson",
            data: geojsonData,
          },
          paint: {
            "circle-radius": 5, // size of circles
            "circle-color": "#3D2E5D", // color of circles
            "circle-stroke-color": "white",
            "circle-stroke-width": 1,
            "circle-opacity": 0.7,
          },
        });
      }
    );

    map.on("click", "locationData", function (e) {
      const features = map.queryRenderedFeatures(e.point, {
        layers: ["locationData"],
      });
      const clickedPoint = features[0].geometry.coordinates;
      flyToLocation(clickedPoint);
      sortByDistance(clickedPoint);
      createPopup(features[0]);
    });

    map.on("mouseenter", "locationData", function () {
      map.getCanvas().style.cursor = "pointer";
    });

    map.on("mouseleave", "locationData", function () {
      map.getCanvas().style.cursor = "";
    });
    buildLocationList(geojsonData);
  }
});

// Modal - popup for filtering results
const filterResults = document.getElementById("filterResults");
const exitButton = document.getElementById("exitButton");
const modal = document.getElementById("modal");

filterResults.addEventListener("click", () => {
  modal.classList.remove("hide-visually");
  modal.classList.add("z5");
});

exitButton.addEventListener("click", () => {
  modal.classList.add("hide-visually");
});

const title = document.getElementById("title");
title.innerText = config.title;
const description = document.getElementById("description");
description.innerText = config.description;

function transformRequest(url, resourceType) {
  var isMapboxRequest =
    url.slice(8, 22) === "api.mapbox.com" ||
    url.slice(10, 26) === "tiles.mapbox.com";
  return {
    url: isMapboxRequest ? url.replace("?", "?pluginName=finder&") : url,
  };
}
</script>

<style>
#sidebarA{
    color: white;
}

#filterResults{
    font-size: 12px;
    color: white;
    background-color: #448EE4;

}

.theme{
    background-color: #2A395F;
}

.select{
    font-style: italic;
    opacity: .7;
}

.mapboxgl-ctrl-geocoder--input {
    padding: 6px 30px;
    margin-left: 5px;
}

.listings .item {
    font-size: 14px;
    word-wrap: break-word;
    text-decoration: none;
    border-bottom: solid rgb(34, 33, 33, .3) 1px;
}

.listings .item:last-child {
    border-bottom: none;
}

.listings .item .title {
    display: inline-block;
    width: 100%;
    color: #2A395F;
    font-weight: 800;
    margin-top: 5px;
    text-decoration: none;
}


::-webkit-scrollbar {
    width: 3px;
    height: 3px;
    border-left: 0;
    background: rgba(0, 0, 0, 0.1);
}

::-webkit-scrollbar-track {
    background: none;
}

::-webkit-scrollbar-thumb {
    border-radius: 0;
    background: #2A395F;
}


.clearfix {
    display: block;
}

.clearfix::after {
    content: '.';
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
}

.mapboxgl-popup-content {
    font: 400 15px/22px 'Source Sans Pro', 'Helvetica Neue', Sans-serif;
    text-align: center;
    padding: 5;
    width: 180px;
  }

.mapboxgl-popup-close-button {
    display: none;
  }

.mapboxgl-popup-content h3 {
    color:black;
    font-weight: 500;
    margin: 0;
    display: block;

  }



</style>



</html>
