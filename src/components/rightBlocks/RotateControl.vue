<template>
  <span class="input-title inline">Поворот </span>
  <input type="number" v-model="tmpLayer.rotate" min="-180" max="180">
  <div class="range-out">
    <div class="scale">
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
      <div>|</div>
    </div>
    <input class="range" type="range" v-model="tmpLayer.rotate" min="-180" max="180">
  </div>
  <!-- <input  type="text" class="in-data" v-model="tmpLayer.rotate" min="-180" :max="180"> -->
</template>
<script>
export default {
  props: {
    tmpLayer: {
      type: Object,
      required: true,
    },
  },
}
</script>
