<template>
    <div class="v_slider">
        <div class="v_slider__list" ref="list">
            <div class="v_slider__track" ref="track" :style="{width: width.track + 'px', transform: 'translate(-' + transform + 'px)', transition: 'transform ' + settings.timing + ' ' + settings.speed + 'ms'}">
                <slot></slot>
            </div>
        </div>
        <ul v-if="dots" class="v_slider__dots">
            
            <li v-for="n in numDot" @click="setDot(n)" :class="{'active': n == numDotActive}">
                <span></span>
            </li>
        </ul>

        <button v-if="nav" class="v_slider__prev" v-html="prevNav" @click="prevSlide"></button>
        <button v-if="nav" class="v_slider__next" v-html="nextNav" @click="nextSlide"></button>
    </div>
</template>

<script>
    export default {
        name: 'slider',
        props: {
            // Количество слайдов на странице
            items: {
                type: Number,
                default: 3
            },

            // Отступы
            margin: {
                type: Number,
                default: 20
            },

            // Навигация
            nav: {
                type: Boolean,
                default: false
            },

            // Точечная навигация
            dots: {
                type: Boolean,
                default: false
            },

            // Перемотка
            loop: {
                type: Boolean,
                default: false
            },

            // Кнопка назад
            prevNav: {
                type: String,
                default: ''
            },

            // Кнопка веред
            nextNav: {
                type: String,
                default: ''
            },

            // Скорость перелистывания (мс)
            speed: {
                type: Number,
                default: 300
            },

            // Анимация перехода (linear, ease-in, ease-out, ease-in-out)
            timing: {
                type: String,
                default: 'ease'
            },

            // Количество смещаемых слайдов
            offset: {
                type: Number,
                default: 1
            },

            // Переключение слайдов соседними блоками
            sibling: {
                type: Boolean,
                default: false
            },

            // Адаптивность
            responsive: {
                type: Object,
                default: {}
            },

        },

        data () {
            return {
                el: {
                    track: null,
                    slides: null,
                },

                width: {
                    document: 0,
                    container: 0,
                    slide: 0,
                    track: 0
                },

                // Первый слайд
                itemActive: 0,

                // Общее количество реальных слайдов
                numItemReal: 0,

                // Общее количество слайдов с клонами
                numItemAll: 0,

                // Список медиа запросов
                breakpoints: [],

                // Количество
                numDot: 0,

                // Активный дот
                numDotActive: 0,

                // Сдвиг по оси Y
                transform: 0,

                // Свойства слайдера
                settings: {
                    items: this.items,
                    margin: this.margin,
                    nav: this.nav,
                    dots: this.dots,
                    loop: this.loop,
                    prevNav: this.prevNav,
                    nextNav: this.nextNav,
                    speed: this.speed,
                    timing: this.timing,
                    offset: this.offset,
                    responsive: this.responsive
                },

                // Флаг назатия мыши
                mouseDown: false,
                
                // Дистанция для свайпа
                swipeDistance: 50,

                // Дистанция перемещения курсора
                dragDistance: 0 
            }
        },

        mounted () {
            this.$nextTick(function () {

                // Объект контейнера
                this.el.list = this.$refs.list

                // Объект трека
                this.el.track = this.$refs.track
                
                // Объекты слайдов
                this.el.slides = this.el.track.children

                // Вычисление количества слайдов
                this.numItemReal = this.el.slides.length

                // Присваиваем класс слайдам
                for (let i = 0; i < this.numItemReal; ++i) {
                    this.el.slides[i].classList.add('v_slider__item')
                }

                // Добавляем клоны
                if (this.settings.loop) {
                    let lastSlide = this.el.track.getElementsByClassName('v_slider__item')
                    for (let i = 0; i < this.numItemReal; ++i) {
                        let clone = lastSlide[i].cloneNode(true)
                        clone.classList.add('cloned')
                        this.el.track.insertBefore(clone, this.el.slides[this.numItemReal - 1 + i].nextSibling)
                    }
                    for (let i = this.numItemReal; i > 0; --i) {
                        let clone = lastSlide[this.numItemReal - 1].cloneNode(true)
                        clone.classList.add('cloned')
                        this.el.track.insertBefore(clone, this.el.slides[0])
                    }
                }

                // Вычисляем общее количество слайдов
                if (this.settings.loop) {
                    this.numItemAll = this.numItemReal * 3
                } else {
                    this.numItemAll = this.numItemReal
                }

                // Обновление при изменении экрана
                window.addEventListener('resize', this.getWidthDocument)

                // Вешаем обработчиски событий на действия мыши
                if ('ontouchstart' in window) {
                    this.el.track.addEventListener('touchstart', this.handleMouseDown)
                    this.el.track.addEventListener('touchend', this.handleMouseUp)
                    this.el.track.addEventListener('touchmove', this.handleMouseMove)
                } else {
                    this.el.track.addEventListener('mousedown', this.handleMouseDown)
                    this.el.track.addEventListener('mouseup', this.handleMouseUp)
                    this.el.track.addEventListener('mousemove', this.handleMouseMove)
                }

                // Вычисляем ширину окна
                this.getWidthDocument()
            })
        },

        beforeDestroy () {
            window.removeEventListener('resize', this.getWidthDocument)

            if ('ontouchstart' in window) {
                this.el.track.removeEventListener('touchstart', this.handleMouseDown)
                this.el.track.removeEventListener('touchend', this.handleMouseUp)
                this.el.track.removeEventListener('touchmove', this.handleMouseMove)
            } else {
                this.el.track.removeEventListener('mousedown', this.handleMouseDown)
                this.el.track.removeEventListener('mouseup', this.handleMouseUp)
                this.el.track.removeEventListener('mousemove', this.handleMouseMove)
            }
        },

        methods: {
            // Размер окна браузера
            getWidthDocument () {
                this.width.document = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth
            },

            // Размеры слйдов и трека
            getWidth () {
                this.width.container = this.el.list.clientWidth
                if(this.settings.items == 1) {
                    this.width.slide = this.width.container
                    this.width.track = this.width.container * this.numItemAll
                } else {
                    this.width.slide = (this.width.container / this.settings.items) - ((this.settings.items - 1) * (this.settings.margin) / this.settings.items),
                    this.width.track = (this.width.container + this.settings.margin) * this.numItemAll
                }
            },

            reload () {
                // вычисляем брейкпоинты
                if(this.responsive){
                    this.breakpoints = Object.keys(this.responsive)
                }

                // Устанавливаем свойства при отзывчивости
                if(this.breakpoints){
                    this.breakpoints.forEach((width) => {
                        if(width <= this.width.document){
                            for (let key in this.responsive[width]) {
                                this.settings[key] = this.responsive[width][key]
                            }
                        }
                    })
                }

                // Вычисляем размеры 
                this.getWidth()

                // Вычисляем отступы
                if(this.settings.items == 1) {
                    this.settings.margin = 0
                } else{
                    this.settings.margin = this.margin
                }

                // Устанавливаем размеры
                for (let i = 0; i < this.numItemAll; ++i) {
                    this.el.slides[i].style.width = this.width.slide + 'px'
                    this.el.slides[i].style.marginRight = this.settings.margin + 'px'
                }

                // Удаляем активные слайды
                for (let i = 0; i < this.numItemAll - 1; ++i) {
                    this.el.slides[i].classList.remove('active')
                }

                // Устанавливаем трек и назначаем активные слайды
                if (this.settings.loop) {
                    this.transform = this.numItemReal * (this.width.slide + this.settings.margin)
                } else {
                    this.transform = 0
                }

                // Устанавливаем активные слайды
                this.addActiveClass(this.itemActive)

                // Вычисляем количество дотов
                this.numDot = Math.ceil(this.numItemReal / this.settings.items)
            },

            // Установка активнго слайда
            addActiveClass (value) {
                if (this.settings.loop) {
                    value += this.numItemReal
                }

                let superClass = value + Math.floor(((this.settings.items / 2)))
                this.el.slides[superClass].classList.add('super')

                for (let i = 0; i < this.settings.items; ++i) {
                    this.el.slides[value].classList.add('active')
                    ++value
                }
            },


            // Перемещение слайдера
            setSlide (n, transition = true) {
                // Удаляем активный слайд
                for (let i = 0; i < this.numItemAll - 1; ++i) {
                    this.el.slides[i].classList.remove('active', 'super')
                }

                // Перелистывание слайдера
                if (this.settings.loop) {
                    this.transform = (n + this.numItemReal - 1) * (this.width.slide + this.settings.margin)
                } else {
                    this.transform = n * (this.width.slide + this.settings.margin)
                }
                this.transform += (this.width.slide + this.settings.margin)

                // Отключение анимации
                if (!transition) {
                    this.settings.speed = 0
                } else {
                    this.settings.speed = this.speed
                }

                if (this.settings.loop) {
                    if (n < 0) {
                        this.itemActive = this.numItemReal - 1
                        setTimeout(() => {
                            this.setSlide(this.itemActive, false)
                        }, this.speed)
                    }
                    else if (n >= this.numItemReal) {
                        this.itemActive = 0
                        setTimeout(() => {
                            this.setSlide(0, false)
                        }, this.speed)
                    } else {
                        this.itemActive = n
                    }
                    this.addActiveClass(this.itemActive)
                } else {
                    if (n < 0) {
                        this.itemActive = 0
                    } else if (n >= this.numItemReal - this.settings.items) {
                        this.itemActive = this.numItemReal - this.settings.items
                    } else {
                        this.itemActive = n
                    }

                    this.transform = this.itemActive * (this.width.slide + this.settings.margin)
                    this.addActiveClass(this.itemActive)
                }
            },

            // Следующий слайдер
            nextSlide () {
                this.setSlide(this.itemActive + this.settings.offset)
            },

            // Предидущий слайдер
            prevSlide () {
                this.setSlide(this.itemActive - this.settings.offset)
            },

            // Переключение дотами
            setDot(n) {
                if (this.settings.loop) {
                    this.setSlide((n - 1) * (this.settings.items))
                } else {
                    if(n == this.numDot) {
                        let remainder = this.numItemReal % this.settings.items
                        if(remainder == 0) {
                            this.setSlide((n - 1) * (this.settings.items))
                        } else {
                            this.setSlide(this.numItemReal - this.settings.items)
                        }
                    } else {
                        this.setSlide((n - 1) * (this.settings.items))
                    }
                }
            },


            // Действия мыши
            handleMouseDown (e) {
                if (!e.touches) {
                    e.preventDefault()
                }
                this.mouseDown = true
                this.dragStartX = ('ontouchstart' in window) ? e.touches[0].clientX : e.clientX
                this.dragStartY = ('ontouchstart' in window) ? e.touches[0].clientY : e.clientY
            },
            handleMouseMove (e) {
                let positionX = ('ontouchstart' in window) ? e.touches[0].clientX : e.clientX
                let positionY = ('ontouchstart' in window) ? e.touches[0].clientY : e.clientY
                let dragDistanceX = Math.abs(positionX - this.dragStartX)
                let dragDistanceY = Math.abs(positionY - this.dragStartY)
                if (dragDistanceX > 3 * dragDistanceY) {
                    this.dragDistance = positionX - this.dragStartX
                    this.disableScroll()
                }
            },
            handleMouseUp () {
                this.mouseDown = false
                this.enableScroll()
            },
            disableScroll () {
                document.ontouchmove = function (e) {
                    e.preventDefault()
                }
            },
            enableScroll () {
                document.ontouchmove = function (e) {
                    return true
                }
            }
        },

        computed: {
            documentWidth: function () {
                return this.width.document
            }
        },

        watch: {
            documentWidth () {
                this.reload()
            },

            dragDistance () {
                if (!this.mouseDown) {
                    return
                }
                if (this.dragDistance > this.swipeDistance) {
                    this.prevSlide()
                    this.handleMouseUp()
                }
                if (this.dragDistance < -1 * this.swipeDistance) {
                    this.nextSlide()
                    this.handleMouseUp()
                }
            }
        },

        updated: function () {
            this.$nextTick(function () {
                // Вычисляем активный дот
                if (this.settings.loop) {
                    this.numDotActive = Math.ceil(this.itemActive / this.settings.items + 0.1)
                } else {
                    let remainder = this.numItemReal % this.settings.items
                    if(remainder == 0) {
                        this.numDotActive = Math.ceil(this.itemActive / this.settings.items + 0.1)
                    } else {
                        if(this.itemActive >= this.numItemReal - this.settings.items) {
                            this.numDotActive = this.numDot
                        } else {
                            this.numDotActive = Math.ceil(this.itemActive / this.settings.items + 0.1)
                        }
                    }
                }


                // Переключение соседними слайдами
                if(this.sibling) {
                    this.el.activeItem = this.$el.getElementsByClassName('v_slider__track')[0]

                    for (let i = 0; i < this.el.activeItem.children.length; ++i) {
                        this.el.activeItem.children[i].removeEventListener('click', this.nextSlide)
                        this.el.activeItem.children[i].removeEventListener('click', this.prevSlide)
                    }

                    this.el.activeItem = this.el.activeItem.getElementsByClassName('super')[0]
                    this.el.activeItem.nextElementSibling.addEventListener('click', this.nextSlide)
                    this.el.activeItem.previousElementSibling.addEventListener('click', this.prevSlide)
                }  
            })
        }
    }
</script>


<style>
    .v_slider { overflow-x: hidden; text-align: center; }
    .v_slider__list { width: 100%; min-height: 1px; float: left; display: block; position: relative; margin-bottom: 40px; }
    .v_slider__track { position: relative; -ms-touch-action: pan-Y; -moz-backface-visibility: hidden; }
    .v_slider__track:after { content:""; display: block; clear: both; }
    .v_slider__item { min-height: 1px; float: left; -webkit-backface-visibility: hidden; -webkit-touch-callout: none; }
    .v_slider__item.clone { z-index: 2; }
    .v_slider__dots { width: 100%; float: left; text-align: center; margin-bottom: 20px; }
    .v_slider__dots li { padding: 8px; display: inline-block; cursor: pointer; }
    .v_slider__dots li span { width: 8px; height: 8px; background: #dde2e6; float: left;  border-radius: 50%; }
    .v_slider__dots li span:hover { background: #fac301; }
    .v_slider__dots li.active span { background: #ff3f4d; }
    
    .v_slider__prev,
    .v_slider__next { outline: none; background:transparent; padding: 10px 15px; border:1px solid #ff3f4d; color:#ff3f4d; font-size: 20px; opacity: 0.9; }
    .v_slider__prev:hover,
    .v_slider__next:hover { border-color: #04a0c5; color:#04a0c5; cursor: pointer; }

</style>