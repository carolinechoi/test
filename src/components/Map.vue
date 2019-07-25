<template>
    <div id="mapTexas"><div id="extentDiv" v-show="showingFlg"></div></div>
</template>
<script>
    import L from "leaflet";
    import * as esri from "esri-leaflet";
    import * as wmts from "leaflet-tilelayer-wmts";
    //import axios from 'axios';
    //import * as MakiMarkers from './scripts/Leaflet.MakiMarkers';
    import './scripts/utility';
    let movesMap;
    let esriAerialsLayer;
    let esriAerialsLabels;
    let esriToposLayer;
    let esriStreetsLayer;
    let googleImageryLayer;
    //let geoJsonQPF_Day1;
    //let geoJsonQPF_Day2;
    let maskingSentinelApr16;
    let esriSnowLayer;
    let popDenseLayer;

    export default {
        name: "Map",
        // components: {
        //     DialogDrag
        // },
        data() {
            return {
                map: [],
                markers: null,
                txCapQueryTotalRecordsCount:null,
                txCapQueryCurrentDate:null,
                txCapQuerySpatial:null,
                showingFlg: false
            }
        },
        methods:{
            toggleLayer: toggleLayer,
            resetMap,
            redrawMap,
            goToBookmark,
            goToLocation,
            getMapInfo,
            changeLayer,
            // extentDiv() {
            //     this.showingFlg = !this.showingFlg;
            //     console.log(this.showingFlg);
            // }
        },
        mounted(){
            // this.$eventHub.$on("extentDiv-created", this.extentDiv);
            esriToposLayer = esri.basemapLayer("Topographic");
            esriAerialsLayer = esri.basemapLayer('Imagery',{attribution: "ESRI et al",hideLogo:"true"});
            esriAerialsLabels = esri.basemapLayer('ImageryLabels');
            esriStreetsLayer = esri.basemapLayer('Streets',{attribution: "ESRI et al",hideLogo:"true"});
            googleImageryLayer = wmts.tileLayerWMTS('https://txgi.tnris.org/login/path/arena-baker-mouse-bonus/wmts?',

                {
                    layer: 'texas',
                    style: "normal",
                    tilematrixSet: "0to20",
                    maxZoom: 20,
                    transparent: true,
                    format: "image/png"
                }
            );
            movesMap = L.map("mapTexas", {
                center: [38.3117, -98.77774],
                zoom: 5,
                layers: [
                    esriToposLayer
                ]
            });
            movesMap.on('moveend', function(ev) {
                console.log(this.getBounds());
                this.$eventHub.$emit('latlng');
                console.log("dklsjfalsdfj");
            });
            esriSnowLayer = esri.featureLayer({
                url: 'http://cumulus.tnc.org/arcgis/rest/services/Atlas/FreshwaterMaps/MapServer/1',
                simplifyFactor: .5,
                precision: 10,
                //may need to use a different property in order to fill polygons correctly
                style: function (feature) {
                    if(feature.properties.Snow_map === 'earlier by over 3 weeks')
                        return {fillColor: "#003F8C", fillOpacity: '1.0'};//, fill_opacity: .99};//, weight: 2, opacity: 255
                    else if (feature.properties.Snow_map === 'earlier by 2 to 3 weeks')
                        return {fillColor: '#4585C4', fillOpacity: '1.0'};

                    else if(feature.properties.Snow_map === 'earlier by 1 to 2 weeks')
                        return {fillColor: '#7DB5FA', fillOpacity: '1.0'};
                    else if(feature.properties.Snow_map === 'less than a week')
                        return {fillColor: '#BED2FF', fillOpacity: '1.0'};
                    else if(feature.properties.Snow_map === 'later by over a week')
                        return {fillColor: '#BED2FF', fillOpacity: '1.0'};
                    else if(feature.properties.Snow_map === 'other')
                        return {fillColor: '#AAAAAA', fillOpacity: '1.0'};
                    else
                        return {fillColor: '#E1E1E1'};


                }

            }).addTo(movesMap);

            movesMap.removeLayer(esriSnowLayer);
        },
        created(){
            //Menu Components used to pass over Global Event hub these events from buttons
            this.$eventHub.$on('toggleMapLayers', this.toggleLayer);
            this.$eventHub.$on('resetMap',this.resetMap);
            this.$eventHub.$on('goToBookmarks',this.goToBookmark);
            this.$eventHub.$on('goToLocation',this.goToLocation);
            //this.$eventHub.$on('goToLocation',this.goToNamedLocation);
            this.$eventHub.$on('gatheringMapInfo',this.getMapInfo);
            this.$eventHub.$on('redrawMap',this.redrawMap);
            //toggleMaskTornado20190416
            //this.$eventHub.$on('toggleMaskTornado20190416', this.addRemoveMaskTornado);
            this.$eventHub.$on('toggleLayer', this.changeLayer); //add function to call
            // this.$eventHub.$on('layerOff', this.layerOff); // add function to call
        }
    }
    function changeLayer(layer) {
        console.log("message recieved and layer on function started");
        if(layer[2]){
            movesMap.removeLayer(getLayer(layer[3]));
        }
        else{
            movesMap.addLayer(getLayer(layer[3]));
        }

        //document.getElementById(layer).visibility = "visible";
        //not working
    }
    function getLayer(layerName){
        console.log("layaerArr function started");
        if(layerName=="esriSnowLayer"){
            return esriSnowLayer;
        }
        else{
            return null;
        }
    }

    function resetMap(){
        movesMap.setView([38.3117, -98.77774], 5);
        movesMap.invalidateSize();
    }
    function redrawMap(){
        console.log('Here');
        movesMap.invalidateSize();
    }
    function getMapInfo(){
        const LatLngAry = [movesMap.getCenter().lat,movesMap.getCenter().lng];
        let mapInfoObj = {ZoomLevel:movesMap.getZoom(),LatLng:LatLngAry};
        //movesMap.setView([32.3117, -99.77774], 6);
        //TODO:Convert to Vuex BAP 04-03-19
        this.$eventHub.$emit('returnedMapInfo',mapInfoObj);
    }
    function goToLocation(incomingObj){
        console.log("Go To Location");
        console.log(incomingObj);
        console.log(incomingObj.LatLng);
        const LatLngAry = [incomingObj.LatLng[0],incomingObj.LatLng[1]];
        console.log(LatLngAry);
        movesMap.setView(LatLngAry,incomingObj.ZoomLevel);
        movesMap.invalidateSize();
    }
    function goToBookmark(incomingBookmark) {
        if (incomingBookmark === 'California') {
            movesMap.setView([36.7783, -119.4179], 6);
        } else if (incomingBookmark === 'Florida') {
            movesMap.setView([27.6648, -81.5158], 6);
        } else if (incomingBookmark === 'Texas') {
            movesMap.setView([32.3117, -99.77774], 6);
        } else {
            movesMap.setView([38.3117, -98.77774], 5);
        }
    }

    function toggleLayer(incomingLayer) {
        //console.log(incomingLayer);
        //console.log(esriToposLayer);
        if (incomingLayer === 'topo') {
            console.log(incomingLayer);
            if (!movesMap.hasLayer(esriToposLayer)) {
                movesMap.addLayer(esriToposLayer);
            }
            checkRemoveAerials();
            checkRemoveStreets();
            checkRemoveGoogleWMTS();
        } else if (incomingLayer === 'aerials') {
            if (movesMap.hasLayer(esriAerialsLayer)) {
                if (!movesMap.hasLayer(esriToposLayer)) {
                    movesMap.addLayer(esriToposLayer);
                }
                checkRemoveStreets();
                checkRemoveAerials();
                checkRemoveGoogleWMTS();
            } else {
                checkRemoveGoogleWMTS();
                movesMap.addLayer(esriAerialsLayer);
                movesMap.addLayer(esriAerialsLabels);
                checkRemoveTopo();
                checkRemoveStreets();
            }
        } else if (incomingLayer === 'streets') {
            if (movesMap.hasLayer(esriStreetsLayer)) {
                if (!movesMap.hasLayer(esriToposLayer)) {
                    movesMap.addLayer(esriToposLayer);
                }
                checkRemoveStreets();
                checkRemoveGoogleWMTS();
                checkRemoveAerials();
            } else {
                movesMap.addLayer(esriStreetsLayer);
                checkRemoveGoogleWMTS();
                checkRemoveTopo();
                checkRemoveAerials();
            }
        } else if (incomingLayer === 'reset') {
            //console.log('Reset');
            if (!movesMap.hasLayer(esriToposLayer)) {
                movesMap.addLayer(esriToposLayer);
                checkRemoveAerials();
                checkRemoveStreets();
                checkRemoveGoogleWMTS();
            }
            movesMap.setView([32.3117, -99.77774], 6);
        } else if (incomingLayer === 'google') {
            if (movesMap.hasLayer(googleImageryLayer)) {
                movesMap.removeLayer(googleImageryLayer);
                checkRemoveLabelsOnly();
                checkRemoveAerials();
                if (!movesMap.hasLayer(esriToposLayer)) {
                    movesMap.addLayer(esriToposLayer);
                }
                //console.log("Am I here?");
            } else {
                if (!movesMap.hasLayer(esriToposLayer)) {
                    movesMap.addLayer(esriToposLayer);
                }
                movesMap.addLayer(googleImageryLayer);
                checkRemoveAerials();
                checkRemoveStreets();
                if (!movesMap.hasLayer(esriAerialsLabels)) {
                    movesMap.addLayer(esriAerialsLabels);
                    //movesMap.bringToFront(esriAerialsLabels);
                }
            }
        }
    }

    function checkRemoveLabelsOnly() {
        if (movesMap.hasLayer(esriAerialsLabels)) {
            movesMap.removeLayer(esriAerialsLabels);
        }
    }

    function checkRemoveAerials() {
        if (movesMap.hasLayer(esriAerialsLayer)) {
            movesMap.removeLayer(esriAerialsLayer);
            movesMap.removeLayer(esriAerialsLabels);
        }
    }

    function checkRemoveStreets() {
        if (movesMap.hasLayer(esriStreetsLayer)) {
            movesMap.removeLayer(esriStreetsLayer);
        }
    }

    function checkRemoveTopo() {
        if (movesMap.hasLayer(esriToposLayer)) {
            movesMap.removeLayer(esriToposLayer);
        }
    }

    function checkRemoveGoogleWMTS() {
        if (movesMap.hasLayer(googleImageryLayer)) {
            movesMap.removeLayer(googleImageryLayer);
        }
        checkRemoveLabelsOnly();

    }

</script>
<style>
    #mapTexas{
        width: 100%;
        height: inherit;
        position: absolute;
        top: 30px;
        right: 0px;
    }
    .leaflet-left .leaflet-control {
        margin-left: 100px !important; /*3750%*/
    }

    .leaflet-top .leaflet-control {
        margin-top: 60px !important;
    }




</style>
