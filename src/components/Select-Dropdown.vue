<template>
    <div class="jq-select-dropdown" :class="className" :style="styles">
    	<slot></slot>
    </div>
</template>

<script>
	import { getStyle } from '../utils/assist';
	import { Popper } from '../utils/popper.js';

	export default {
        name: 'SelectDropdown',
        props: {
            placement: {
                type: String,
                default: 'bottom-start'
            },
            className: {
                type: String
            },
            secondSelect: {
                type: Boolean,
                default: false
            },
            isFixed: {
                type: Boolean,
                default: false
            } 
        },
        data () {
            return {
                popper: null,
                width: ''
            };
        },
        computed: {
            styles () {
                let style = {};
                if (this.width) style.width = `${this.width}px`;
                return style;
            }
        },
        created () {
            this.$on('on-update-popper', this.update);
            this.$on('on-destroy-popper', this.destroy);
        },
        beforeDestroy () {
            if (this.popper) {
                this.popper.destroy();
            }
        },
        methods: {
        	update () {
        		let param = {};
        		if (this.popper) {
                    this.$nextTick(() => {
                        this.popper.update();
                    });
                } else {
                    this.$nextTick(() => {
                    	if (this.secondSelect || this.isFixed) {
                    		param = {
                                gpuAcceleration: false,
                                placement: this.placement,
                                boundariesPadding: 0,
                                forceFixed: true,
                                boundariesElement: 'body',
                                offset: this.placement.indexOf('right') > -1 ? -8 : 0
                            };   
                    	} else {
                    		param = {
                                gpuAcceleration: false,
                                placement: this.placement,
                                boundariesPadding: 0,
                                forceAbsolute: true,
                                boundariesElement: 'body',
                                offset: this.placement.indexOf('right') > -1 ? -8 : 0
                            };   
                    	}

                        this.popper = new Popper(this.$parent.$refs.reference, this.$el, param);

                        this.popper.onCreate(popper => {
                            this.resetTransformOrigin(popper);
                        });
                    });
                }
                if (this.$parent.$options.name === 'Select' || this.$parent.$options.name === 'Dropdown') {
                    this.width = parseInt(getStyle(this.$parent.$el, 'width'));
                }
        	},
        	destroy () {
                if (this.popper) {
                    this.resetTransformOrigin(this.popper);
                    setTimeout(() => {
                        this.popper.destroy();
                        this.popper = null;
                    }, 300);
                }
            },
            resetTransformOrigin(popper) {
                let placementMap = {top: 'bottom', bottom: 'top'};
                let placement = popper._popper.getAttribute('x-placement').split('-')[0];
                let origin = placementMap[placement];
                popper._popper.style.transformOrigin = `center ${ origin }`;
            }
        }
    }


</script>