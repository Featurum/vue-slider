# Vue Slider

> Слайдер контента для Vue.js

## Установка

``` bash

cd example/

# Установка зависимостей
npm install

# Запуск сервера (localhost:8080)
npm run dev

# Билд
npm run build
```

## Опции

### Props
| Параметры   | Тип           | По умолчанию  | Описание  |
| ----------- |:--------------| ---------|--------------|
| items       | Number        | 3        | Количество слайдов |
| margin      | Number        | 20       | Дистанция между слайдами   |
| nav         | Boolean       | false    | Навигация   |
| dots        | Boolean       | false    | Точечная навигация   |
| loop        | Boolean       | false    | Бесконечная прокрутка   |
| prevNav     | String        |          | Кнопка назад   |
| nextNav     | String        |          | Кнопка веред   |
| speed       | Number        | 300      | Скорость перелистывания (мс)   |
| timing      | String        | 'ease'   | Анимация перехода (linear, ease-in, ease-out, ease-in-out)  |
| offset      | Number        | 1        | оличество смещаемых слайдов   |
| sibling     | Boolean       | false    | Переключение слайдов соседними блоками   |
| responsive  | Object        | {}       | Отзывчивость   |


## Пример подключения

```html
<template>
  <div>
    <v-slider v-bind="options">
      <div class="item">1</div>
      <div class="item">2</div>
      <div class="item">3</div>
    </v-slider>
  </div>
</template>
<script>
  import slider from './slider.vue'

  new Vue({
    el: 'main',
    data: {
      options: {
          items: 5,
          margin: 20,
          nav: true,
          dots: true,
          loop: true,
          timing: 'cubic-bezier(0, 0.72, 0.64, 1.06)',
          offset: 1,
          prevNav: 'Туда',
          nextNav: 'Сюда',
          sibling: true,
          responsive : {
              0: {
                  items: 1
              },
              768: {
                  items: 3
              },
              999: {
                  items: 5
              }
          }
      }
    },
    components: {
        'v-slider': slider,
    }
  }
</script>
```
