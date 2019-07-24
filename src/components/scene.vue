<template>
    <v-container v-show="testFlg">
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
    </v-container>
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
        data() {
            return {
                testFlg: false
            }
        },
        mounted() {
            this.$eventHub.$on("init", this.initMap);
        },
        methods: {
            initMap(x0, y0, z0) {
                this.testFlg = !this.testFlg;
                console.log(this.testFlg);
                loadModules(['esri/views/SceneView', 'esri/config', 'esri/WebScene'])
                    .then(([SceneView, esriConfig, WebScene]) => {
                        esriConfig.portalUrl = "https://agportalw-sec-green7.csr.utexas.edu/portal";
                        var scene = new WebScene({
                            portalItem: {
                                id: "a3bbc622780843b08eb5d13ff2d65a10"
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