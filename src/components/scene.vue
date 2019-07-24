<template>
        <v-sheet id="sheet1"
            color="grey lighten-3"
            height="600"
            width="600"
        >
            <p>x-coordinate: {{ x0 }}</p>
            <p>y-coordinate: {{ y0 }}</p>
            <p>z-coordinate: {{ z0 }}</p>
            <div id="viewDiv"></div>
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
        components: {
            // SheetFooter: {
            //     functional: true,

            //     render(h, {children}) {
            //         return h('v-sheet', {
            //             staticClass: 'mt-auto align-center justify-center d-flex',
            //             props: {
            //                 color: 'rgba(0, 0, 0, .36)',
            //                 dark: true,
            //                 height: 600,
            //                 width: 600
            //             }
            //         }, children)
            //     }
            // }
        },
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
                loadModules(['esri/Map', 'esri/views/SceneView', 'esri/layers/IntegratedMeshLayer'])
                    .then(([Map, SceneView, IntegratedMeshLayer]) => {
                        var layer = new IntegratedMeshLayer({
                            url: "https://tiles.arcgis.com/tiles/FQD0rKU8X5sAQfh8/arcgis/rest/services/VRICON_Yosemite_Sample_Integrated_Mesh_scene_layer/SceneServer"
                        });
                        var map = new Map({
                            basemap: "satellite",
                            layers: [layer],
                            ground: "world-elevation"
                        });
                        console.log("started Sceneview");
                        var view = new SceneView({
                            container: "viewDiv",
                            map: map,
                            camera: {
                                position: {
                                    x: this.x0,
                                    y: this.y0,
                                    z: this.z0,
                                    spatialReference: {
                                        wkid: 3857
                                    }
                                },
                                tilt: 84,
                                heading: 85
                            }
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
        z-index: 1008;
        /*display: none;*/
    }
    #viewDiv {
        z-index: 2000;
        width: 400px;
        height: 400px;
    }
</style>