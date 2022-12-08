<template>
  <div class="wrapper d-flex justify-content-center">
    <div class="left">
      <add-buttons :tmpLayer="tmpLayer" :layers="layers" @addButtonsEmit="addButtonsEmit"/>
      <layers-block :tmpLayer="tmpLayer" :layers="layers" @layersBlockEmit="layersBlockEmit"/>
      <save-buttons v-if="layers.length" @saveButtonsEmit="saveButtonsEmit"/>
      <br>
      <br>
      <progress-bar/>
    </div>
    <!--    -->
    <editor :layers="layers" :tmpLayer="tmpLayer" :defaultBg="defaultBg" :bgSrc="bgSrc" :showAlert="showAlert" :alertText="alertText"
            :noBorder="noBorder" :loader="loader" />
    <!--    -->
    <div class="right">
      <div class="right-inner">
        <button v-if="!showBgGalery" @click="showBgList" class="btn btn-secondary">Выбор фона</button>
        <!--      <button @click="loadBg" type="button" class="btn btn-secondary" style="margin-left: 1em">загрузить свой фон
              </button>-->
        <div v-if="showBgGalery">
          <bg-block @changeBgImg="makeBgImg" @changeBgColor="makeBgColor"/>
        </div>
        <div v-if="(tmpLayer.type == 'txt' && tmpLayer.inUse) && !showBgGalery">
          <text-area-block :tmpLayer="tmpLayer" :layers="layers"/>
          <font-family-select :tmpLayer="tmpLayer" :layers="layers" :fontList="fontList"/>
          <font-size-control :tmpLayer="tmpLayer" :layers="layers"/>
          <letter-spacing-control :tmpLayer="tmpLayer" :layers="layers"/>
          <text-color-control :tmpLayer="tmpLayer"/>
          <br>
          <span class="input-title inline">Тень</span>&nbsp;&nbsp;
          <!--        -->
          <input type="checkbox" @change="toggleShadow" v-model="tmpLayer.isShadow"/>
          <br>
          <shadow-control :tmpLayer="tmpLayer"/>
          <scale-control :tmpLayer="tmpLayer"/>
          <span class="input-title">Координаты</span>
          <div class="d-flex flex-column">
            <div>
              X
              <move-control coords="x" :tmpLayer="tmpLayer" :layers="layers"/>
            </div>
            <br>
            <div>
              Y
              <move-control coords="y" :tmpLayer="tmpLayer" :layers="layers"/>
            </div>
          </div>
          <br>
          <rotate-control :tmpLayer="tmpLayer"/>
          <br>
          <br>
          <allign-block :tmpLayer="tmpLayer" :layers="layers" :parentWidth="parentWidth" :parentHeight="parentHeight"/>
          <br>
          <br>
          <reset-transform-button :tmpLayer="tmpLayer" :layers="layers"/>
        </div>
        <!--  для картинки    -->
        <div v-else-if="(tmpLayer.type == 'img' && tmpLayer.inUse) && !showBgGalery">
          <scale-control :tmpLayer="tmpLayer"/>
          <span class="input-title">Координаты</span>
          <div class="d-flex flex-column">
            <div>
              X
              <move-control coords="x" :tmpLayer="tmpLayer" :layers="layers"/>
            </div>
            <br>
            <div>
              Y
              <move-control coords="y" :tmpLayer="tmpLayer" :layers="layers"/>
            </div>
          </div>
          <br>
          <rotate-control :tmpLayer="tmpLayer"/>
          <br>
          <allign-block :tmpLayer="tmpLayer" :layers="layers" :parentWidth="parentWidth" :parentHeight="parentHeight"/>
          <br>
          <br>
          <reset-transform-button :tmpLayer="tmpLayer" :layers="layers"/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
// import {defineComponent} from "vue";
// import Moveable from "vue3-moveable";
//
import Editor from '@/components/Editor';
//
import {settings} from "@/_config";
import RotateControl from '@/components/rightBlocks/RotateControl';
import MoveControl from '@/components/rightBlocks/MoveControl';
import ResetTransformButton from "@/components/rightBlocks/ResetTransformButton.vue";
import AllignBlock from '@/components/rightBlocks/AllignBlock';
import ScaleControl from '@/components/rightBlocks/ScaleControl';
import ShadowControl from '@/components/rightBlocks/ShadowControl';
import TextColorControl from '@/components/rightBlocks/TextColorControl';
import LetterSpacingControl from '@/components/rightBlocks/LetterSpacingControl';
import FontSizeControl from '@/components/rightBlocks/FontSizeControl';
import FontFamilySelect from '@/components/rightBlocks/FontFamilySelect';
import TextAreaBlock from '@/components/rightBlocks/TextAreaBlock';
import BgBlock from '@/components/rightBlocks/BgBlock';
import AddButtons from '@/components/leftBlocks/AddButtons';
import LayersBlock from '@/components/leftBlocks/LayersBlock';
import SaveButtons from '@/components/leftBlocks/SaveButtons';
import ProgressBar from '@/components/leftBlocks/ProgressBar';

export default ({
  components: {
    Editor,
    RotateControl,
    MoveControl,
    ResetTransformButton,
    AllignBlock,
    ScaleControl,
    ShadowControl,
    TextColorControl,
    LetterSpacingControl,
    FontSizeControl,
    FontFamilySelect,
    TextAreaBlock,
    BgBlock,
    AddButtons,
    LayersBlock,
    SaveButtons,
    ProgressBar,
  },
  data() {
    return {
      loader: false,
      showAlert: false,
      alertText: 'Выберите слой',
      parentWidth: settings.containerW, // 400
      parentHeight: settings.containerH, // 773
      defaultText: 'ваш текст',
      defaultColor: '#0000ff',
      defaultFontSize: settings.fontSize, // 24
      defaultLineHeight: this.defaultFontSize,
      defaultTextAlign: 'left',
      defaultFontFamily: 'Arial',
      defaultTextWidth: 0,
      defaultTextHeight: 0,

      defaultBg: 'black.png',
      bgSrc: '',
      showBgGalery: true,

      noBorder: false,
      //
      fontList: [
        'Arial',
        'comic sans ms',
        'segoe script',
      ],
      //
      tmpLayer: { // сюда копируються данные активного слоя для редактирования
        id: '',
        class: '',
        visibility: 'visible',
        zIndex: 1000,
        type: '',
        fileName: '',
        src: '',
        content: this.defaultText,
        x: 0,
        y: 0,
        scaleX: 1,
        scaleY: 1,
        w: 0,
        h: 0,
        fontFamily: this.defaultFontFamily,
        fontSize: this.defaultFontSize,
        interval: 0,
        lineHeight: this.defaultLineHeight,
        textAlign: 'left',
        lineCount: 1,
        color: this.defaultColor,
        rotate: '',
        isActive: true,
        inUse: false,
        //
        shadowX: 0,
        shadowY: 0,
        shadowR: 0,
        shadowColor: 'transparent',
        isShadow: false,
        //
        percentX: 100, // трансформация (то же что scaleX) только в процентах
        percentY: 100,
      },
      layers: [],
    };
  },
  methods: {
    addButtonsEmit($event) {
      if($event){
        if($event.loader){
        this.showAlert = true;
        this.alertText = 'загружаю...';
        this.loader = true;
      }else{
        this.loader = false;
        this.showAlert = false;
        // this.alertText = '';
      }
      }
      
      this.showBgGalery = false;
    },
    layersBlockEmit() {
      document.getElementById('img-container').style.border = 'none';
      this.clearActive();
      this.showAlert = false;
      // this.alertText = '';
      this.hideProgress();
      this.showBgGalery = false;
      this.showAlert = false;
      this.alignCenter = (this.tmpLayer.textAlign === 'center') ? true : false
    },
    saveButtonsEmit($event) {
      this.clearActive();
      this.hideControls();// убираем управляющие элементы перед скриншотом
      this.noBorder = true;
      this.tmpLayer.inUse = false;
      this.showAlert = false;
      // this.alertText = '';
      this.hideProgress();
      if ($event.clear) { // кнопка "очистить все"
        this.clearTmpLayer();
        this.bgColor = '';
        // this.bgSrc = null;
        this.layers = [];
        localStorage.clear();
        this.tmpSrc = this.tmpW = this.tmpH = this.tmpBg = this.tmpFileName = undefined;
      }

    },
    init() {
      if (this.layers.length) {
        this.hideControls();
        this.clearActive();
      }
    },
    hideControls() { // скрываем эл-ты управления
      this.showAlert = false;
      // this.alertText = '';
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0;
      while (controls[i]) {
        if (controls[i].style.display === 'block') {
          controls[i].style.display = 'none';
        }
        i++;
      }
    },
    hideBorder() { // убираем управляющие элементы перед скриншотом
      this.noBorder = true;
      this.hideControls();
      this.tmpLayer.inUse = false;
      // let container = this.$refs.imgContainer;
      this.$refs.imgContainer.style.border = 'none';
      this.clearActive();
      this.showAlert = false;
      // this.alertText = '';
      this.hideProgress();
    },
    startLoader() {
      document.body.style.cursor = 'progress';
      this.loader = true;
    },
    stopLoader() {
      document.body.style.cursor = ''
      this.loader = false;
    },
    clearActive() { // все слои неактивными
      this.layers.forEach(item => {
        item.isActive = false;
        item.zIndex = item.id;
      })
      this.tmpLayer.inUse = false;
      this.showAlert = true;
      // this.alertText = '';
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0;
      while (controls[i]) {
        controls[i].style.display = 'none';
        i++;
      }
      // this.showAlert = false;
    },
    save() { // слои сохраняем в localStorage
      try {
        localStorage.setItem('layers', JSON.stringify(this.layers));
      } catch (e) {
        console.log(e);
        alert('Превышена квота LocalStorage');
        return
      }
    },
    initTxtWidth() { // финт ушами для вычесления ширины текстового блока
      let el = document.createElement('span');
      el.style.fontSize = this.defaultFontSize + 'px';
      el.style.fontFamily = this.defaultFontFamily;
      el.style.letterSpacing = 0;
      el.innerHTML = this.defaultText;
      document.body.appendChild(el);
      let w = el.offsetWidth;
      this.defaultTextWidth = w;
      document.body.removeChild(el);
    },
    initTxtHeight() { // финт ушами для вычесления высоты текстового блока
      let el = document.createElement('span');
      el.style.fontSize = this.defaultFontSize + 'px';
      el.style.fontFamily = this.defaultFontFamily;
      el.style.letterSpacing = 0;
      el.innerHTML = this.defaultText;
      document.body.appendChild(el);
      let h = el.offsetHeight;
      this.defaultTextHeight = h;
      document.body.removeChild(el);
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
    toggleShadow() {
      this.tmpLayer.isShadow = !this.tmpLayer.isShadow;
      if (this.tmpLayer.isShadow) {
        this.tmpLayer.isShadow = false;
        this.tmpLayer.shadowX = 0;
        this.tmpLayer.shadowY = 0;
        this.tmpLayer.shadowR = 0;
        this.tmpLayer.shadowColor = 'transparent';
      } else {
        this.tmpLayer.isShadow = true;
      }
    },
    getLayers() { // достаем если есть сохраненные слои
      let localLayers = localStorage.getItem('layers');
      if (localLayers) {
        this.layers = JSON.parse(localLayers);
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
    hideProgress() { // скрываем прогресс бар загрузки
      document.querySelector('.progress').style.visibility = 'hidden';
      let alert = document.querySelector('.alert');
      alert.style.display = 'none';
      alert.style.visibility = 'hidden';
    },
    clearBgData() { // фон приводим в значение по умолчанию
      this.bgSrc = null;
      this.bgColor = '';
      localStorage.removeItem('bgColor');
      localStorage.removeItem('bgSrc');
      localStorage.removeItem('bgIndex');
      localStorage.removeItem('bgFileName');
    },
    makeBgImg($event) { // изменяем фоновую картинку
      this.showAlert = false;
      this.clearBgData();
      // console.log('here');
      // console.log($event);
      this.defaultBg = $event;
      // this.showAlert = false;
      // this.alertText = '';
      // this.clearBgData();
      // this.bgArr[index].isActive = true;
      // this.defaultBg = fileName;
      // localStorage.setItem('bgIndex', index);
      // localStorage.setItem('bgFileName', fileName);
    },
    makeBgColor($event) {
      // console.log($event.bgColor);
      this.showAlert = false;
      this.clearBgData();
      this.bgColor = $event.bgColor;
      this.bgSrc = $event.bgSrc;
      // console.log('makeBgC');
      // console.log($event);
      localStorage.setItem('bgColor', this.bgColor);
      localStorage.setItem('bgSrc', this.bgSrc);
    },
    showBgList() { // показываем галерею фоновых картинок
      this.showBgGalery = true;
    },
  },
  mounted() {
    if (this.layers.length && (!this.tmpLayer.inUse && !this.loader)) {
      this.showAlert = true;
      // this.alertText = 'Выберите слой';
    }
    // достаем сохраненные данные
    let bgSrc = localStorage.getItem('bgSrc');
    if (bgSrc) { // есть сохранненый вариант фона из предложенных
      this.bgSrc = bgSrc;
    }
    let bgColor = localStorage.getItem('bgColor');
    if (bgColor) { // есть сохранненый вариант фона из предложенных
      this.bgColor = bgColor;
    }
    this.getLayers(); // слои
    this.init();
  },
  watch: {
    layers: {
      handler() {
        this.save();
      },
      deep: true
    },
    tmpLayer: {
      handler(updatedList) {
        if (updatedList.inUse) {
          // this.layers[updatedList.id].zIndex = updatedList.zIndex;
          // this.layers[updatedList.id].class = updatedList.class;
          // this.layers[updatedList.id].x = updatedList.x;
          // this.layers[updatedList.id].y = updatedList.y;
          this.layers[updatedList.id].scaleX = updatedList.scaleX;
          this.layers[updatedList.id].scaleY = updatedList.scaleY;
          this.layers[updatedList.id].w = updatedList.w;
          this.layers[updatedList.id].h = updatedList.h;
          this.layers[updatedList.id].content = updatedList.content;
          // this.layers[updatedList.id].src = updatedList.src;
          this.layers[updatedList.id].fontSize = updatedList.fontSize;
          this.layers[updatedList.id].fontFamily = updatedList.fontFamily;
          this.layers[updatedList.id].interval = updatedList.interval;
          this.layers[updatedList.id].lineHeight = updatedList.lineHeight;
          this.layers[updatedList.id].lineCount = updatedList.lineCount;
          this.layers[updatedList.id].textAlign = updatedList.textAlign;
          this.layers[updatedList.id].color = updatedList.color;
          this.layers[updatedList.id].rotate = updatedList.rotate;
          //
          this.layers[updatedList.id].shadowX = updatedList.shadowX;
          this.layers[updatedList.id].shadowY = updatedList.shadowY;
          this.layers[updatedList.id].shadowR = updatedList.shadowR;
          this.layers[updatedList.id].shadowColor = updatedList.shadowColor;
          this.layers[updatedList.id].isShadow = updatedList.isShadow;
          this.layers[updatedList.id].textAlign = updatedList.textAlign;
          // this.save();
        }
      },
      deep: true
    },
  },
});
</script>
