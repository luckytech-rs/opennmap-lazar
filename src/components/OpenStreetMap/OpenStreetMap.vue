<template>
  <div>
    <div ref="map" class="map"></div>
    <div
      id="popup"
      class="ol-popup"
      @mouseenter="onPopupMouseEnter"
      v-click-outside="hidePopup"
      :class="{
        'popup-above': showPopupAbove,
        'popup-left': showPopupLeft,
        'popup-right': showPopupRight,
      }"
    >
      <button class="ol-popup-closer" @click="hidePopup">X</button>
      <div v-if="markerDetails">
        <div class="popup-images">
          <ImageSlider :images="markerDetails.pictures" />
        </div>
        <div class="popup-content">
          <h2>{{ markerDetails.title }}</h2>
          <p class="multiline-ellipsis">{{ markerDetails.description }}</p>
          <p class="price">
            {{ formatCurrency(markerDetails.price) }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "ol/ol.css";
import { Map, View } from "ol";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import VectorLayer from "ol/layer/Vector";
import { Vector } from "ol/source";
import Feature from "ol/Feature";
import Point from "ol/geom/Point";
import { fromLonLat } from "ol/proj";
import axios from "axios";
import { Style, Fill, Text, Stroke, RegularShape } from "ol/style";
import Overlay from "ol/Overlay";

import vClickOutside from "click-outside-vue3";

import ImageSlider from "@/components/ImageSlider/ImageSlider.vue";

export default {
  data() {
    return {
      map: null,
      markerDetails: null,
      showPopupAbove: false,
      showPopupLeft: false,
      showPopupRight: false,
      popupOverlay: null,
      isMouseOverPopup: false,
      currency: "CHF",
    };
  },
  components: {
    ImageSlider,
  },
  directives: {
    clickOutside: vClickOutside.directive,
  },
  mounted() {
    this.map = new Map({
      target: this.$refs.map,
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
      ],
      view: new View({
        center: [0, 0],
        zoom: 2,
      }),
    });

    this.popupOverlay = new Overlay({
      element: document.getElementById("popup"),
      positioning: "bottom-center",
      offset: [0, -10],
      autoPan: false,
    });

    this.map.addOverlay(this.popupOverlay);

    window.addEventListener("resize", () => {
      this.map.updateSize();
    });

    axios
      .get("https://blattcodeservices.com/pct/search")
      .then((response) => {
        const data = response.data;

        const vectorSource = new Vector({
          features: data.publications.map((item) => {
            const coordinates = fromLonLat([
              item.address.coordinates.longitude,
              item.address.coordinates.latitude,
            ]);
            return new Feature({
              geometry: new Point(coordinates),
              title: item.publicationTitle,
              description: item.description,
              pictures: item.pictures,
              price: item.price,
              width: item.price.toString().length * 10,
              height: 20,
            });
          }),
        });
        const vectorLayer = new VectorLayer({
          source: vectorSource,
          style: (feature) => {
            return new Style({
              image: new RegularShape({
                fill: new Fill({
                  color: "#000000",
                }),
                stroke: new Stroke({
                  color: "#fff",
                  width: 2,
                }),
                points: 4,
                radius: feature.get("width"),
                angle: Math.PI / 4,
                scale: [1.1, feature.get("height") / feature.get("width")],
                join: "round",
              }),
              text: new Text({
                text: this.formatCurrency(feature.get("price")),
                font: "12px sans-serif",
                offsetY: 0,
                offsetX: 0,
                textAlign: "center",
                fill: new Fill({
                  color: "white",
                }),
              }),
            });
          },
        });

        this.map.addLayer(vectorLayer);

        this.map.on("pointermove", (evt) => {
          const feature = this.map.forEachFeatureAtPixel(
            evt.pixel,
            (feature) => feature
          );
          if (feature) {
            this.showPopup(evt, feature);
          }
          const pixel = this.map.getEventPixel(evt.originalEvent);
          const hit = this.map.hasFeatureAtPixel(pixel);
          this.$refs.map.style.cursor = hit ? "pointer" : "";
        });

        const extent = vectorSource.getExtent();
        this.map.getView().fit(extent, { padding: [50, 50, 50, 50] });
      })
      .catch((error) => {
        console.error("Error fetching data:", error);
      });
  },
  methods: {
    showPopup(evt, feature) {
      const coordinates = feature.getGeometry().getCoordinates();
      this.popupOverlay.setPosition(coordinates);
      const properties = feature.getProperties();

      const markerPixel = this.map.getPixelFromCoordinate(coordinates);
      const mapWidth = this.map.getSize()[0];
      const mapHeight = this.map.getSize()[1];

      this.showPopupAbove = markerPixel[1] > mapHeight / 2;
      this.showPopupLeft = markerPixel[0] > mapWidth / 2;
      this.showPopupRight = !this.showPopupLeft;

      this.markerDetails = {
        title: properties.title,
        description: properties.description,
        pictures: properties.pictures,
        price: properties.price,
      };
    },
    hidePopup() {
      this.popupOverlay.setPosition(undefined);
      this.markerDetails = null;
    },
    onPopupMouseEnter() {
      this.isMouseOverPopup = true;
    },
    formatCurrency(price) {
      const formatter = new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: this.currency,
      });
      return formatter.format(price);
    },
  },
};
</script>
<style lang="scss">
.map {
  width: 100%;
  height: 100vh;
}
.ol-popup {
  position: absolute;
  background-color: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  z-index: 1000;
  width: 400px;
  padding: 0;
  &.popup-above {
    bottom: auto;
    top: 100%;
    transform: translateY(-100%);
  }

  &.popup-left {
    left: auto;
    right: 100%;
  }
  &.popup-right {
    right: auto;
    left: 100%;
  }
  .popup-images {
    margin-bottom: 10px;
  }
  .popup-content {
    padding: 16px;
    display: flex;
    flex-direction: column;
  }
  h2 {
    font-size: 22px;
    margin: 0 0 10px;
    color: #333;
    text-align: left;
  }
  p {
    font-size: 14px;
    margin: 0;
    color: #555;
    text-align: left;
    &.multiline-ellipsis {
      text-align: left;
      overflow: hidden;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 2;
      white-space: pre-wrap;
    }
    &.price {
      display: flex;
      flex-direction: row;
      align-items: flex-start;
      justify-content: flex-start;
      font-size: 22px;
      font-weight: bold;
    }
  }
}
.ol-pointer-hover {
  cursor: pointer;
}
.ol-popup-closer {
  position: absolute;
  top: 0;
  left: 0;
  margin: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2px 6px;
  line-height: 22px;
  font-size: 22px;
  background-color: #00000060;
  border: 1px solid #000000;
  color: #000000;
  border-radius: 50%;
  cursor: pointer;
  z-index: 99999999;
  &:hover {
    font-weight: bold;
    margin: 9px;
    border: 2px solid #000000;
  }
}
img {
  max-width: 100%;
  height: auto;
}
.bubble-marker {
  background-color: black;
  color: white;
  border-radius: 12px;
  padding: 8px;
  text-align: center;
}
</style>
