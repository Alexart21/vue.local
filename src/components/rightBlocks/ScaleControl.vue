<template>
  <span class="input-title">Масштабирование %</span>
  <div class="d-flex" style="margin-top: -.5em">
    <div class="d-flex flex-column" style="margin-right: 1em">
      <span class="input-title">Ширина</span>
      <input @input="scalePercentX(tmpLayer.id)" min="-1000" max="1000" type="number"
             v-model="tmpLayer.percentX">
    </div>
    <div class="d-flex flex-column">
      <span class="input-title">Высота</span>
      <input @input="scalePercentY(tmpLayer.id)" min="-1000" max="1000" type="number"
             v-model="tmpLayer.percentY">
    </div>
  </div>
</template>
<script>
export default {
  props: {
    tmpLayer: {
      type: Object,
      required: true,
    },
  },
  methods: {
    scalePercentX(index) { // трансформируем приводя проценты к дробному числу
      if (String(index)) {
        this.tmpLayer.scaleX = this.tmpLayer.percentX / 100;
      }
    },
    scalePercentY(index) { // трансформируем приводя проценты к дробному числу
      if (String(index)) {
        this.tmpLayer.scaleY = this.tmpLayer.percentY / 100;
      }
    },
  },
}
</script>
