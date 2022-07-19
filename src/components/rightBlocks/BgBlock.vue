<template>
  <div class="bg-galery d-flex">
    <div @click="changeBg(index, bg.fullName)" v-for="(bg, index) in bgArr" :key="index"
         :class="{activeBg: bg.isActive}"
         class="d-flex flex-column bg-galery-parent">
      <div class="bg-galery-item"
           :style="{backgroundImage: 'url(' + require('@/assets/img/' + bg.fullName +  '') + ')'}" :key="index">
      </div>
      <div class="bg-title">{{ bg.name }}</div>
    </div>
    <!--     фейковая картинка     -->
    <img id="red" :src="require('@/assets/img/red.png')" alt="" style="display: none">
    <!--          -->
    <br>
  </div>
  <h4>Задать свой цвет</h4>
  <div class="clr-field" :style="{color: palleteColor}">
    <button class="color-btn" aria-labelledby="clr-open-label"></button>
    <!--     плагин цвета инициализирован в public/index.html     -->
    <input @change="changeBgColor" v-model="bgColor" type="text" class="coloris" :style="{width: bgColor ? '7em' : '2em'}">
  </div>
</template>
<script>
// доставание cookie
function readCookie(name) {
  const matches = document.cookie.match(new RegExp(
    "(?:^|; )" + name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, '\\$1') + "=([^;]*)"
  ));
  return matches ? decodeURIComponent(matches[1]) : undefined;
}

let bgArr = readCookie('bg'); // на бэкенде куки установлены в файле views/layouts/constructor.php
if (bgArr) {
  bgArr = JSON.parse(bgArr);
} else { // это затык для локальной работы без сервера. лучше закоментить перед билдом
  bgArr = [
    {
      "fullName": "black.png",
      "name": "black",
      "mime": "image/png",
      "width": 400,
      "height": 773,
      "isActive": true,
    },
    {
      "fullName": "silver.png",
      "name": "silver",
      "mime": "image/png",
      "width": 400,
      "height": 773,
      "isActive": false,
    },
    {
      "fullName": "red.png",
      "name": "red",
      "mime": "image/png",
      "width": 400,
      "height": 773,
      "isActive": false,
    }
  ]
}
//
export default {
  emits: ['changeBgImg', 'changeBgColor'],
  data() {
    return {
      // defaultBg: 'black.png',
      bgArr: bgArr,
      bgColor: '',
      bgSrc: '',
      bgColorChanged: 0,
      palleteColor: '',
    }
  },
  methods: {
    clearBgData() { // фон приводим в значение по умолчанию
      this.bgColor = '';
      this.palleteColor = '';
      this.bgSrc = '';
      this.bgArr.forEach(item => {
        item.isActive = false;
      });
      localStorage.removeItem('bgColor');
      localStorage.removeItem('bgIndex');
      localStorage.removeItem('bgFileName');
    },
    changeBg(index, fileName) { // изменяем фоновую картинку
      this.clearBgData();
      this.$emit('changeBgImg', fileName);
      // this.showAlert = false;
      // this.alertText = '';
      this.clearBgData();
      this.bgArr[index].isActive = true;
      this.defaultBg = fileName;
      localStorage.setItem('bgIndex', index);
      localStorage.setItem('bgFileName', fileName);
      // console.log('дочка');
      // console.log(fileName);
    },
    /*
    здесь меняем не фоновую картинку а попиксельно меняем один цвет на другой!!
    */
    changeBgColor() {
      this.bgArr.forEach(item => {
        item.isActive = false;
      });
      if (this.bgColor) {
        // console.log('method changeBgColor this.bgColor = ' + this.bgColor);
        this.palleteColor = this.bgColor;
        // return;
        if (localStorage.getItem('bgIndex')) { // если есть удаляем прежние данные
          localStorage.removeItem('bgIndex');
          localStorage.removeItem('bgFileName');
        }
        // конвертация hex => rgb
        let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(this.bgColor);
        // здесь rgb вида [255,0,0]
        let color = result ? [parseInt(result[1], 16), parseInt(result[2], 16), parseInt(result[3], 16)] : null;
        // заранее подготовленная картинка с красныч фоном
        let img = document.getElementById('red'); // в этой картинке заменяем красный цвет[255,0,0] на нужный
        let canvas = document.createElement("canvas");
        canvas.width = img.width;
        canvas.height = img.height;
        let ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0);
        let imgData = ctx.getImageData(0, 0, canvas.width, canvas.height)
        let data = imgData.data;
        let len = data.length;
        let oldColor = [255, 0, 0];
        let newColor = color;
        for (let x = 0; x < len; x += 4) {
          if ((data[x] == oldColor[0]) && (data[x + 1] == oldColor[1]) && (data[x + 2] == oldColor[2])) {
            data[x] = newColor[0];
            data[x + 1] = newColor[1];
            data[x + 2] = newColor[2];
          }
        }
        ctx.putImageData(imgData, 0, 0);
        let newSrc = canvas.toDataURL();
        this.bgSrc = newSrc;
      }
      this.$emit('changeBgColor', {bgColor: this.bgColor, bgSrc: this.bgSrc});
    },
  },
  mounted() {
    let bgIndex = localStorage.getItem('bgIndex');
    if (bgIndex) { // есть сохранненый вариант фона из предложенных
      this.changeBg(bgIndex, localStorage.getItem('bgFileName'))
    }
    let bgSrc = localStorage.getItem('bgSrc');
    if (bgSrc) { // есть сохранненый вариант фона из предложенных
      this.bgArr.forEach(item => {
        item.isActive = false;
      });
    }
    let bgColor = localStorage.getItem('bgColor');
    if (bgColor) { // есть сохранненый вариант фона из предложенных
      this.palleteColor = bgColor;
      // this.bgColor = bgColor;
    }
    // console.log('bg=' + this.bgColor)
  },
  watch: {
    bgColor: {
      handler() {
        if (this.bgColor) {
          this.changeBgColor();
        }
      }
    },
  },
}
</script>
