<template lang="html">
  <div class="layer-selection">
    <b-dropdown
      id="map-dropdown"
      ref="mapDropdown"
      class="tweaked-dropdown"
      :text="currMapName"
    >
      <b-dropdown-form v-on:submit.prevent="onInputFilterSubmit">
        <b-form-group label="" label-for="map-filter">
          <b-form-input
            id="map-filter"
            size="sm"
            ref="filterMapInputRef"
            v-on:input="filterMap"
            v-model="filterMapInputValue"
            v-on:keyup.stop="onKeyup"
          />
        </b-form-group>
        <b-dropdown-item v-for="map in mapNames" v-on:click="selectMap(map)">
          {{ map }}
        </b-dropdown-item>
      </b-dropdown-form>
    </b-dropdown>
    <b-dropdown
      ref="layerDropdown"
      id="layer-dropdown"
      class="tweaked-dropdown"
      :text="currLayerName"
    >
      <b-dropdown-item
        v-for="layer in Object.keys(map.raasData[currMapName])"
        v-on:click="selectLayer(layer)"
      >
        {{ layer }}
      </b-dropdown-item>
    </b-dropdown>
  </div>
</template>

<script>
import Vue from "vue";
import { BootstrapVue, IconsPlugin } from "bootstrap-vue";
import "bootstrap/dist/css/bootstrap";
import "bootstrap-vue/dist/bootstrap-vue";
import { isSmallTouchDevice } from "./utils";

Vue.use(BootstrapVue);
Vue.use(IconsPlugin);

export default Vue.extend({
  props: {
    map: Object,
    startingMapName: String,
    startingLayerName: String,
  },
  created() {
    this.mapNames = Object.keys(this.map.raasData);

    // read map and layer from URL hash part if possible
    this.selectMapAndLayerFromUrl()

    // listen to back and forwards button
    window.onpopstate = (event) => {
      this.selectMapAndLayerFromUrl()
    };
  },
  data() {
    return {
      filterMapInputValue: "",
      currMapName: null,
      currLayerName: null,
      mapNames: null,
      currMapNameInput: "",
    };
  },
  mounted() {
    this.$root.$on("bv::dropdown::shown", (bvEvent) => {
      this.filterMapInputValue = "";
      this.filterMap();
      if (!isSmallTouchDevice()) {
        var input = this.$refs.filterMapInputRef;
        setTimeout(() => input.$el.focus(), 10);
      }
    });

    this.keyListener = (e) => {
      switch (e.key) {
        case "m":
          this.$refs.mapDropdown.show();
          break;
        case "l":
          this.$refs.layerDropdown.show();
          break;
      }
    };
    document.addEventListener("keyup", this.keyListener);
    this.changeMapAndLayer(this.currMapName, this.currLayerName);
  },
  beforeDestroy() {
    document.removeEventListener("keyup", this.keylistener);
  },
  methods: {
    selectMap(map) {
      this.changeMapAndLayer(map, null);
    },
    selectLayer(layer) {
      this.changeMapAndLayer(this.currMapName, layer);
    },
    filterMap() {
      this.mapNames = Object.keys(this.map.raasData).filter((mapName) => {
        const normalized = mapName.toLowerCase().replace(" ", "");
        const normalizedFilter = this.filterMapInputValue
          .toLowerCase()
          .replace(" ", "");
        return normalized.includes(normalizedFilter);
      });
    },
    onInputFilterSubmit(e) {
      if (this.mapNames.length > 0) {
        this.selectMap(this.mapNames[0]);
        this.$refs.mapDropdown.hide(true);
      }
    },
    onKeyup(e) {},
    changeMapAndLayer(map, layer) {
      this.currMapName = map;
      this.currMapNameInput = this.currMapName;

      if (!layer) {
        this.currLayerName = Object.keys(
          this.map.raasData[this.currMapName]
        )[0];
      } else {
        this.currLayerName = layer;
      }

      var urlHashParams = new URLSearchParams(location.hash.substr(1));
      urlHashParams.set("map", this.currMapName);
      urlHashParams.set("layer", this.currLayerName);
      location.hash = urlHashParams.toString();
      this.map.changeMap(this.currMapName, this.currLayerName);
    },
    selectMapAndLayerFromUrl() {
      var urlHashParams = new URLSearchParams(location.hash.substr(1));
      this.changeMapAndLayer(
        urlHashParams.get("map") || this.startingMapName,
        urlHashParams.get("layer") || this.startingLayerName
      );
    }
  },
});
</script>

<style lang="scss">
.layer-selection {
  display: flex;
  flex-wrap: nowrap;
  max-width: 600px;
}

.layer-selection > :first-child {
  margin-right: 10px;
}

.dropdown-filter {
  padding: 0.25em;
}

#map-datalist-input {
  flex-grow: 1.5;
}

#layer-dropdown {
  flex-grow: 0.2;
}

.dropdown-menu {
  max-height: 100vh;
  overflow-y: auto;
}
</style>
