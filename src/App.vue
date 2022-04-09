<template>
  <div class="wrapper d-flex justify-content-center">
    <div class="left">
      <div class="d-flex">
        <input id="imgInput" type="file" @input="readFile" style="display: none">
        <!-- фейковые кнопки -->
        <button id="insert" type="button" @click="insertImg" style="display: none"></button>
        <button id="push" type="button" @click="pushImg" style="display: none"></button>
        <!---->
        <div class="img-tab btn btn-light d-flex" @click="addImg">
          <div class="img-tab-icon"></div>
          <div class="tab-title">+ изображение</div>
        </div>
        <div class="text-tab btn btn-light d-flex" @click="addText">
          <div class="text-tab-icon"></div>
          <div class="tab-title">+ текст</div>
        </div>
      </div>
      <div style="height: 2em" class="text-center" :class="{visible: !tmpLayer.inUse}">{{ isActiveLayer }}</div>
      <div v-if="layers.length" v-for="(layer, index) in layers" :key="index">
        <div v-if="layer.type == 'txt'" class="btn btn-light layer"
             :class="{ activeLayer: layer.isActive }">
              <span @click="toggleViz(index)" class="eye"
                    :class="{ icon_hidden: layer.visibility === 'hidden'  }"></span>
          <div class="txt-layer-icon"></div>
          <div :id="'ltab' + String(index)" class="layer-tab-text" @click="setActive(index)"
               :style="{fontFamily: layer.fontFamily, color: layer.color}">
            {{ layer.content }}
          </div>
          <div class="trash" @click="del(index)"></div>
        </div>
        <div v-else="layer.type == 'img'" class="btn btn-light layer"
             :class="{ activeLayer: layer.isActive }">
          <span @click="toggleViz(index)" class="eye" :class="{ icon_hidden: layer.visibility === 'hidden'  }"></span>
          <div class="img-layer-icon" :style="{backgroundImage: 'url(' + layer.src + ')'}"></div>
          <div :id="'ltab' + String(index)" class="layer-tab-text" @click="setActive(index)"
               :style="{fontFamily: layer.fontFamily}">
            {{ layer.fileName }}
          </div>
          <div class="trash" @click="del(index)"></div>
        </div>
      </div>
      <div v-else style="text-align: center; margin-top: .5em">Вы пока ничего не добавили</div>
      <div v-if="layers.length" class="d-flex justify-content-between" style="margin-top: 1em">
        <button class="btn btn-success" @click="exportAll">сохранить</button>
        <button class="btn btn-secondary" @click="hideBorder">предпросмотр</button>
        <button id="clear-btn" class="btn btn-danger" @click="clear"><i class="fa fa-trash"> </i>Очистить все
        </button>
      </div>
      <br>
      <br>
      <button v-if="layers.length" class="btn btn-success" @click="send">Отправить</button>
      <br>
      <br>
      <div class="progress">
        <div class="progress-bar progress-bar-striped bg-success"></div>
      </div>
      <br>
      <div class="alert" role="alert"></div>
    </div>
    <!--    -->
    <div class="editor">
      <div id="img-container" :style="{backgroundImage: getBgSrc}">
        <div v-if="layers.length" v-for="(layer, index) in layers" :key="index">
          <!--     текст     -->
          <div v-if="layer.type === 'txt'">
            <div :id="index" class="contenteditable" :class="layer.class"
                 :style="{transform: 'translate(' + layer.x + 'px, ' + layer.y + 'px) ' + 'rotate(' + layer.rotate + 'deg) ' + 'scale(' + layer.scaleX + ', ' + layer.scaleY + ')', fontFamily: layer.fontFamily, color: layer.color, fontSize: layer.fontSize + 'px', letterSpacing: layer.interval + 'px', visibility: layer.visibility, zIndex: layer.zIndex, textShadow: layer.shadowX + 'px ' + layer.shadowY + 'px ' + layer.shadowR + 'px ' + layer.shadowColor, lineHeight: layer.lineHeight + 'px', textAlign: layer.textAlign}">
              {{ layer.content }}
            </div>
            <moveable
              v-bind:target="['.' + layer.class]"
              v-bind:throttleDrag="1"
              v-bind:edgeDraggable="false"
              v-bind:startDragRotate="0"
              v-bind:throttleDragRotate="0"
              className="moveable"
              v-bind:draggable=true
              v-bind:scalable=true
              v-bind:rotatable=true
              @drag="onDrag"
              @scale="onScale"
              @rotate="onRotate"
            />
          </div>
          <!--     картинка     -->
          <div v-else>
            <div :id="index" class="imgL" :class="layer.class"
                 :style="{backgroundImage: 'url(' + layer.src + ')', width: layer.w + 'px', height: layer.h + 'px', transform: 'translate(' + layer.x + 'px, ' + layer.y + 'px) ' + 'rotate(' + layer.rotate + 'deg) ' + 'scale(' + layer.scaleX + ', ' + layer.scaleY + ')', visibility: layer.visibility, zIndex: layer.zIndex}">
            </div>
            <moveable
              v-bind:target="['.' + layer.class]"
              v-bind:throttleDrag="1"
              v-bind:edgeDraggable="false"
              v-bind:startDragRotate="0"
              v-bind:throttleDragRotate="0"
              className="moveable"
              v-bind:draggable=true
              v-bind:scalable=true
              v-bind:rotatable=true
              @drag="onDrag"
              @scale="onScale"
              @rotate="onRotate"
            />
          </div>
          <!--          <div id="zatyk"></div>&lt;!&ndash; перекрывает все слои кроме активного &ndash;&gt;-->
        </div>
        <h1 v-if="showAlert" id="layers-over">выберите слой</h1>
      </div>
      <!--    -->
    </div>
    <div class="right">
      <div class="right-inner">
      <button v-if="!showBgGalery" @click="showBgList" class="btn btn-secondary">Выбор фона</button>
      <!--      <button @click="loadBg" type="button" class="btn btn-secondary" style="margin-left: 1em">загрузить свой фон
            </button>-->
      <div v-if="showBgGalery">
        <div class="bg-galery d-flex">
          <div @click="changeBg(index, bg.fullName)" v-for="(bg, index) in bgArr" :class="{activeBg: bg.isActive}"
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
        <div class="clr-field" :style="{color: bgColor}">
          <button class="color-btn" aria-labelledby="clr-open-label"></button>
          <!--     плагин цвета инициализирован в public/index.html     -->
          <input @change="changeBgColor" v-model="bgColor" type="text" class="coloris">
        </div>
      </div>
      <div v-if="(tmpLayer.type == 'txt' && tmpLayer.inUse) && !showBgGalery">
        <span class="input-title">Текст</span>
        <textarea  rows="2" @input="setTxtBoxSize" class="inner-text" type="text"
                  v-model="tmpLayer.content" :style="{fontFamily: this.tmpLayer.fontFamily}"></textarea>
        <div v-if="tmpLayer.lineCount > 1" class="d-inline-flex">
          <div @click="setTextAlign('left')" class="align-icon align-left" :class="{iconActive: tmpLayer.textAlign === 'left'}"></div>
          <div @click="setTextAlign('center')" class="align-icon align-center" :class="{iconActive: tmpLayer.textAlign === 'center'}"></div>
          <div @click="setTextAlign('right')" class="align-icon align-right" :class="{iconActive: tmpLayer.textAlign === 'right'}"></div>
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
            <input @input="setTxtBoxSize" class="range" type="range" v-model="tmpLayer.lineHeight" min="0"
                   max="100">
          </div>
          <input @input="setTxtBoxSize" type="text" class="in-data" v-model="tmpLayer.lineHeight">
<!--          <div style="margin-top: -1em">выровнять по центру <input @change="setTextAlign" type="checkbox" v-model="alignCenter" /></div>-->
        </div>
        <span class="input-title" style="">Шрифт</span>
        <select @change="setTxtBoxSize" id="font" class="border" v-model="tmpLayer.fontFamily">
          <option style="font-family: Arial" value="Arial" selected>Arial</option>
          <option style="font-family: comic sans ms" value="comic sans ms">Comic Sans MS</option>
          <option style="font-family: segoe script" value="segoe script">Segoe Script</option>
        </select>
        <span @input="setTxtBoxSize" class="input-title" style="margin-bottom: -2em">Размер шрифта</span>
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
          <input @input="setTxtBoxSize" class="range" type="range" v-model="tmpLayer.fontSize" min="0"
                 max="600">
        </div>
        <input class="in-data" id="fontSize" type="text" min="1" max="600"
               v-model="tmpLayer.fontSize">
        <span class="input-title" style="margin-bottom: -2em;margin-top: -1em">Межсимвольный интервал</span>
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
          <input @change="setTxtBoxSize" class="range" type="range" v-model="tmpLayer.interval" min="0"
                 max="100">
        </div>
        <input @change="setTxtBoxSize" type="text" class="in-data" min="0" max="100" v-model="tmpLayer.interval">
        <span class="input-title" style="margin-top: -1em">Цвет текста</span>
        <!--        <input type="color" v-model="tmpLayer.color" >-->
        <div class="clr-field" :style="{color: tmpLayer.color}">
          <button aria-labelledby="clr-open-label" class="color-btn"></button>
          <!--     плагин цвета инициализирован в public/index.html     -->
          <input v-model="tmpLayer.color" type="text" class="coloris">
        </div>
        <br>
        <span class="input-title inline">Тень</span>&nbsp;&nbsp;
        <!--        -->
        <input type="checkbox" @change="txtShadow" :checked="setShadow" v-model="setShadow"/>
        <br>
        <!--        -->
        <div v-if="setShadow">
          <span class="input-title inline">X </span>
          &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowX">&nbsp;
          <span class="input-title inline">Y </span>
          &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowY">&nbsp;
          <span class="input-title inline">Разброс </span>
          &nbsp;<input type="number" min="0" max="300" v-model="tmpLayer.shadowR">&nbsp;
          <br>
          <span class="input-title">Цвет тени</span>
          <div class="clr-field" :style="{color: tmpLayer.shadowColor}">
            <button aria-labelledby="clr-open-label" class="color-btn"></button>
            <input v-model="tmpLayer.shadowColor" type="text" class="coloris">
          </div>
          <br>
        </div>
        <span class="input-title">Масштабирование %</span>
        <div class="d-flex" style="margin-top: -.5em">
          <div class="d-flex flex-column" style="margin-right: 1em">
            <span class="input-title">Ширина</span>
            <input @input="scalePercentX(tmpLayer.id)" min="-1000" max="1000" type="number"
                   v-model="this.tmpLayer.percentX">
          </div>
          <div class="d-flex flex-column">
            <span class="input-title">Высота</span>
            <input @input="scalePercentY(tmpLayer.id)" min="-1000" max="1000" type="number"
                   v-model="this.tmpLayer.percentY">
          </div>
        </div>
        <span class="input-title" style="margin-bottom: -2em">Координаты</span>
        <div class="d-flex flex-column">
          <div>
            X
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
              <input class="range" type="range" v-model="this.layers[this.tmpLayer.id].x" min="0" max="400">
            </div>
            <input type="text" class="in-data" v-model="this.layers[this.tmpLayer.id].x">
          </div>
          <div style="margin-top: -2em">
            Y
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
              <input class="range" type="range" v-model="this.layers[this.tmpLayer.id].y" min="0" max="700">
            </div>
            <input type="text" class="in-data" v-model="this.layers[this.tmpLayer.id].y">
          </div>
        </div>
        <span class="input-title" style="margin: -1em 0 -2em 0">Поворот </span>
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
        <input type="text" class="in-data" v-model="tmpLayer.rotate" min="-180" max="180">
        <span class="input-title" style="margin-top: -1em">Выравнивание</span>
        <button class="btn align-btn" type="button" @click="setHorisontalAlign">центр по горизонтали</button>
        <br>
        <button class="btn align-btn" type="button" @click="setVerticalAlign">центр по вертикали</button>
        <br>
        <br>
        <button @click="resetTransform(tmpLayer.id)" type="button" class="btn btn-warning">сбросить все</button>
      </div>
      <!--  для картинки    -->
      <div v-else v-if="(tmpLayer.type == 'img' && tmpLayer.inUse) && !showBgGalery">
        <!--        <div v-if="tmpLayer.inUse && (tmpLayer.scaleX !== 1 || tmpLayer.scaleY !== 1)">-->
        <span class="input-title" style="margin-bottom: -.5em">Масштабирование %</span>
        <div class="d-flex">
          <div class="d-flex flex-column" style="margin-right: 1em">
            <span class="input-title">Ширина</span>
            <input @input="scalePercentX(tmpLayer.id)" min="-1000" max="1000" type="number"
                   v-model="this.tmpLayer.percentX">
          </div>
          <div class="d-flex flex-column">
            <span class="input-title">Высота</span>
            <input @input="scalePercentY(tmpLayer.id)" min="-1000" max="1000" type="number"
                   v-model="this.tmpLayer.percentY">
          </div>
        </div>
        <span class="input-title">Координаты</span>
        <div class="d-flex flex-column">
          <div>
            X <input type="number" v-model="this.layers[this.tmpLayer.id].x">
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
              <input class="range" type="range" v-model="this.layers[this.tmpLayer.id].x" min="0" max="400">
            </div>
          </div>
          <br>
          <div>
            Y <input type="number" v-model="this.layers[this.tmpLayer.id].y">
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
              <input class="range" type="range" v-model="this.layers[this.tmpLayer.id].y" min="0" max="700">
            </div>
          </div>
        </div>
        <!--        </div>-->
        <br>
        <!--        <div v-if="tmpLayer.rotate">-->
        <span class="input-title inline">Поворот </span><input type="number" v-model="tmpLayer.rotate" min="-180"
                                                               max="180">
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
        <br>
        <span class="input-title">Выравнивание</span>
        <button class="btn align-btn" type="button" @click="setHorisontalAlign">центр по горизонтали</button>
        <br>
        <button class="btn align-btn" type="button" @click="setVerticalAlign">центр по вертикали</button>
        <br>
        <br>
        <button @click="resetTransform(tmpLayer.id)" type="button" class="btn btn-warning">сбросить все</button>
      </div>
    </div>
    </div>
  </div>
</template>
<script>
let tmpW, tmpH, tmpSrc, tmpBg, tmpBgFileName, tmpFileName;
//
const maxImgWidth = 360;
// const maxImgSize = 4; // объявил в views/layout/constructor.php !
const maxImgSize = 4; // в мегабайтах
const containerW = 400; // в пикселях
const containerH = 773;
const fontSize = 24;

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
import {defineComponent} from "vue";
import Moveable from "vue3-moveable";
import html2canvas from "@/assets/js/html2canvas";


export default defineComponent({
  components: {
    Moveable,
  },
  data() {
    return {
      parentWidth: containerW,
      parentHeight: containerH,
      defaultText: 'ваш текст',
      defaultColor: '#0000ff',
      defaultFontSize: fontSize, // 24
      defaultLineHeight: this.defaultFontSize,
      defaultTextAlign: 'left',
      defaultFontFamily: 'Arial',
      defaultTextWidth: 0,
      defaultTextHeight: 0,
      defaultBg: 'black.png',
      bgArr: bgArr,
      showBgGalery: true,
      setShadow: false,
      bgColor: '',
      bgSrc: '',
      showAlert: false,
      bgColorChanged: 0,
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
        fontFamily: 'Arial',
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
        //
        percentX: 100, // трансформация (то же что scaleX) только в процентах
        percentY: 100,
      },
      layers: [],
    };
  },
  methods: {
    init() {
      if (this.layers.length) {
        this.hideControls();
        this.clearActive();
      }
    },
    onDrag(e) {
      try {
        let index = e.inputEvent.target.id;
        this.layers[index].x = e.translate[0];
        this.layers[index].y = e.translate[1];
      } catch {
      }
      e.target.style.transform = e.transform;
    },
    onRotate(e) {
      // let el = e.inputEvent.target;
      let index = e.target.getAttribute('id');
      this.layers[index].rotate = this.tmpLayer.rotate = e.rotate;

      e.target.style.transform = e.transform;
    },
    onScale(e) {
      let scaleX = e.scale[0]
      let scaleY = e.scale[1]
      let index = e.target.getAttribute('id');
      if (index) {
        this.layers[index].scaleX = this.tmpLayer.scaleX = scaleX;
        this.tmpLayer.percentX = Math.floor(scaleX * 100);
        this.layers[index].scaleY = this.tmpLayer.scaleY = scaleY;
        this.tmpLayer.percentY = Math.floor(scaleY * 100);
      }

      e.target.style.transform = e.transform;
    },
    setActive(index) { // делает слой активным. копируем данные слоя во временный слой для редактирования
      this.showBgGalery = false;
      this.hideProgress();
      this.clearActive();
      this.showAlert = false;
      if (this.layers[index].shadowColor !== 'transparent') {
        this.setShadow = true;
      } else {
        this.setShadow = false;
      }
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
      this.layers[index].isActive = true;
      this.tmpLayer.inUse = true;
      //
      this.alignCenter = (this.tmpLayer.textAlign === 'center') ? true : false;
    },
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
    hideControls() { // скрываем эл-ты управления
      this.showAlert = false;
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
      this.hideControls();
      this.tmpLayer.inUse = false;
      let container = document.getElementById('img-container');
      container.style.border = 'none';
      this.clearActive();
      this.showAlert = false;
      this.hideProgress();
    },
    startLoader() {
      document.body.style.cursor = 'progress';
    },
    stopLoader() {
      document.body.style.cursor = ''
    },
    clearActive() { // все слои неактивными
      this.layers.forEach(item => {
        item.isActive = false;
        item.zIndex = item.id;
      })
      this.tmpLayer.inUse = false;
      this.showAlert = true;
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0;
      while (controls[i]) {
        controls[i].style.display = 'none';
        i++;
      }
      // this.showAlert = false;
    },
    exportAll() {
      let container = document.getElementById('img-container'); // скриншот с этого блока
      this.hideControls();// убираем управляющие элементы перед скриншотом
      this.save();
      this.screenshot(container);
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
    send() { // отправка на сервер
      this.hideControls();// убираем управляющие элементы перед скриншотом
      this.startLoader();
      let container = document.getElementById('img-container');
      let progressBlock = document.querySelector('.progress');
      let progressBar = document.querySelector('.progress-bar');
      let formData;
      let $alert = document.querySelector('.alert');
      const url = '/constructor';
      // const url = 'https://alexart.houme21.ru/constructor';
      html2canvas(container, {}).then(function (canvas) {
        canvas.toBlob(function (blob) {
          formData = new FormData();
          formData.append('screen', blob, 'test.png');
          // csrf токен забит в куки в файле шаблона views/layout/constructor.php
          formData.append(readCookie('csrf_param'), readCookie('csrf_token'));
          // отправка
          /* Fetch без индикатора загрузки */
          /*let response = fetch(url, {
              method: 'POST',
              body: formData,
          }).then(function (response) {
              response.text().then(function (data) {
                  if(response.ok){
                      $alert.show().addClass('alert-success').text('Изображение загружено');
                  }else{
                      let code = response.status;
                      console.log(response)
                      let errText;
                      switch (code) {
                          case 411 : errText = 'Слишком большой файл';break;
                          case 415 : errText = 'Не распознан файл изображения';break;
                          default : '';
                      }
                      $alert.show().addClass('alert-danger').text('Ошибка ' + code + ' ' + errText);
                  }
              });
          });*/
          //
          /* AJAX с прогрессбаром */
          let xhr = new XMLHttpRequest();
          xhr.timeout = 20000;
          xhr.onreadystatechange = () => {
            if (xhr.readyState == 1) {
              progressBlock.style.visibility = 'visible';
              progressBar.classList.add('progress-bar-animated');
              if (progressBar.classList.contains('bg-danger')) {
                progressBar.classList.remove('bg-danger');
                progressBar.classList.add('bg-success');
              }
              progressBar.innerText = '';
            }
          };
          xhr.upload.onprogress = (e) => {
            let percentComplete = Math.ceil((e.loaded / e.total) * 100);
            progressBar.style.width = percentComplete + '%';
            progressBar.innerText = percentComplete + '%';
          };
          // Ждём завершения: неважно, успешного или нет
          xhr.onloadend = () => {
            if (xhr.status == 200) {
              $alert.style.visibility = 'visible';
              $alert.style.display = 'block';
              $alert.classList.add('alert-success');
              $alert.innerHTML = 'Изображение загружено';
            } else {
              let code = xhr.status;
              console.log("Ошибка " + code);
              let errText;
              switch (code) {
                case 413 :
                  errText = 'Слишком большой файл.';
                  break;
                case 415 :
                  errText = 'Не распознан файл изображения.';
                  break;
                default :
                  errText = '';
              }
              progressBar.classList.remove('bg-success');
              progressBar.classList.add('bg-danger');
              progressBar.innerText = 'Ошибка !';
              // $alert.show().addClass('alert-danger').text('Ошибка ' + code + '. ' + errText);
              $alert.style.visibility = 'visible';
              $alert.style.display = 'block';
              $alert.classList.add('alert-danger');
              $alert.innerHTML = 'Ошибка ' + code + '. ' + errText;
            }
            document.body.style.cursor = '';
            progressBar.classList.remove('progress-bar-animated');
            document.body.style.cursor = '';
          };
          xhr.open('POST', url);
          xhr.send(formData);
          //
        }, 'image/png');
      });
    },
    screenshot(container) { // собственно скриншот
      let fileName = this.strRand(12);
      html2canvas(container).then(function (canvas) {
        canvas.toBlob(function (blob) {
          // после того, как Blob создан, загружаем его
          let link = document.createElement('a');
          link.download = fileName + '.png';
          link.href = URL.createObjectURL(blob);
          link.click();
          // удаляем внутреннюю ссылку на Blob чтоб не висела впамяти
          URL.revokeObjectURL(link.href);
        }, 'image/png');
      });
    },
    strRand(len) { // случайная строка для имени файла
      let result = '';
      let words = '0123456789qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM';
      let max_position = words.length - 1;
      for (let i = 0; i < len; ++i) {
        let position = Math.floor(Math.random() * max_position);
        result = result + words.substring(position, position + 1);
      }
      return result;
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
      if(lineCount > 1){ // многострочный текст
        arr = text.split('\n');
        arr.forEach((item) => { // находим самую длинную строку в многострочном тексте
          if(item.length > maxStr.length){
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
    addText() { // добавляем текстовый слой
      this.hideProgress();
      this.initTxtWidth();
      this.initTxtHeight();
      let id;
      // назначаем id
      if (!this.layers.length) { // еще ничего не добавлено
        id = 0;
      } else {
        let arr = [];
        this.layers.forEach((item) => {
          arr.push(item.id);
        });
        id = Math.max(...arr) + 1;
      }
      //
      this.layers.push({
        id: id,
        class: 'l' + String(id),
        visibility: 'visible',
        zIndex: id,
        type: 'txt',
        fileName: '',
        content: this.defaultText,
        src: '',
        x: 40,
        y: 40,
        scaleX: 1,
        scaleY: 1,
        w: this.defaultTextWidth,
        h: this.defaultTextHeight,
        fontFamily: 'Arial',
        fontSize: this.defaultFontSize,
        color: this.defaultColor,
        interval: 0,
        lineHeight: this.defaultLineHeight,
        lineCount: 1,
        textAlign: 'left',
        rotate: 0,
        isActive: true,
        shadowX: 0,
        shadowY: 0,
        shadowR: 0,
        shadowColor: 'transparent',
      });
      this.clearActive();
      this.setActive(id);
      this.save();
    },
    addImg() { // добавление изображения
      this.hideProgress();
      this.clearActive();
      document.getElementById('imgInput').click(); // клик по input[type=file]
    },
    readFile() { // по событию onchange на input[type=file]
      let input = document.getElementById('imgInput');
      let file = input.files[0];
      //
      if (this.getFileExt(file.name) != 'png' || file.type != 'image/png') {
        alert('Недопустимый тип файла! (только PNG)');
        return;
      }
      if (file.size > maxImgSize * 1024 * 1024) {
        alert('Превышен размер файла (не более ' + maxImgSize + ' мегабайт)');
        return;
      }

      //
      async function f() {
        let reader = new FileReader();
        await reader.readAsDataURL(file);
        reader.onload = await function () {
          tmpSrc = reader.result;
          tmpFileName = file.name;
          input.value = null; // очищаем а то при добавлении одинаковых файлов трабл
          document.getElementById('insert').click(); // клик на фейковой кнопке и переход к insertImg()
        }
        reader.onerror = function () {
          alert(reader.error);
        };
      }

      //
      f();
    },
    insertImg() {
      let id = this.layers.length ? this.layers.length : 0;

      //
      async function f() {
        // финт ушами чтобы узнать реальный размер Base64 изображения
        let tmpImg = new Image();
        tmpImg.src = tmpSrc;
        tmpImg.onload = await function () {
          tmpW = tmpImg.naturalWidth;
          tmpH = tmpImg.naturalHeight;
          //
          if (tmpW > maxImgWidth) { // превышает допустимый размер - уменьшаем
            let ratio = tmpW / tmpH;
            let canvas = document.createElement('canvas');
            canvas.width = maxImgWidth;
            let height = Math.floor(maxImgWidth / ratio);
            canvas.height = height;
            var ctx = canvas.getContext("2d");
            ctx.drawImage(this, 0, 0, maxImgWidth, height);
            tmpW = maxImgWidth;
            tmpH = height;
            canvas.toBlob(function (blob) {
              let reader = new FileReader();
              reader.readAsDataURL(blob); // конвертирует Blob в base64 и вызывает onload
              reader.onload = function () {
                tmpSrc = reader.result; // url с данными
              };
            }, 'image/png');
          }
          document.getElementById('push').click(); // идем к методу pushImg()
        }
      }

      //
      f();
    },
    pushImg() { // добавляем в массив
      // получим id
      let id;
      if (!this.layers.length) { // еще ничего не добавлено
        id = 0;
      } else {
        let arr = [];
        this.layers.forEach((item) => {
          arr.push(item.id);
        });
        id = Math.max(...arr) + 1;
      }
      //
      this.layers.push({
        id: id,
        class: 'l' + String(id),
        visibility: 'visible',
        zIndex: id,
        type: 'img',
        fileName: tmpFileName,
        src: tmpSrc,
        x: 20,
        y: 20,
        scaleX: 1,
        scaleY: 1,
        w: tmpW,
        h: tmpH,
        originalW: tmpW,
        originalH: tmpH,
        rotate: 0.00001,
        isActive: true,
      });
      this.clearActive();
      this.setActive(id);
      this.save();
    },
    getFileExt(fname) { // получаем расширение файла
      return fname.slice((fname.lastIndexOf(".") - 1 >>> 0) + 2);
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
    del(index) { // удаление слоя
      this.clearTmpLayer();
      this.hideProgress();
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
        }
      }
    },
    hideProgress() { // скрываем прогресс бар загрузки
      document.querySelector('.progress').style.visibility = 'hidden';
      let alert = document.querySelector('.alert');
      alert.style.display = 'none';
      alert.style.visibility = 'hidden';
    },
    clear() { // удаление всех слоев
      this.clearTmpLayer();
      this.bgColor = null;
      // this.bgSrc = null;
      this.layers = [];
      localStorage.clear();
      this.hideProgress();
      this.showAlert = false;
    },
    resetTransform(id) { // сброс всех трансформаций слоя
      if (id >= 0) {
        document.getElementById('ltab' + String(id)).click();
        this.tmpLayer.scaleX = 1;
        this.tmpLayer.percentX = 100;
        this.tmpLayer.scaleY = 1;
        this.tmpLayer.percentY = 100;
        this.tmpLayer.rotate = 0;
        this.tmpLayer.fontSize = this.defaultFontSize;
        this.tmpLayer.interval = 0;
        this.tmpLayer.x = 20;
        this.tmpLayer.y = 20;
        this.layers[id].x = 20;
        this.layers[id].y = 20;
        //
        if (this.tmpLayer.type == 'img') {
          this.layers[id].w = this.layers[id].originalW;
          this.layers[id].h = this.layers[id].originalH;
        }
      }
    },
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
    clearBgData() { // фон приводим в значение по умолчанию
      this.bgSrc = null;
      this.bgColor = null;
      this.bgArr.forEach(item => {
        item.isActive = false;
      });
      localStorage.removeItem('bgColor');
      localStorage.removeItem('bgIndex');
      localStorage.removeItem('bgFileName');
    },
    changeBg(index, fileName) { // изменяем фоновую картинку
      this.showAlert = false;
      this.clearBgData();
      this.bgArr[index].isActive = true;
      this.defaultBg = fileName;
      localStorage.setItem('bgIndex', index);
      localStorage.setItem('bgFileName', fileName);
    },
    showBgList() { // показываем галерею фоновых картинок
      this.showBgGalery = true;
    },
    /*
    здесь меняем не фоновую картинку а попиксельно меняем один цвет на другой!!
    */
    changeBgColor() {
      if (this.bgColorChanged > 1) { // первый раз при mount overlay не убираем
        this.showAlert = false;
      }
      this.bgColorChanged += 1;
      if (this.bgColor) {
        this.bgArr.forEach(item => {
          item.isActive = false;
        });
        if (localStorage.getItem('bgIndex')) { // если есть удаляем прежние данные
          localStorage.removeItem('bgIndex');
          localStorage.removeItem('bgFileName');
        }
        // конвертация hex => rgb
        let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(this.bgColor);
        // здесь rgb вида [255,0,0]
        let color = result ? [parseInt(result[1], 16), parseInt(result[2], 16), parseInt(result[3], 16)] : null;
        let img = document.getElementById('red'); // в этой картинке заменяем красный цвет[255,0,0] на нужный
        let canvas = document.createElement("canvas");
        canvas.width = img.width;
        canvas.height = img.height;
        var ctx = canvas.getContext("2d");
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
        localStorage.setItem('bgColor', this.bgColor)
      }
    },
    setHorisontalAlign() { // выравнивание слоя по горизонтали
      this.layers[this.tmpLayer.id].x = (this.parentWidth - this.layers[this.tmpLayer.id].w) / 2;
    },
    setVerticalAlign() { // выравнивание слоя по вертикали
      this.layers[this.tmpLayer.id].y = (this.parentHeight - this.layers[this.tmpLayer.id].h) / 2;
    },
    setTextAlign(align){
        this.tmpLayer.textAlign = align;
    }
  },
  mounted() {
    if (this.layers.length && !this.tmpLayer.inUse) {
      this.showAlert = true;
    }
    // достаем сохраненные данные
    this.getLayers(); // слои
    let bgColor = localStorage.getItem('bgColor'); // цвет фона
    if (bgColor) { //есть сохраненный цвет фона
      let img = document.getElementById('red'); // в этой картинке заменяем красный цвет[255,0,0] на нужный
      img.onload = () => { // ждем загрузки иначе не пашет
        this.bgColorChanged += 1;
        this.bgColor = bgColor;
      }
    } else {
      let bgIndex = localStorage.getItem('bgIndex');
      if (bgIndex) { // есть сохранненый вариант фона из предложенных
        this.changeBg(bgIndex, localStorage.getItem('bgFileName'))
      }
    }
    this.init();
    // document.getElementById('main-overlay').style.display = 'none';
    // document.getElementById('loader').style.display = 'none';
  },
  computed: {
    getBgSrc() {
      if (!this.bgSrc) {
        return 'url(' + require('@/assets/img/' + this.defaultBg) + ')'; // это картинка фона по умолчанию
      } else {
        return 'url(' + this.bgSrc + ')'
      }

    },
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
          return 'Выберите слой'
        }
      }
    },
  },
  watch: {
    layers: {
      handler(updatedList) {
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
          this.layers[updatedList.id].textAlign = updatedList.textAlign;
          this.save();
        }

      },
      deep: true
    },
    bgColor: {
      handler() {
        this.changeBgColor();
      }
    },
  },
});
</script>
