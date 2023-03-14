<template>
  <button @click="resetTransform(tmpLayer.id)" type="button" class="btn btn-warning">сбросить все</button>
</template>
<script>
export default {
  props: {
    tmpLayer: {
      type: Object,
      required: true,
    },
    layers: {
      type: Array,
      required: true,
    },
  },
  methods: {
    resetTransform(id) { // сброс всех трансформаций слоя
      if (id >= 0) {
        document.getElementById('ltab' + String(id)).click();
        this.tmpLayer.scaleX = 1;
        this.tmpLayer.percentX = 100;
        this.tmpLayer.scaleY = 1;
        this.tmpLayer.percentY = 100;
        this.tmpLayer.rotate = 0;
        this.tmpLayer.fontSize = this.defaultFontSize;
        this.tmpLayer.interval = 0;

        this.tmpLayer.shadowX = 0;
        this.tmpLayer.shadowY = 0;
        this.tmpLayer.shadowR = 0;
        this.tmpLayer.isShadow = false;
        this.tmpLayer.fontSize = 24;
        this.tmpLayer.interval = 0;
        this.tmpLayer.fontFamily = 'Arial';

        this.layers[id].x = 20;
        this.layers[id].y = 20;
        //
        if (this.tmpLayer.type == 'img') {
          this.layers[id].w = this.layers[id].originalW;
          this.layers[id].h = this.layers[id].originalH;
        }
      }
    },
  },
}
</script>