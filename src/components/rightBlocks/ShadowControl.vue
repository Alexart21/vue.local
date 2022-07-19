<template>
  <div v-if="tmpLayer.isShadow">
    <span class="input-title inline">X </span>
    &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowX">&nbsp;
    <span class="input-title inline">Y </span>
    &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowY">&nbsp;
    <span class="input-title inline">Размытие </span>
    &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowR">&nbsp;
    <br>
    <span class="input-title">Цвет тени</span>
    <div class="clr-field" :style="{color: tmpLayer.shadowColor}">
      <button aria-labelledby="clr-open-label" class="color-btn"></button>
      <input v-model="tmpLayer.shadowColor" type="text" class="coloris">
    </div>
    <br>
  </div>
</template>
<script>
export default {
  props: {
    tmpLayer: {
      type: Object,
      required: true,
    },
  }
}
</script>
