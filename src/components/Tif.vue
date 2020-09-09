<template>
  <div class="tif">
    <h1>Image tag:</h1>
    <img :width="width" :height="height" :src="imgData" class="image" />
    <h1>Canvas tag:</h1>
    <canvas ref="canvas" class="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import UTIF from "utif";

@Component
export default class Tif extends Vue {
  width = 0;
  height = 0;
  imgData = "";

  mounted() {
    this.fetchImage();
  }

  fetchImage() {
    fetch("bali.tif")
      .then(response => {
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        return response.arrayBuffer();
      })
      .then(data => {
        const ifds = UTIF.decode(data);
        UTIF.decodeImage(data, ifds[0]);
        const rgba = UTIF.toRGBA8(ifds[0]);
        const widh = ifds[0].width;
        const height = ifds[0].height;
        const canvas = this.$refs.canvas as HTMLCanvasElement;
        canvas.width = widh;
        canvas.height = height;
        this.width = widh;
        this.height = height;
        const ctx = canvas.getContext("2d");
        const imgData = new ImageData(
          new Uint8ClampedArray(rgba.buffer),
          widh,
          height
        );
        ctx?.putImageData(imgData, 0, 0);
        canvas.toBlob(blob => {
          this.imgData = URL.createObjectURL(blob);
        });
      });
  }
  beforeDestroy() {
    URL.revokeObjectURL(this.imgData);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.image {
  display: flex;
  margin: auto;
}
.canvas {
  flex: none;
}
</style>
