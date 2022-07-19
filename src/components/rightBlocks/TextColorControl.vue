<template>
  <span class="input-title" style="margin-top: -1em">Цвет текста</span>
  <div class="clr-field" :style="{color: tmpLayer.color}">
    <button aria-labelledby="clr-open-label" class="color-btn"></button>
    <!--     плагин цвета инициализирован в public/index.html     -->
    <input v-model="tmpLayer.color" type="text" class="coloris">
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
