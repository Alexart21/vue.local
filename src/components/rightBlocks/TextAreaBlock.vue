<template>
  <span class="input-title">Текст</span>
  <textarea rows="2" @input="setTxtBoxSize" class="inner-text" type="text" v-model="tmpLayer.content"
            :style="{fontFamily: this.tmpLayer.fontFamily}"></textarea>
  <div v-if="tmpLayer.lineCount > 1" class="d-inline-flex">
    <div @click="setTextAlign('left')" class="align-icon align-left"
         :class="{iconActive: tmpLayer.textAlign === 'left'}"></div>
    <div @click="setTextAlign('center')" class="align-icon align-center"
         :class="{iconActive: tmpLayer.textAlign === 'center'}"></div>
    <div @click="setTextAlign('right')" class="align-icon align-right"
         :class="{iconActive: tmpLayer.textAlign === 'right'}"></div>
  </div>
  <div v-if="tmpLayer.lineCount > 1">
    <span class="input-title" style="margin-bottom: -2em">Межстрочный интервал</span>
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
      <input @input="setTxtBoxSize()" class="range" type="range" v-model="tmpLayer.lineHeight" min="0"
             max="100">
    </div>
    <input @input="setTxtBoxSize()" type="text" class="in-data" v-model="tmpLayer.lineHeight">
    <!--          <div style="margin-top: -1em">выровнять по центру <input @change="setTextAlign" type="checkbox" v-model="alignCenter" /></div>-->
  </div>
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
    setTextAlign(align) {
      this.tmpLayer.textAlign = align;
    },
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
