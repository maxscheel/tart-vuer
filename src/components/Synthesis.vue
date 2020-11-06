<template>
<div>
    <v-alert dense v-if="mode!='vis'" type="warning" prominent outlined>
        <div class="title">
            Operating Mode: {{mode}}
        </div>
        <div>
            Visibilities most likely outdated because the telescope is currently not operating in visibility mode.
        </div>
    </v-alert>
    <v-card :loading="!curl" class="mx-auto">
        <v-card-title class="my-0 py-1 pr-0">
            <h4 class="teal--text text--lighten-2 text-uppercase"> Realtime View </h4>
            <v-spacer />
            <slot name="enlarge"></slot>
        </v-card-title>
        <v-img aspect-ratio="1" contain :src="curl">
            <svg id='overlaySVG' viewBox="0 0 512 512">
                <GridOverlay v-if="show_grid" dims="512"></GridOverlay>
                <g v-if="show_sat">
                    <satellite v-for="sat in sat_list" v-bind:sat="sat" :key="sat.name"> </satellite>
                </g>
            </svg>
        </v-img>
        <v-card-actions class="py-0 my-0">
            <v-spacer />
            <v-switch v-model="show_sat" label="Satellites" />
            <v-spacer />
            <v-switch v-model="show_grid" label="Grid" />
            <v-spacer />
        </v-card-actions>
    </v-card>
</div>
</template>

<script>
import syn from '@/plugins/api_synthesis.js';
import Satellite from '@/components/Satellite.vue'
import GridOverlay from '@/components/GridOverlay.vue'

var cm = require("colormap")({
    colormap: 'viridis',
    nshades: 256,
    format: 'rgb',
    alpha: 0
})

export default {
    name: 'SynthesisComponent',
    components: {
        Satellite,
        GridOverlay,
    },
    data: function () {
        return {
            show_grid: true,
            show_sat: true,
            show_names: true,
            retrieving: false,
            curl: null
        }
    },
    watch: {
        vis: function () {
            this.redraw()
        },
    },
    methods: {
        redraw: function () {
            if (this.vis && this.antennas && this.gain) {

                let image = syn.gen_image(this.vis, this.antennas, this.gain, this.nw, this.num_bin)
                let virtual_canvas = document.createElement("canvas")
                let virtual_ctx = virtual_canvas.getContext("2d")
                virtual_canvas.width = this.num_bin
                virtual_canvas.height = this.num_bin
                let imgData = virtual_ctx.getImageData(0, 0, this.num_bin, this.num_bin);
                let data = imgData.data;

                for (let j = 0; j < image.shape[1]; j++) {
                    for (let i = 0; i < image.shape[0]; i++) {
                        let s = 4 * (i + j * image.shape[1]);
                        let x = cm[Math.floor(image.get(i, j))];
                        data[s] = x[0];
                        data[s + 1] = x[1];
                        data[s + 2] = x[2];
                        data[s + 3] = 255;
                    }
                }
                virtual_ctx.putImageData(imgData, 0, 0);
                this.curl = virtual_canvas.toDataURL()
            }
        },
    },
    computed: {
        mode() {
            return this.$store.state.telescope_mode
        },
        num_bin() {
            return this.$store.state.num_bin
        },
        nw() {
            return this.$store.state.nw
        },
        vis() {
            return this.$store.state.vis
        },
        gain() {
            return this.$store.state.gain
        },
        antennas() {
            return this.$store.state.antennas
        },
        sat_list() {
            return this.$store.state.sat_list
        },
        ant_sel_i() {
            return this.sel_baseline[0]
        },
        ant_sel_j() {
            return this.sel_baseline[1]
        },
    },
}
</script>

<style scoped>
#overlaySVG {
    position: absolute;
    left: 0px;
    top: 0px;
    z-index: 2;
}
</style>
