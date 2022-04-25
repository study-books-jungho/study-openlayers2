<script>
    import {Map, Overlay, View} from "ol";
    import TileLayer from "ol/layer/Tile.js";
    import {fromLonLat} from "ol/proj.js";
    import {OSM, XYZ} from "ol/source.js";
    import {Vector as VectorHaHa} from "ol/source.js";
    import {onMount} from "svelte";
    import {Tile, Vector} from "ol/layer";
    import {GeoJSON} from "ol/format";
    import {Style, Circle, Fill, Stroke, Text} from "ol/style";


    export let name;

    onMount(() => {
        let center = [15091875.539375868, -2890099.0297847847];
        const map = new Map({
            view: new View({
                center: center,
                zoom: 1,
                extent: [8700313.865909653, -9512187.262171041, 21430987.92760313, 2627132.2147866394],
            }),
            layers: [
                new Tile({
                    source: new OSM()
                })
            ],
            target: 'openlayers-map'
        })

        const aussieCitiesStyle = function (feature) {
            let cityID = feature.get('ID');
            let cityIDString = cityID.toString();
            const styles = [
                new Style({
                    image: new Circle({
                        fill: new Fill({
                            color: [77, 219, 105, 0.6]
                        }),
                        stroke: new Stroke({
                            color: [6, 125, 31, 1],
                            width: 2
                        }),
                        radius: 12
                    }),
                    text: new Text({
                        text: cityIDString,
                        scale: 1.5,
                        fill: new Fill({
                            color: [232, 26, 26, 1]
                        }),
                        stroke: new Stroke({
                            color: [232, 26, 26, 1],
                            width: 0.3
                        })
                    })
                })
            ]
            return styles
        }

        const styleForSelect = function (feature) {
            let cityID = feature.get('ID');
            let cityIDString = cityID.toString();
            const styles = [
                new Style({
                    image: new Circle({
                        fill: new Fill({
                            color: [247, 26, 10, 0.5]
                        }),
                        stroke: new Stroke({
                            color: [6, 125, 34, 1],
                            width: 2
                        }),
                        radius: 12
                    }),
                    text: new Text({
                        text: cityIDString,
                        scale: 1.5,
                        fill: new Fill({
                            color: [87, 9, 9, 1]
                        }),
                        stroke: new Stroke({
                            color: [87, 9, 9, 1],
                            width: 0.5
                        })
                    })
                })
            ]
            return styles
        }


        const austCitiesLayer = new Vector({
            source: new VectorHaHa({
                format: new GeoJSON(),
                url: '/data/map.geojson'
            }),
            style: aussieCitiesStyle
        })
        map.addLayer(austCitiesLayer)

        // Map Features Click Logic
        const navElements = document.querySelector('.column-navigation');
        const cityNameElement = document.getElementById('cityname')
        const cityImageElement = document.getElementById('cityimage')
        const mapView = map.getView()

        map.on('singleclick', function (evt) {
            console.log("evt.coordinate = ", evt.coordinate)
            map.forEachFeatureAtPixel(evt.pixel, function (feature, layer) {
                let featureName = feature.get("CityName");
                console.log(featureName)
                let navElement = navElements.children.namedItem(featureName);
                console.log("navElement = ", navElement)
                mainLogic(feature, navElement)
            })
        })

        function mainLogic(feature, clickedAnchorElement) {
            // Re-assign active class to the clicked element
            let currentActiveStyledElement = document.getElementsByClassName('active')[0]
            // fixme : class add and remove is not working
            currentActiveStyledElement.classList.remove("active");
            clickedAnchorElement.classList.add("active");

            // Default style for all features
            let aussieCitiesFeatures = austCitiesLayer.getSource().getFeatures();
            aussieCitiesFeatures.forEach(function (feature) {
                feature.setStyle(aussieCitiesStyle)
            })

            // Home Element : Change content in the menu to HOME
            if (clickedAnchorElement.id === 'Home') {
                console.log(clickedAnchorElement)
                mapView.animate({center: center}, {zoom:4} )
                cityNameElement.innerHTML = "Welcome to Australian Capital Cities Tour Map"
                cityImageElement.setAttribute('src', '/data/city_image/Adel_image.jpeg')
            }
            // Change view, and content in the menu based on the feature
            else {
                feature.setStyle(styleForSelect)
                let featureCoordinates = feature.get('geometry').getCoordinates();
                mapView.animate({center: featureCoordinates}, {zoom: 5})
                let featureName = feature.get('CityName');
                let featureImage = feature.get('CityImage')

                cityNameElement.innerHTML = 'Name of the city : ' + featureName
                cityImageElement.setAttribute('src', '/data/city_image/' + featureImage + '.jpeg')
            }
        }

        // Navigation logic
        const anchorNavElements = document.querySelectorAll(".column-navigation > a")
        for (const anchorNavElement of anchorNavElements) {
            anchorNavElement.addEventListener("click", function (e) {
                let clickedAnchorElement = e.currentTarget
                let clickedAnchorElementID = e.currentTarget.id
                let aussieCitiesFeatures = austCitiesLayer.getSource().getFeatures();
                aussieCitiesFeatures.forEach(function (feature) {
                    let featureCityName = feature.get('CityName')
                    if (clickedAnchorElementID === featureCityName) {
                        mainLogic(feature, clickedAnchorElement);
                    }
                })

                // Home Navigation Case
                if (clickedAnchorElementID === 'Home') {
                    mainLogic(undefined, clickedAnchorElement)
                }
            });
        }
        // Feature Hover Logic
        const popOverTextElement = document.getElementById('popover-text');
        const popOverTextLayer = new Overlay({
            element : popOverTextElement,
            positioning  : 'bottom-center',
            stopEvent : false
        });

        map.addOverlay(popOverTextLayer)

        map.on('pointermove', function (evt) {
            let isFeatureAtPixel  = map.hasFeatureAtPixel(evt.pixel)
            if (isFeatureAtPixel) {
                let featureAtPixel = map.getFeaturesAtPixel(evt.pixel);
                let featureName = featureAtPixel[0].get("CityName");
                popOverTextLayer.setPosition(evt.coordinate);
                popOverTextElement.innerHTML = featureName
              map.getViewport().style.cursor = "pointer"
            } else {
                popOverTextLayer.setPosition(undefined)
                map.getViewport().style.cursor = ""
            }
        })
    });




</script>


<div class="grid-container">
    <div class="grid-1">
        <div class="column-navigation">
            <a title='Home' id='Home' class='active'><i class="fas fa-home" id='Home'></i></a>
            <a title='Perth' id='Perth'><i class="far fa-circle" id='Perth'></i></a>
            <a title='Adelaide' id='Adelaide'><i class="far fa-circle" id='Adelaide'></i></a>
            <a title='Melbourne' id='Melbourne'><i class="far fa-circle" id='Melbourne'></i></a>
            <a title='Brisbane' id='Brisbane'><i class="far fa-circle" id='Brisbane'></i></a>
            <a title='Darwin' id='Darwin'><i class="far fa-circle" id='Darwin'></i></a>
            <a title='Hobart' id='Hobart'><i class="far fa-circle" id='Hobart'></i></a>
            <a title='Sydney' id='Sydney'><i class="far fa-circle" id='Sydney'></i></a>
        </div>
    </div>

    <!--    <div class="grid-2">-->
    <div class="grid-2" id='openlayers-map'></div>
    <!--    </div>-->

    <div class="grid-3">
        <div class="column-menu" id='menu-information'>
            <p id='cityname'>Welcome to Australian Capital Cities Tour Map</p>
            <img id='cityimage' src="/data/city_image/Adel_image.jpeg" alt="Adel_image">
        </div>
    </div>
</div>

<div class="popover-information">
    <p id="popover-text"></p>
</div>

<style>

    * {
        box-sizing: border-box
    }

    body {
        padding: 0;
        margin: 0;
        overflow: auto;
    }

    .grid-container {
        display: grid;
        grid-template-columns: 4vw 50vw 46vw;
        grid-template-rows: 100vh
    }

    /* Columns */
    .grid-1 {
        height: 100%;
        width: 100%;
        background-color: #555
    }

    .grid-2 {
        height: 100%;
        width: 100%
    }

    .grid-3 {
        height: 100%;
        width: 100%
    }

    .column-navigation a {
        display: block;
        text-align: center;
        padding: 10px;
        color: white;
        font-size: 15px;
    }

    .column-navigation a:hover {
        background-color: #000
    }

    .active {
        background-color: #4CAF50
    }

    .column-menu {
        height: 100%;
        width: 100%;
        padding: 1px;
        background-color: #f1f1f1
    }


    /* IMAGE */

    #cityname {
        margin: 0;
        padding: 40px;
        width: 100%;
        height: 20%;
        text-align: center;
        font-size: 28px;
    }

    #cityimage {
        width: 100%;
        height: 60%;
        border-radius: 10px;
    }

    #popover-text{
        background-color : #9c9a9a;
        font-size : 18px;
    }
</style>