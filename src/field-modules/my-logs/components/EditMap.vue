<template>
  <Map
    id="map"
    :overrideStyles="{ height: 'calc(100vh - 3rem)' }"
    @update-wkt="updateMovement"
    :drawing="true"
    :options="{
      controls: (defs) => defs.filter(def => def.constructor.name !== 'FullScreen'),
      units: systemOfMeasurement,
    }"
    :wkt=mapLayers
    :geojson="{
      title: 'areas',
      url: areaGeoJSON,
      color: 'grey',
    }"/>
</template>

<script>
import Map from '@/components/Map';
import { mergeGeometries } from '@/utils/geometry';

export default {
  name: 'EditMap',
  components: { Map },
  props: ['logs',
    'assets',
    'id',
    'systemOfMeasurement'],

  computed: {
    areaGeoJSON() {
      return (process.env.NODE_ENV === 'development')
        ? 'http://localhost:8080/farm/areas/geojson/all'
        : `${localStorage.getItem('host')}/farm/areas/geojson/all`;
    },
    currentLog() {
      return this.logs.find(log => log.localID === +this.id) || this.logs[0];
    },
    /*
    Assemble layers for display.
    The 'previous' layer is assembled from the geofield plus
    all area geometires associated with the log.
    The 'movement' layer is the geometry in the log's movement field
    */
    mapLayers() {
      const movement = {
        title: 'movement',
        wkt: this.currentLog.movement?.geometry,
        color: 'orange',
        visible: true,
        weight: 0,
        canEdit: true,
      };
      const previousGeoms = this.currentLog.asset
        ?.map(logAsset => this.assets
          ?.find(asset => asset.id === logAsset.id)?.geometry);
      const previousWKT = mergeGeometries(previousGeoms);
      const previous = {
        title: 'previous',
        wkt: previousWKT,
        color: 'green',
        visible: true,
        weight: 1,
        canEdit: false,
      };
      return [previous, movement];
    },
  },

  methods: {
    updateMovement(wkt) {
      const props = {
        movement: {
          area: this.currentLog.movement?.area,
          geometry: wkt,
        },
        localID: +this.id,
      };
      this.$store.dispatch('updateLog', props);
    },
  },
};
</script>

<style scoped>

</style>
