<template>
        <v-sheet id="sheet1"
            color="grey lighten-3"
            height="600"
            width="600"
        >
            <p>x-coordinate: {{ x0 }}</p>
            <p>y-coordinate: {{ y0 }}</p>
            <p>z-coordinate: {{ z0 }}</p>
            <div id="viewDiv" class="esri-widget"></div>
        </v-sheet>
</template>

<script>
    // downloaded the following from https://github.com/Esri/esri-loader
    import { loadModules } from 'esri-loader';
    import L from 'leaflet';

    let x0;
    let y0;
    let z0;

    export default {
        name: "Scene",
        props: {
            x0: Number,
            y0: Number,
            z0: Number
        },
        mounted() {
            this.$eventHub.$on("init", this.initMap);
        },
        methods: {
            initMap(x0, y0, z0) {
                console.log("started initMap() with values:" + this.x0 + this.y0);
                loadModules(['esri/config', 'esri/views/SceneView', 'esri/WebScene'])
                    .then(([SceneView, esriConfig, WebScene]) => {
                        // esriConfig.portalUrl = "https://agportalw-sec-green7.csr.utexas.edu/portal";
                        var scene = new WebScene({
                            portalItem: {
                                id: "3a9976baef9240ab8645ee25c7e9c096"
                            }
                        });
                        console.log("started Sceneview");
                        var view = new SceneView({
                            map: scene,
                            container: "viewDiv"
                        });
                        console.log("built Sceneview");
                    }).catch(e => {
                    console.log(e);
                });
            }
        }
    }
</script>

<style>
    #sheet1{
        position: absolute;
        bottom: 3%;
        right: 2%;
        z-index: 1010;
        /*display: none;*/
    }
    #viewDiv {
        z-index: 2000;
        width: 400px;
        height: 400px;
    }
</style>