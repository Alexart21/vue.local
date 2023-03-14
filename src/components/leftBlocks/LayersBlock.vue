<template>
  <div style="color:red;height:2em;letter-spacing:2px;text-decoration:underline" class="text-center"
       :class="{visible: !tmpLayer.inUse}">{{ isActiveLayer }}
  </div>
  <div v-if="layers.length">
    <div v-for="(layer, index) in layers" :key="index">
      <div v-if="layer.type == 'txt'" class="btn btn-light layer"
           :class="{ activeLayer: layer.isActive }">
        <div class="layer-num">слой{{ index }}</div>
        <span @click="toggleViz(index)" class="eye"
              :class="{ icon_hidden: layer.visibility === 'hidden'  }"></span>
        <div class="txt-layer-icon"></div>
        <div :id="'ltab' + String(index)" class="layer-tab-text" @click="setActive(index)"
             :style="{fontFamily: layer.fontFamily, color: layer.color}">
          {{ layer.content }}
        </div>
        <div class="trash" @click="del(index)"></div>
      </div>
      <div v-else-if="layer.type == 'img'" class="btn btn-light layer"
           :class="{ activeLayer: layer.isActive }">
        <div class="layer-num">слой{{ index }}</div>
        <span @click="toggleViz(index)" class="eye" :class="{ icon_hidden: layer.visibility === 'hidden'  }"></span>
        <div class="img-layer-icon" :style="{backgroundImage: 'url(' + layer.src + ')'}"></div>
        <div :id="'ltab' + String(index)" class="layer-tab-text" @click="setActive(index)"
             :style="{fontFamily: layer.fontFamily}">
          {{ layer.fileName }}
        </div>
        <div class="trash" @click="del(index)"></div>
      </div>
    </div>
  </div>
  <div v-else style="text-align: center; margin-top: .5em">Вы пока ничего не добавили</div>
</template>
<script>
export default {
  emits: ['layersBlockEmit'],
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
    toggleViz(index) { // переключение видимости слоя
      let elem = document.getElementById(String(index));
      let control = elem.nextElementSibling;
      if (this.layers[index].visibility === 'hidden') {
        this.layers[index].visibility = 'visible';
        control.style.visibility = 'visible';
      } else {
        this.layers[index].visibility = 'hidden';
        control.style.visibility = 'hidden';
      }
    },
    setActive(index) { // делает слой активным. копируем данные слоя во временный слой для редактирования
      this.$emit('layersBlockEmit');
      this.clearActive();
      this.showAlert = false;
      this.alertText = '';
      //
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0;
      while (controls[i]) { // эл-ты управления оставляем только для активного слоя
        if (i == index) {
          controls[i].style.display = 'block';
        }
        i++;
      }
      //
      this.tmpLayer.id = this.layers[index].id;
      this.tmpLayer.class = this.layers[index].class;
      this.tmpLayer.visibility = this.layers[index].visibility;
      this.tmpLayer.type = this.layers[index].type;
      this.tmpLayer.fileName = this.layers[index].fileName;
      this.tmpLayer.content = this.layers[index].content;
      this.tmpLayer.src = this.layers[index].src;
      this.tmpLayer.x = this.layers[index].x;
      this.tmpLayer.y = this.layers[index].y;
      this.tmpLayer.scaleX = this.layers[index].scaleX;
      //
      if (this.layers[index].scaleX != 1) { // вычисляем трансформацию в процентах для показа
        this.tmpLayer.percentX = Math.floor(this.layers[index].scaleX * 100);
      }
      if (this.layers[index].scaleY != 1) {
        this.tmpLayer.percentY = Math.floor(this.layers[index].scaleY * 100);
      }
      //
      this.tmpLayer.scaleY = this.layers[index].scaleY;
      this.tmpLayer.w = this.layers[index].w;
      this.tmpLayer.h = this.layers[index].h;
      this.tmpLayer.fontFamily = this.layers[index].fontFamily;
      this.tmpLayer.fontSize = this.layers[index].fontSize;
      this.tmpLayer.interval = this.layers[index].interval;
      this.tmpLayer.lineHeight = this.layers[index].lineHeight;
      this.tmpLayer.textAlign = this.layers[index].textAlign;
      this.tmpLayer.lineCount = this.layers[index].lineCount;
      this.tmpLayer.color = this.layers[index].color;
      this.tmpLayer.rotate = this.layers[index].rotate;
      //
      this.tmpLayer.shadowX = this.layers[index].shadowX;
      this.tmpLayer.shadowY = this.layers[index].shadowY;
      this.tmpLayer.shadowR = this.layers[index].shadowR;
      this.tmpLayer.shadowColor = this.layers[index].shadowColor;
      this.tmpLayer.isShadow = this.layers[index].type === 'txt' && this.layers[index].shadowColor !== 'transparent';
      this.layers[index].isActive = true;
      this.tmpLayer.inUse = true;
      //
      // this.alignCenter = (this.tmpLayer.textAlign === 'center') ? true : false;
    },
    clearActive() { // все слои неактивными
      this.layers.forEach(item => {
        item.isActive = false;
        item.zIndex = item.id;
      })
      this.tmpLayer.inUse = false;
      this.showAlert = true;
      this.alertText = '';
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0;
      while (controls[i]) {
        controls[i].style.display = 'none';
        i++;
      }
    },
    del(index) { // удаление слоя
      this.clearTmpLayer();
      this.clearActive();
      let control = document.getElementById(index).nextElementSibling;
      if (this.layers[index]) {
        this.layers.splice(index, 1);
        if (control) {
          control.remove();
        }
        if (this.layers.length) { // пересортировка
          let i = 0;
          this.layers.forEach((item) => {
            item.id = i;
            item.class = 'l' + String(i);
            item.zIndex = i;
            i++;
          })
        } else {
          this.tmpLayer.inUse = false;
          this.showAlert = false;
          this.alertText = '';
        }
      }
    },
    clearTmpLayer() { // очистка(возврат к умолчаеию) временного слоя
      this.tmpLayer.id = '';
      this.tmpLayer.class = '';
      this.tmpLayer.visibility = 'visible';
      this.tmpLayer.zIndex = 1000;
      this.tmpLayer.type = '';
      this.tmpLayer.fileName = '';
      this.tmpLayer.src = '';
      this.tmpLayer.content = this.defaultText;
      this.tmpLayer.x = 0;
      this.tmpLayer.y = 0;
      this.tmpLayer.scaleX = 1;
      this.tmpLayer.scaleY = 1;
      this.tmpLayer.w = 0;
      this.tmpLayer.h = 0;
      this.tmpLayer.fontFamily = 'Arial';
      this.tmpLayer.fontSize = this.defaultFontSize,
        this.tmpLayer.interval = 0;
      this.tmpLayer.color = this.defaultColor;
      this.tmpLayer.rotate = '';
      this.tmpLayer.isActive = true;
      this.tmpLayer.inUse = false;
      this.tmpLayer.shadowX = 0;
      this.tmpLayer.shadowY = 0;
      this.tmpLayer.shadowR = 0;
      this.tmpLayer.shadowColor = 'transparent',
        this.tmpLayer.percentX = 100;
      this.tmpLayer.percentY = 100;
    },
  },
  computed: {
    isActiveLayer() { // предупреждение слева если слой не выбран
      if (!this.layers.length) {
        return null;
      } else {
        let flag = false;
        this.layers.forEach(item => {
          if (item.isActive) {
            flag = true;
          }
        })
        if (!flag) {
          this.alertText = 'Выберите слой!';
          return this.alertText;
        }
      }
    },
  }
}
</script>
