<template>
    <div id="main">
        <div class="attach">
            <div class="slider_case">
                <v-slider v-bind="options">
                    <div class="item">1</div>
                    <div class="item">2</div>
                    <div class="item">3</div>
                    <div class="item">4</div>
                    <div class="item">5</div>
                    <div class="item">6</div>
                    <div class="item">7</div>
                    <div class="item">8</div>
                    <div class="item">9</div>
                    <div class="item">10</div>
                    <div class="item">11</div>
                    <div class="item">12</div>
                    <div class="item">13</div>
                    <div class="item">14</div>
                    <!-- <div class="item">15</div> -->
                </v-slider>
            </div>

            <pre>{{options}}</pre>
        </div>
    </div>
</template>

<script>
    import slider from './slider.vue'

    export default {
        name: 'main',
        
        data () {
            return {
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
            }
        },

        components: {
            'v-slider': slider,
        }
    }
</script>

<style>
    * {
        margin: 0;
        padding: 0;
        border:0;
        max-height: 99999999px;
        -webkit-font-smoothing: antialiased;


        -webkit-box-sizing: border-box;
        -moz-box-sizing: border-box;
        box-sizing: border-box;
    }

    #main {
        font-family: "Roboto", Arial, sans-serif;
        font-size: 14px;
    }

    .attach {
        width: 100%;
        max-width: 1000px;
        min-width: 480px;
        padding: 50px 15px;
        margin: 0 auto;
        position: relative;
    }

    .slider_case {
        margin-bottom: 40px;
    }

    .item {
        background: #00800b;
        text-align: center;
        padding: 100px 0;
        color: #fff;
        font-size: 26px;
        font-style: italic;
        font-weight: 300;
    }

    .item.active {
        background: #ff3f4d;
    }
    
    .item.super {
        background: #04a0c5;
    }

    pre {
        border:1px solid #eee;
        padding:15px;
        background: #f5f5f5;
    }



</style>