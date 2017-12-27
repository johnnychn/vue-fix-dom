<template>
    <div ref="parent" style="width: 100%;height: 100%;overflow: hidden;position: relative">
        <div ref="child" :style="{'width':width,height:height}">
            <slot>

            </slot>
        </div>
    </div>
</template>
<style>
    .IIV::-webkit-media-controls-play-button,
    .IIV::-webkit-media-controls-start-playback-button {
        opacity: 0;
        pointer-events: none;
        width: 5px;
    }
</style>


/*!
* vue-keyframe-animation v0.0.1 (https://github.com/johnnyGoo/vue-movieclip)
* Author: Johnny chen
*
* Copyright 2013-2016 Johnny chen
*/
<script>
    function _cssToDom(el, obj) {
        if (!el) {
            return
        }
        for (let i in obj) {
            if (el.style) {
                el.style[i] = obj[i];
            }
        }
    };

    function fixCss(name, attr) {
        let cssObj = {};
        if (!attr || attr === '') {
            return cssObj;
        }

        cssObj[name] = attr;
        cssObj['-webkit-' + name] = attr;
        cssObj['-moz-' + name] = attr;
        cssObj['-ms-' + name] = attr;
        cssObj['-o-' + name] = attr;
        return cssObj;
    }

    // 注册
    export default {
        name: 'vue-fullscreen-video',
        // 声明 props
        props: {
            align: {
                type: String,
                default: 'default'
            },
            width: {
                type: String,
                default: 'auto'
            },
            height: {
                type: String,
                default: 'auto'
            },
            scroll: {
                type: Boolean,
                default: false
            }
        },
        data: function () {
            return {
                child: null, parent: null
            }
        },
        watch: {
            align: function (ov, nv) {
                this.update();
            },
            width: function (ov, nv) {
                this.update();
            },
            height: function (ov, nv) {
                this.update();
            }
        },
        methods: {
            update: function () {
                _cssToDom(this.child,fixCss('transform-origin','0% 0%'));
                _cssToDom(this.child,fixCss('transform','translate(0,0) scale(1,1)'));

                let width = this.parent.clientWidth;
                let height = this.parent.clientHeight;
                let radio = width / height;
                let cut_w, cut_h;

                if (this.width === 'auto') {
                    cut_w = this.child.clientWidth;
                } else {
                    cut_w = Number(this.width.replace('px', ''))
                }
                if (this.width === 'auto') {
                    cut_h = this.child.clientHeight;
                } else {
                    cut_h = Number(this.height.replace('px', ''))
                }


                let radio2 = cut_w / cut_h;
                let end_w, end_h;

                switch (this.align) {
                    case 'crop':
                        if (radio2 > radio) {
                            end_h = height;
                            end_w = end_h * radio2;
                        } else {
                            end_w = width;
                            end_h = end_w / radio2;
                        }
                        break;
                    default:
                        if (radio2 > radio) {
                            end_w = width;
                            end_h = end_w / radio2;

                        } else {
                            end_h = height;
                            end_w = end_h * radio2;
                        }
                        break;
                }
                end_w=Math.ceil(end_w);
                end_h=Math.ceil(end_h);

                let mov_x=-Math.ceil((end_w - width) / 2) + 'px';
                let mov_y= -Math.ceil((end_h - height) / 2) + 'px';
                let scale=end_w/cut_w;


                _cssToDom(this.child,fixCss('transform-origin','0% 0%'));

                _cssToDom(this.child,fixCss('transform','translate('+mov_x+','+mov_y+') scale('+scale+','+scale+')'));
                //
                // this.child.width = end_w + 'px';
                // this.child.height = end_h + 'px';

            }
        },


        mounted: function () {
            let self = this;
            this.parent = this.$refs.parent;
            this.child = this.$refs.child;
            self.update();
            window.onresize = function () {
                self.update();
            };
            if (false === this.scroll) {
                this.parent.setAttribute('ontouchmove', 'event.preventDefault();');
            }


        }


    }


</script>