<template>
  <div class="d-flex justify-content-between" style="margin-top: 1em">
    <button class="btn btn-success" @click="exportAll">сохранить</button>
    <button class="btn btn-secondary" @click="hideBorder">предпросмотр</button>
    <button id="clear-btn" class="btn btn-danger" @click="clear">
      <i class="fa fa-trash"> </i>Удалить все
    </button>
  </div>
  <br />
  <button class="btn btn-success" @click="send">Отправить</button>
  <br />
  <template v-if="percentage">
    <br />
    <progress-bar :bootstrapClass="progressClass" :percentage="percentage" />
  </template>
  <h3 :style="{ color: success ? 'green' : 'red' }">{{ statusText }}</h3>
</template>

<script>
// доставание cookie
function readCookie(name) {
  const matches = document.cookie.match(
    new RegExp(
      '(?:^|; )' +
        name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, '\\$1') +
        '=([^;]*)'
    )
  )
  return matches ? decodeURIComponent(matches[1]) : undefined
}

//
import html2canvas from '@/assets/js/html2canvas'
import axios from 'axios';
import ProgressBar from '@/components/leftBlocks/ProgressBar.vue'
export default {
  components: {
    ProgressBar
  },
  emits: ['saveButtonsEmit'],
  data() {
    return {
      percentage: 0,
      success: true,
      statusText: ''
    }
  },
  methods: {
    hideBorder() {
      // убираем управляющие элементы перед скриншотом
      this.$emit('saveButtonsEmit', { clear: false })
    },
    exportAll() {
      this.$emit('saveButtonsEmit', { clear: false })
      let container = document.getElementById('img-container')
      this.screenshot(container)
    },
    screenshot(container) {
      // собственно скриншот и сохранение локально на комп
      html2canvas(container).then((canvas) => {
        canvas.toBlob((blob) => {
          // после того, как Blob создан, загружаем его
          let link = document.createElement('a')
          let fileName = this.strRand(12)
          link.download = fileName + '.png'
          link.href = URL.createObjectURL(blob)
          link.click()
          // удаляем внутреннюю ссылку на Blob чтоб не висела впамяти
          URL.revokeObjectURL(link.href)
        }, 'image/png')
      })
    },
    strRand(len) {
      // случайная строка для имени файла
      let result = ''
      let words =
        '0123456789qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM'
      let max_position = words.length - 1
      for (let i = 0; i < len; ++i) {
        let position = Math.floor(Math.random() * max_position)
        result = result + words.substring(position, position + 1)
      }
      return result
    },
    async send() {
      // отправка на сервер
      this.percentage = 0
      this.statusText = ''
      document.body.style.cursor = 'progress'
      this.$emit('saveButtonsEmit', { clear: false })
      let container = document.getElementById('img-container') // скриншот с этого блока
      let formData
      const url = '/constructor'

      await html2canvas(container, {}).then(canvas => {
        canvas.toBlob(blob => {
          formData = new FormData()
          formData.append('screen', blob, 'test.png')
          formData.append('_token', readCookie('csrf'))
          /* axios */
          axios({
            method: 'POST',
            url: url,
            data: formData,
            headers: {
              'Content-Type': 'multipart/form-data'
            },
            onUploadProgress: progressEvent => {
              this.percentage = parseInt(
                Math.round((progressEvent.loaded / progressEvent.total) * 100)
              )
            }
          })
            .then(response => {
              if (response.data.success) {
                this.success = true
                this.statusText = 'Успешно!'
              } else {
                this.success = false
                // console.log('here1');
                this.statusText = 'Что то пошло не так...'
              }
            })
            .catch(error => {
              this.success = false
              let errText
              if (error.response.status == 422) {
                let errors = error.response.data.errors
                errText = errors.screen[0]
              } else if (error.response.status == 403) {
                errText = 'Требуется авторизация!'
              } else if (error.response.status == 500) {
                errText = 'Ошибка сервера!'
              } else {
                errText = `Ошибка ${error.response.status} ${error.response.statusText}!`
              }
              this.statusText = errText
              console.log(error.response)
            })
        }, 'image/png')
      })
      document.body.style.cursor = ''
    },
    clear() {
      // удаление всех слоев
      this.$emit('saveButtonsEmit', { clear: true })
    }
  },
  computed: {
    progressClass() {
      return this.success ? 'bg-success' : 'bg-danger'
    }
  }
}
</script>
