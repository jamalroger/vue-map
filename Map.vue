<template>
  <div>
    <input
      type="text"
      class="LocationPicker__autocomplete"
      ref="input"
      placeholder="Search The place"
    />

    <div
      ref="googleMap"
      id="map_main"
      style="
        position: absolute !important;
        overflow: hidden;
        bottom: 0px;
        left: 0px;
        top: 0px;
        right: 0px;
      "
    ></div>
  </div>
</template>

<script>

let MapMarker = function (options) {
  this.position = options.pos;
  this.map = options.map;
  this.className = options.className;
  this.id = options.id;
  this.data = options.data;
  this.htmlContent = options.htmlContent;
  this.div = document.createElement("div");
  this.setMap(options.map);
};
MapMarker.prototype = new window.google.maps.OverlayView();

MapMarker.prototype.onRemove = function () {};

MapMarker.prototype.draw = function () {
  this.div.className = this.className;
  this.div.innerHTML = this.htmlContent;
  this.div.id = "marker_" + this.id;

  const panes = this.getPanes();
  panes.overlayImage.appendChild(this.div);

  let overlayProjection = this.getProjection();
  let position = overlayProjection.fromLatLngToDivPixel(this.position);
  this.div.style.left = position.x + "px";
  this.div.style.top = position.y + "px";
};

MapMarker.prototype.getPosition = function () {
  return this.position;
};

MapMarker.prototype.addEventListener = function (event, listener) {
  this.div.addEventListener(event, listener(this));
};

MapMarker.prototype.onRemove = function () {
  if (this.div) {
    this.div.parentNode.removeChild(this.div);
    delete this.div;
  }
};

export default {
  name: "maps",
  props: ["markers", "clickMarker"],

  data: () => ({
    geocoder: null,
    googleMap: null,
    autocomplete: null,
  }),
  computed: {
    google() {
      return window.google;
    },
  },
  watch: {
    markers(newMarkers) {
      this.addMarkers(newMarkers);
    },
  },
  mounted() {
    this.initMap(() => {
      this.addMarkers(this.markers);
    });
  },
  methods: {
    initMap(callback) {
      this.googleMap = new this.google.maps.Map(this.$refs.googleMap, {
        center: { lat: 33.6990826, lng: -7.3781075 },
        zoom: 14,
        mapTypeId: "hybrid",
        // mapTypeId: this.google.maps.MapTypeId.ROADMAP,
        disableDefaultUI: true,
        maxZoom: 20,
        minZoom: 4,
      });

      this.googleMap.controls[this.google.maps.ControlPosition.TOP_LEFT].push(
        this.$refs.input
      );

      this.autocomplete = new this.google.maps.places.Autocomplete(
        this.$refs.input,
        {
          types: ["geocode"],
        }
      );
      this.autocomplete.addListener("place_changed", () => {
        var place = this.autocomplete.getPlace();
        var location = place.geometry && place.geometry.location;
        if (location) {
          // this.$emit("update:position", location);
          this.googleMap.panTo(location);
        }
      });

      callback();
    },
    addMarkers(markers) {
      for (const item of markers) {
        if (item.position) {
          let marker = new MapMarker({
            id: item.id,
            pos: new this.google.maps.LatLng(
              item.position.lat,
              item.position.lng
            ),
            map: this.googleMap,
            data: {
              id: item.id,
              img: item.img,
              full_name: item.full_name,
              subjects: item.subjects || [],
              levels: item.levels || [],
              about: item.about,
              pack: item.pack || null,
            },
            className: "arrow_box",
            htmlContent: `<img src="${item.img}">`,
          });
          marker.addEventListener("click", (marker) => () => {
            this.clickMarker(marker);
          });
        }
      }
    },
  },
};
</script>
<style lang="scss">
div.arrow_box {
  width: 34px;
  height: 39px;
  font-size: u6px;
  position: absolute;
  background-image: url(/marker.png);
  background-repeat: round;
}
div.arrow_box img {
  margin-left: 8px;
  margin-top: 4px;
  max-height: 46%;
  border-radius: 50%;
}

.top-search:before {
}
</style>
