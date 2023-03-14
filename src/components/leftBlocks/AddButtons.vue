<template>
  <div class="d-flex">
    <input
      id="imgInput"
      type="file"
      @input="readFile"
      ref="imgInp"
      style="display: none"
    />
    <div class="img-tab btn btn-light d-flex" @click="addImg">
      <div class="img-tab-icon"></div>
      <div class="tab-title">+ изображение</div>
    </div>
    <div class="text-tab btn btn-light d-flex" @click="addText">
      <div class="text-tab-icon"></div>
      <div class="tab-title">+ текст</div>
    </div>
  </div>
</template>
<script>
import { settings } from '@/_config'
export default {
  emits: ['addButtonsEmit'],
  props: {
    tmpLayer: {
      type: Object,
      required: true
    },
    layers: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      tmpW: undefined,
      tmpH: undefined,
      tmpSrc: undefined,
      tmpBg: undefined,
      tmpFileName: undefined,
      //
      defaultText: 'ваш текст',
      defaultColor: '#0000ff',
      defaultFontSize: settings.fontSize, // 24
      defaultLineHeight: this.defaultFontSize,
      defaultTextAlign: 'left',
      defaultFontFamily: 'Arial',
      defaultTextWidth: 0,
      defaultTextHeight: 0
    }
  },
  methods: {
    addImg() {
      // добавление изображения
      this.$refs.imgInp.click() // клик по скрытому input[type=file]
    },
    readFile() {
      // по событию onchange на input[type=file]
      this.$emit('addButtonsEmit', { loader: true })
      let input = this.$refs.imgInp
      let file = input.files[0]
      //
      if (this.getFileExt(file.name) != 'png' || file.type != 'image/png') {
        alert('Недопустимый тип файла! (только PNG)')
        this.$emit('addButtonsEmit', { loader: false })
        return
      }
      if (file.size > settings.maxImgSize * 1024 * 1024) {
        alert(
          'Превышен размер файла (не более ' +
            settings.maxImgSize +
            ' мегабайт)'
        )
        this.$emit('addButtonsEmit', { loader: false })
        return;
      }
      // чтобы не терять контекст оборачиваем в стрелочную функцию
      (async () => {
        let reader = new FileReader()
        await reader.readAsDataURL(file)
        reader.onload = async () => {
          // console.log(this);
          this.tmpSrc = await reader.result
          this.tmpFileName = await file.name
          input.value = null // очищаем а то при добавлении одинаковых файлов трабл
          this.insertImg()
        }
        reader.onerror = () => {
          alert(reader.error)
          this.loader = false
        }
      })()
    },
    insertImg() {
      let id = this.layers.length ? this.layers.length : 0;
      // чтобы не терять контекст оборачиваем в стрелочную функцию
      (async () => {
        // финт ушами чтобы узнать реальный размер Base64 изображения
        let tmpImg = new Image()
        tmpImg.src = this.tmpSrc
        tmpImg.onload = () => {
          this.tmpW = tmpImg.naturalWidth
          this.tmpH = tmpImg.naturalHeight
          //
          if (this.tmpW > settings.maxImgWidth) {
            // превышает допустимый размер - уменьшаем
            let ratio = this.tmpW / this.tmpH
            let canvas = document.createElement('canvas')
            canvas.width = settings.maxImgWidth
            let height = Math.floor(settings.maxImgWidth / ratio)
            canvas.height = height
            let ctx = canvas.getContext('2d')
            ctx.drawImage(tmpImg, 0, 0, settings.maxImgWidth, height)
            this.tmpW = settings.maxImgWidth
            this.tmpH = height
            canvas.toBlob(blob => {
              let reader = new FileReader()
              reader.readAsDataURL(blob) // конвертирует Blob в base64 и вызывает onload
              reader.onload = () => {
                this.tmpSrc = reader.result // url с данными
              }
              reader.onerror = () => {
                console.log(reader.error)
              }
            }, 'image/png')
          }
          this.pushImg()
        }
      })()
    },
    pushImg() { // добавляем в массив слоев
      // получим id
      let id
      if (!this.layers.length) {
        // еще ничего не добавлено
        id = 0
      } else { // из имеющихся следующий по порядку
        let arr = []
        this.layers.forEach(item => {
          arr.push(item.id)
        })
        id = Math.max(...arr) + 1
      }
      //
      this.layers.push({
        id: id,
        class: 'l' + String(id),
        visibility: 'visible',
        zIndex: id,
        type: 'img',
        fileName: this.tmpFileName,
        src: this.tmpSrc,
        x: 20,
        y: 20,
        scaleX: 1,
        scaleY: 1,
        w: this.tmpW,
        h: this.tmpH,
        originalW: this.tmpW,
        originalH: this.tmpH,
        rotate: 0.00001,
        isActive: true
      })
      this.clearActive()
      this.setActive(id)
      this.$emit('addButtonsEmit', { loader: false })
    },
    clearActive() {
      // все слои неактивными
      this.layers.forEach(item => {
        item.isActive = false
        item.zIndex = item.id
      })
      this.tmpLayer.inUse = false
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0
      while (controls[i]) {
        controls[i].style.display = 'none'
        i++
      }
    },
    setActive(index) {
      // делает слой активным. копируем данные слоя во временный слой для редактирования
      this.clearActive()
      let controls = document.getElementsByClassName('moveable-control-box')
      let i = 0
      while (controls[i]) {
        // эл-ты управления оставляем только для активного слоя
        if (i == index) {
          controls[i].style.display = 'block'
        }
        i++
      }
      //
      this.tmpLayer.id = this.layers[index].id
      this.tmpLayer.class = this.layers[index].class
      this.tmpLayer.visibility = this.layers[index].visibility
      this.tmpLayer.type = this.layers[index].type
      this.tmpLayer.fileName = this.layers[index].fileName
      this.tmpLayer.content = this.layers[index].content
      this.tmpLayer.src = this.layers[index].src
      this.tmpLayer.x = this.layers[index].x
      this.tmpLayer.y = this.layers[index].y
      this.tmpLayer.scaleX = this.layers[index].scaleX
      //
      if (this.layers[index].scaleX != 1) {
        // вычисляем трансформацию в процентах для показа в ползунках input[type=range]
        this.tmpLayer.percentX = Math.floor(this.layers[index].scaleX * 100)
      }
      if (this.layers[index].scaleY != 1) {
        this.tmpLayer.percentY = Math.floor(this.layers[index].scaleY * 100)
      }
      //
      this.tmpLayer.scaleY = this.layers[index].scaleY
      this.tmpLayer.w = this.layers[index].w
      this.tmpLayer.h = this.layers[index].h
      this.tmpLayer.fontFamily = this.layers[index].fontFamily
      this.tmpLayer.fontSize = this.layers[index].fontSize
      this.tmpLayer.interval = this.layers[index].interval
      this.tmpLayer.lineHeight = this.layers[index].lineHeight
      this.tmpLayer.textAlign = this.layers[index].textAlign
      this.tmpLayer.lineCount = this.layers[index].lineCount
      this.tmpLayer.color = this.layers[index].color
      this.tmpLayer.rotate = this.layers[index].rotate
      //
      this.tmpLayer.shadowX = this.layers[index].shadowX
      this.tmpLayer.shadowY = this.layers[index].shadowY
      this.tmpLayer.shadowR = this.layers[index].shadowR
      this.tmpLayer.shadowColor = this.layers[index].shadowColor
      this.tmpLayer.isShadow =
        this.layers[index].type === 'txt' &&
        this.layers[index].shadowColor !== 'transparent'
      this.layers[index].isActive = true
      this.tmpLayer.inUse = true
      //
      this.alignCenter = this.tmpLayer.textAlign === 'center' ? true : false
    },
    getFileExt(fname) {
      // получаем расширение файла
      return fname.slice(((fname.lastIndexOf('.') - 1) >>> 0) + 2)
    },
    addText() {
      // добавляем текстовый слой
      let id
      // назначаем id
      if (!this.layers.length) {
        // еще ничего не добавлено
        id = 0
      } else {
        let arr = []
        this.layers.forEach(item => {
          arr.push(item.id)
        })
        id = Math.max(...arr) + 1
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
        fontFamily: this.defaultFontFamily,
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
        isShadow: false
      })
      this.clearActive()
      this.setActive(id)
    }
  }
}
</script>
