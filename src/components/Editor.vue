<template>
  <div class="editor">
    <div id="img-container" :style="{backgroundImage: getBgSrc}">
      <div v-if="layers.length">
        <div v-for="(layer, index) in layers" :key="index">
          <!--     текст     -->
          <div v-if="layer.type === 'txt'">
            <div :id="index" class="contenteditable" :class="[layer.class, noBorder ? '' : 'brd']"
                 :style="{transform: 'translate(' + layer.x + 'px, ' + layer.y + 'px) ' + 'rotate(' + layer.rotate + 'deg) ' + 'scale(' + layer.scaleX + ', ' + layer.scaleY + ')', fontFamily: layer.fontFamily, color: layer.color, fontSize: layer.fontSize + 'px', letterSpacing: layer.interval + 'px', visibility: layer.visibility, zIndex: layer.zIndex, textShadow: layer.shadowX + 'px ' + layer.shadowY + 'px ' + layer.shadowR + 'px ' + layer.shadowColor, lineHeight: layer.lineHeight + 'px', textAlign: layer.textAlign}">
              {{ layer.content }}
            </div>
            <moveable
              :target="['.' + layer.class]"
              :throttleDrag="1"
              :edgeDraggable="false"
              :startDragRotate="0"
              :throttleDragRotate="0"
              className="moveable"
              :draggable=true
              :scalable=true
              :rotatable=true
              @drag="onDrag"
              @scale="onScale"
              @rotate="onRotate"
            />
          </div>
          <!--     картинка     -->
          <div v-else>
            <div :id="index" class="imgL" :class="[layer.class, noBorder ? '' : 'brd']"
                 :style="{backgroundImage: 'url(' + layer.src + ')', width: layer.w + 'px', height: layer.h + 'px', transform: 'translate(' + layer.x + 'px, ' + layer.y + 'px) ' + 'rotate(' + layer.rotate + 'deg) ' + 'scale(' + layer.scaleX + ', ' + layer.scaleY + ')', visibility: layer.visibility, zIndex: layer.zIndex}">
            </div>
            <moveable
              :target="['.' + layer.class]"
              :throttleDrag="1"
              :edgeDraggable="false"
              :startDragRotate="0"
              :throttleDragRotate="0"
              className="moveable"
              :draggable=true
              :scalable=true
              :rotatable=true
              @drag="onDrag"
              @scale="onScale"
              @rotate="onRotate"
            />
          </div>
          <!--          <div id="zatyk"></div>&lt;!&ndash; перекрывает все слои кроме активного &ndash;&gt;-->
        </div>
      </div>
      <h1 v-if="showAlert" id="layers-over">{{ alertText }}</h1>
      <loader v-show="loader" />
    </div>
    <!--    -->
  </div>
</template>
<script>
import Moveable from "vue3-moveable";
import Loader from '@/components/Loader';
export default {
  components: {
    Moveable,
    Loader,
  },
  props: {
    layers: {
      type: Array,
      required: true,
    },
    tmpLayer: {
      type: Object,
      required: true,
    },
    defaultBg: {
      type: String,
      required: true,
    },
    bgSrc: {
      // type: String,
      required: true,
    },
    showAlert: {
      type: Boolean,
      default: true,
    },
    loader: {
      type: Boolean,
      default: false,
    },
    alertText: {
      type: String,
      default: 'Выберите слой!',
    },
    noBorder: {
      type: Boolean,
      default: false,
    }
  },
  methods: {
    onDrag(e) {
      try {
        let index = e.inputEvent.target.id;
        if (this.layers[index].isActive) { // позволяем двигать только активный слой
          this.layers[index].x = e.translate[0];
          this.layers[index].y = e.translate[1];
          e.target.style.transform = e.transform;
        }
      } catch {
      }
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
  },
  computed: {
    getBgSrc() {
      if (!this.bgSrc) {
        return 'url(' + require('@/assets/img/' + this.defaultBg) + ')'; // это картинка фона по умолчанию
      } else {
        return 'url(' + this.bgSrc + ')'
      }

    },
  },
}
</script>
