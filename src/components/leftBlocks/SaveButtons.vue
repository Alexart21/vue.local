<template>
  <div class="d-flex justify-content-between" style="margin-top: 1em">
    <button class="btn btn-success" @click="exportAll">сохранить</button>
    <button class="btn btn-secondary" @click="hideBorder">предпросмотр</button>
    <button id="clear-btn" class="btn btn-danger" @click="clear"><i class="fa fa-trash"> </i>Очистить все</button>
  </div>
  <br>
  <button class="btn btn-success" @click="send">Отправить</button>
</template>
<script>
// доставание cookie
function readCookie(name) {
  const matches = document.cookie.match(new RegExp(
    "(?:^|; )" + name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, '\\$1') + "=([^;]*)"
  ));
  return matches ? decodeURIComponent(matches[1]) : undefined;
}

//
import html2canvas from "@/assets/js/html2canvas";

export default {
  emits: ['saveButtonsEmit'],
  methods: {
    hideBorder() { // убираем управляющие элементы перед скриншотом
      this.$emit('saveButtonsEmit', {clear: false});
    },
    exportAll() {
      this.$emit('saveButtonsEmit', {clear: false});
      // this.save();
      let container = document.getElementById('img-container');
      this.screenshot(container);
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
    send() { // отправка на сервер
      console.log('send');
      document.body.style.cursor = 'progress';
      this.$emit('saveButtonsEmit', {clear: false});
      let container = document.getElementById('img-container'); // скриншот с этого блока
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
          // это для YII2
          // formData.append(readCookie('csrf_param'), readCookie('csrf_token'));
          // это для Laravel
          formData.append('_token', readCookie('csrf'));
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
              let errText;
              // это было для YII2
              /*
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
              */
             // Для Laravel
             if(code === 422){
                let resp = JSON.parse(xhr.response)
                errText = resp.errors.screen[0];
             }
              progressBar.classList.remove('bg-success');
              progressBar.classList.add('bg-danger');
              progressBar.innerText = 'Ошибка !';
              // $alert.show().addClass('alert-danger').text('Ошибка ' + code + '. ' + errText);
              $alert.style.visibility = 'visible';
              $alert.style.display = 'block';
              // loader.style.display = 'none';
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
      // this.stopLoader();
    },
    clear() { // удаление всех слоев
      this.$emit('saveButtonsEmit', {clear: true});
    },
  },
}
</script>
