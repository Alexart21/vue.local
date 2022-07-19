<template>
  <input type="number" v-model="layers[tmpLayer.id][coords]">
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
    <input class="range" type="range" v-model="layers[tmpLayer.id][coords]" min="0" max="700">
  </div>
</template>
<script>
export default {
  props: {
    coords: {
      type: String,
      required: true,
    },
    tmpLayer: {
      type: Object,
      required: true,
    },
    layers: {
      type: Array,
      required: true,
    },
  },
}
</script>
