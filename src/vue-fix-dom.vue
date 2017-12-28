<template>
    <div ref="parent" style="width: 100%;height: 100%;position: relative">
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

<script>

    function bindEvent (dom, event, cb, useCapture) {
        function remove() {
            dom.removeEventListener(event, icc, useCapture)
        }

        function icc(e) {
            if (cb(e) == true) {
                remove();
            }
        }

        dom.addEventListener(event, icc, useCapture);
        return remove;
    };

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
                default: '320px'
            },
            height: {
                type: String,
                default: '640px'
            },
            scroll: {
                type: Boolean,
                default: false
            },
            transitionTime: {
                type: String,
                default: '0s'
            }
        },
        data: function () {
            return {
                child: null, parent: null,eventRemoveHandle:function () {
                    
                }
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

                let self = this;
                let width = self.parent.clientWidth;
                let height = self.parent.clientHeight;


                let radio = width / height;
                let cut_w, cut_h;

                if (self.width === 'auto') {
                    cut_w = self.child.clientWidth;
                } else {
                    cut_w = Number(this.width.replace('px', ''))
                }
                if (self.width === 'auto') {
                    cut_h = self.child.clientHeight;
                } else {
                    cut_h = Number(self.height.replace('px', ''))
                }


                let radio2 = cut_w / cut_h;
                let end_w, end_h;

                switch (self.align) {
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
                end_w = Math.ceil(end_w);
                end_h = Math.ceil(end_h);

                let mov_x = -Math.ceil((end_w - width) / 2) + 'px';
                let mov_y = -Math.ceil((end_h - height) / 2) + 'px';
                let scale = Math.min(end_w / cut_w);


                _cssToDom(self.child, fixCss('transform-origin', '0% 0%'));

                _cssToDom(self.child, fixCss('transform', 'translate(' + mov_x + ',' + mov_y + ') scale('+scale+') '));



            }
        },
        
        beforeDestroy:function () {
            this.eventRemoveHandle()
        },


        mounted: function () {
            let self = this;
            this.parent = this.$refs.parent;
            this.child = this.$refs.child;
            _cssToDom(this.child, fixCss('transition-property', 'transform,width'));
            _cssToDom(this.child, fixCss('transition-duration', this.transitionTime));

            self.update();
            self.eventRemoveHandle=bindEvent(window, 'resize', function () {
                self.update();
            })
            if (false === this.scroll) {
                this.parent.setAttribute('ontouchmove', 'event.preventDefault();');
            }


        }


    }


</script>