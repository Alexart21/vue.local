<template>
  <span class="input-title">Выравнивание</span>
  <button class="btn align-btn" type="button" @click="setHorisontalAlign">центр по горизонтали</button>
  <br>
  <button class="btn align-btn" type="button" @click="setVerticalAlign">центр по вертикали</button>
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
    parentWidth: {
      type: Number,
      required: true,
    },
    parentHeight: {
      type: Number,
      required: true,
    }
  },
  methods: {
    setHorisontalAlign() { // выравнивание слоя по горизонтали
      this.layers[this.tmpLayer.id].x = (this.parentWidth - this.layers[this.tmpLayer.id].w) / 2;
    },
    setVerticalAlign() { // выравнивание слоя по вертикали
      this.layers[this.tmpLayer.id].y = (this.parentHeight - this.layers[this.tmpLayer.id].h) / 2;
    },
  },
}
</script>>
