<template>
  <span class="input-title" style="">Шрифт</span>
  <select @change="setTxtBoxSize()" id="font" class="border" v-model="tmpLayer.fontFamily">
    <option v-for="(fontFamily,i) in fontList" :key="i" :style="{fontFamily: fontFamily}" :value="fontFamily">
      {{ fontFamily }}
    </option>
  </select>
</template>
<script>
export default {
  props: {
    tmpLayer: {
      type: Object,
      required: true,
    },
    fontList: {
      type: Array,
      required: true,
    },
    layers: {
      type: Array,
      required: true,
    },
  },
  methods: {
    setTxtBoxSize() { // устанавливаем размеры txt блока
      let text = this.tmpLayer.content;
      let pos = 0;
      let lineCount = 1;
      while (true) { // ищем количество переводов строк в тексте
        let foundPos = text.indexOf('\n', pos);
        if (foundPos == -1) break;
        pos = foundPos + 1; // продолжаем со следующей позиции
        lineCount += 1;
      }
      this.tmpLayer.lineCount = lineCount;
      let arr = [];
      let maxStr = '';
      if (lineCount > 1) { // многострочный текст
        arr = text.split('\n');
        arr.forEach((item) => { // находим самую длинную строку в многострочном тексте
          if (item.length > maxStr.length) {
            maxStr = item;
          }
        });
        text = maxStr;
      }
      let el = document.createElement('span');
      el.style.fontSize = this.tmpLayer.fontSize + 'px';
      el.style.fontFamily = this.tmpLayer.fontFamily;
      el.style.letterSpacing = this.tmpLayer.interval + 'px';
      el.style.lineHeight = this.tmpLayer.lineHeight + 'px';
      el.innerHTML = text;
      document.body.appendChild(el);
      let w = el.offsetWidth;
      let h = el.offsetHeight * this.tmpLayer.lineCount; // помножим на количество строк
      document.body.removeChild(el);
      this.tmpLayer.w = w;
      this.layers[this.tmpLayer.id].w = w;
      this.tmpLayer.h = h;
      this.layers[this.tmpLayer.id].h = h;
    },
  }
}
</script>
