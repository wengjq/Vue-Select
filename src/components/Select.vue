<template>
	<div :class="classes" v-clickoutside="handleClose">
		<div :class="[prefixCls + '-selection']" ref="reference" @click="toggleMenu">
			<slot name="input">
				<span :class="[prefixCls + '-placeholder']" v-show="showPlaceholder && !filterable">{{ placeholder }}</span>
				<span :class="[prefixCls + '-selected-value']" v-show="!showPlaceholder && !filterable">{{ selectedSingle }}</span>
				<input type="text" v-if="filterable" v-model="query" :disabled="disabled" :class="[prefixCls + '-input']" :placeholder="showPlaceholder ? placeholder : ''" @blur="handleBlur" ref="input">
				<i :class="iconCls"></i>
			</slot>
		</div>
		<transition :name="transitionName">
			<select-dropdown-component :is-fixed="isFixed" v-show="dropVisible" :placement="placement" ref="drop">
				<ul v-show="notFoundShow" :class="[prefixCls + '-not-found']">
					<li>{{ notFoundText }}</li>
				</ul>
				<ul v-show="!notFound" :class="[prefixCls + '-dropdown-list']">
					<slot></slot>
				</ul>
			</select-dropdown-component>
		</transition>
	</div>
</template>
<script>
import selectDropdownComponent from './Select-Dropdown';
import clickoutside from '../directives/clickoutside';
import Emitter from '../utils/emitter';
import { oneOf } from '../utils/assist';

const prefixCls = 'jq-select';

export default {
    name: 'Select',
    mixins: [ Emitter ],
    components: { selectDropdownComponent },
    directives: { clickoutside },
    props: {
        value: {
            type: [String, Number],
            default: ''
        },
        label: {
            type: [String, Number],
            default: ''
        },
        disabled: {
            type: Boolean,
            default: false
        },
	    placeholder: {
            type: String,
            default: '请选择'
        },
        filterable: {
            type: Boolean,
            default: false
        },
        notFoundText: {
            type: String,
            default: '无匹配数据'
        },
        labelInValue: {
            type: Boolean,
            default: false
        },
        placement: {
            validator (value) {
                return oneOf(value, ['top', 'bottom']);
            },
            default: 'bottom'
        },
        isFixed: {
        	type: Boolean,
            default: false
        }
    },
    data () {
        return {
        	prefixCls: prefixCls,
            visible: false,
            options: [],
            optionInstances: [],
            selectedSingle: '',    
            selectedMultiple: [],
            query: '',
            lastQuery: '',
            selectToChangeQuery: false,   
            notFound: false,
            model: this.value,
            currentLabel: this.label
        }  
    },
    computed: {
    	classes () {
    		return [
                `${prefixCls}`,
                {
                    [`${prefixCls}-visible`]: this.visible,
                    [`${prefixCls}-disabled`]: this.disabled
                }
            ];
    	},
    	iconCls () {
			return [
                `${prefixCls}-arrow`,
                {
                    [`${prefixCls}-trigger-arrow`]: this.visible
                }
            ];	
    	},
    	showPlaceholder () {
            let status = false;
            if ((typeof this.model) === 'string') {
                if (this.model === '') {
                    status = true;
                }
            } else if( this.model === null){
                status = true;
            }
            return status;
        },
        transitionName () {
            return this.placement === 'bottom' ? 'slide-up' : 'slide-down';
        },
        dropVisible () {
        	let status = true;
            const options = this.$slots.default || [];

            if (this.query === '' && !options.length) status = false;
            if (!options.length) status = false;

            return this.visible && status;
        },
        notFoundShow () {
            const options = this.$slots.default || [];
            return this.notFound || !options.length;
        }
    },
    methods: {
    	toggleMenu () {
    		if (this.disabled) {
                return false;
            }

            this.visible = !this.visible;
    	},
    	hideMenu () {
    		this.visible = false;
            this.broadcast('SelectItem', 'on-select-close');
    	},
    	handleClose () {
    		this.hideMenu();
    	},
    	findChild (cb) {
    		const find = function (child) {
                const name = child.$options.componentName;
                if (name) {
                    cb(child);
                } else if (child.$children.length) {
                    child.$children.forEach((innerChild) => {
                        find(innerChild, cb);
                    });
                }
            };

            if (this.optionInstances.length) {
                this.optionInstances.forEach((child) => {
                    find(child);
                });
            } else {
                this.$children.forEach((child) => {
                    find(child);
                });
            }
    	},
    	modelToQuery () {
    		if (this.filterable && this.model !== undefined) {
                this.findChild((child) => {
                    if (this.model === child.value) {
                        if (child.label) {
                            this.query = child.label;
                        } else if (child.searchLabel) {
                            this.query = child.searchLabel;
                        } else {
                            this.query = child.value;
                        }
                    }
                });
            }
    	},
    	broadcastQuery (val) {
    		this.broadcast('SelectItem', 'on-query-change', val);
    	},
    	updateOptions (slot = false) { 
    		let options = [];

            this.findChild((child) => {
                options.push({
                    value: child.value,
                    label: (child.label === undefined) ? child.$el.textContent : child.label
                });
                this.optionInstances.push(child);
            });
            this.options = options;
    	},
    	updateSingleSelected (init = false, slot = false) {
            const type = typeof this.model;

            if (type === 'string' || type === 'number') {
                let findModel = false;
                for (let i = 0; i < this.options.length; i++) {
                    if (this.model === this.options[i].value) {
                        this.selectedSingle = this.options[i].label;
                        findModel = true;
                        break;
                    }
                }
                if (slot && !findModel) {
                    this.model = '';
                    this.query = '';
                }
            }
            this.toggleSingleSelected(this.model, init);
        },
        toggleSingleSelected (value, init = false) {
            let label = '';

            this.findChild((child) => {
                if (child.value === value) {
                    child.selected = true;
                    label = (child.label === undefined) ? child.$el.innerHTML : child.label;
                } else {
                    child.selected = false;
                }
            });

            this.hideMenu();

            if (!init) {
                if (this.labelInValue) {
                    this.$emit('on-change', {
                        value: value,
                        label: label
                    });
                } else {
                    this.$emit('on-change', value);
                }
            }
        },
        slotChange () {
            this.options = [];
            this.optionInstances = [];
        },
        handleBlur () {
        	setTimeout(() => {
                const model = this.model;

                if (model !== '') {
                    this.findChild((child) => {
                        if (child.value === model) {
                            this.query = child.label === undefined ? child.searchLabel : child.label;
                        }
                    });
                } else {
                    this.query = '';
                }
            }, 300);
        }
    },
    watch: {
    	value (val) {
    		this.model = val;
            if (val === '') this.query = '';
    	},
    	label (val) {
    		this.currentLabel = val;
    	},
    	model () {
    		this.$emit('input', this.model);
    		this.modelToQuery();

    		this.updateSingleSelected();

    		if (!this.visible && this.filterable) {
                this.$nextTick(() => {
                    this.broadcastQuery('');
                });
            }
    	},
    	visible (val) {
    		if (val) {
                if (this.filterable) {
                    this.$refs.input.select();
                }
                this.broadcast('SelectDropdown', 'on-update-popper');
            } else {
                if (this.filterable) {
                    this.$refs.input.blur();

                    setTimeout(() => {
                        this.broadcastQuery('');
                    }, 300);
                }
                this.broadcast('SelectDropdown', 'on-destroy-popper');
            }
    	},
    	query (val) {
    		if (!this.selectToChangeQuery) {
                this.$emit('on-query-change', val);
            }
                
            this.broadcastQuery(val);

            var is_hidden = true;

            this.$nextTick(() => {
                this.findChild((child) => {
                    if (!child.hidden && child.$parent.$options.name !== "DropdownMenu") {
                        is_hidden = false;
                    }
                });
                this.notFound = is_hidden;
            });

            this.selectToChangeQuery = false;

            this.broadcast('SelectDropdown', 'on-update-popper');
    	}
    },
    mounted () {
    	this.modelToQuery();

        this.$nextTick(() => {
            this.broadcastQuery('');
        });	

        this.updateOptions(true);	

        this.$on('append', () => {
            this.modelToQuery();
            this.$nextTick(() => {
                this.broadcastQuery('');
            });
      
            this.slotChange();
            this.updateOptions(true, true);
        });

        this.$on('remove', () => {
            this.modelToQuery();
            this.$nextTick(() => {
                this.broadcastQuery('');
            });

            this.slotChange();
            this.updateOptions(true, true);
        });

        this.$on('on-select-selected', (value) => {
            if (this.model === value) {
                this.hideMenu();
            } else {
                this.model = value;
                if (this.filterable) {
                    this.findChild((child) => {
                        if (child.value === value) {
                            if (this.query !== '') this.selectToChangeQuery = true;
                            this.query = child.label === undefined ? child.searchLabel : child.label;
                        }
                    });
                }
            }
        });
    }
}   
</script>

<style>
.jq-select {display: inline-block; width: 100px; vertical-align: middle; color: #333; font-size: 13px; font-family: 微软雅黑;}
.jq-select .jq-select-selection {position: relative; height: 30px;}
.jq-select-selection {position: relative;display: block; outline: 0; -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none; cursor: pointer; border-radius: 2px; border: 1px solid #e3e2e8; transition: all .2s ease-in-out;}
.jq-select-disabled .jq-select-selection {border-color: #E3E2E8; background-color: #F2F2F2; cursor: not-allowed; color: #ccc;}
.jq-select-not-found {color: #cccccc; text-align: center; padding: 0; list-style: none;}
.jq-select ::-webkit-input-placeholder {color:#333; font-size:13px;}
.jq-select ::-moz-placeholder {color:#333; font-size:13px;}
.jq-select ::-ms-input-placeholder {color:#333; font-size:13px;}

.jq-select .jq-select-selection .jq-select-placeholder, .jq-select .jq-select-selection .jq-select-selected-value {display: block; height: 30px; line-height: 30px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; padding-left: 10px; padding-right: 10px;}
.jq-select .jq-select-arrow {position: absolute; top: 1px; right: 3px; display: block; width: 25px; height: 28px; background: url(../assets/select.png) no-repeat; background-position: 7px -13px; transition: all .3s;}
.jq-select-disabled .jq-select-arrow {background: none;}
.jq-select .jq-select-trigger-arrow {-webkit-transform: rotate(180deg); -moz-transform: rotate(180deg); -o-transform: rotate(180deg); -ms-transform: rotate(180deg); transform: ratate(180deg);}

.jq-select-dropdown {position: absolute; width: inherit; margin: 8px 0; padding: 5px 0; background-color: #fff; border-radius: 2px; box-shadow: 0 0 8px 1px rgba(0,0,0,0.14); z-index: 1;}
.jq-select-dropdown-list {list-style: none; padding: 0; margin: 0;}
.jq-select-dropdown[x-placement="right-start"] { margin-left: 10px;}
.jq-select-item {position: relative; margin: 2px 5px !important;line-height: normal; padding: 6px 0 !important; clear: both; color: #333; white-space: nowrap; list-style: none; text-align: center; cursor: pointer;}
.jq-select-item .second-select-icon { position: absolute; border: solid transparent 4px; border-left-color: #333; right: 3px; margin-top: 5px;}
.jq-select-item-selected, .jq-select-item:hover {color: #fff; background: #5874D8;}
.jq-select-item:hover .second-select-icon {border-left-color: #fff; background: #5874D8;}
.jq-select-item-disabled, .jq-select-item-disabled:hover {color: #ccc; cursor: not-allowed;}
.jq-select-item-disabled:hover {background-color: #fff;}

.jq-select-input {position: relative; display: inline-block; width: 100%; height: 30px; line-height: 30px; text-indent: 10px; font-size: 13px; outline: 0; border: none; color: #333; background-color: transparent;  cursor: pointer;}
	
@keyframes jqFadeIn {
    0% {
        opacity: 0
    }

    to {
        opacity: 1
    }
}

@keyframes jqFadeOut {
    0% {
        opacity: 1
    }

    to {
        opacity: 0
    }
}

.fade-appear,.fade-enter-active,.fade-leave-active {
    animation-duration: .3s;
    animation-fill-mode: both;
    animation-play-state: paused
}

.fade-appear,.fade-enter-active {
    animation-name: jqFadeIn;
    animation-play-state: running
}

.fade-leave-active {
    animation-name: jqFadeOut;
    animation-play-state: running
}

.fade-appear,.fade-enter-active {
    opacity: 0
}

.fade-appear,.fade-enter-active,.fade-leave-active {
    animation-timing-function: linear
}


@keyframes jqSlideUpIn {
    0% {
        opacity: 0;
        transform-origin: 0 0;
        transform: scaleY(.8)
    }

    to {
        opacity: 1;
        transform-origin: 0 0;
        transform: scaleY(1)
    }
}

@keyframes jqSlideUpOut {
    0% {
        opacity: 1;
        transform-origin: 0 0;
        transform: scaleY(1)
    }

    to {
        opacity: 0;
        transform-origin: 0 0;
        transform: scaleY(.8)
    }
}

.slide-up-appear,.slide-up-enter-active,.slide-up-leave-active {
    animation-duration: .3s;
    animation-fill-mode: both;
    animation-play-state: paused
}

.slide-up-appear,.slide-up-enter-active {
    animation-name: jqSlideUpIn;
    animation-play-state: running
}

.slide-up-leave-active {
    animation-name: jqSlideUpOut;
    animation-play-state: running
}

.slide-up-appear,.slide-up-enter-active {
    opacity: 0;
    animation-timing-function: ease-in-out
}

.slide-up-leave-active {
    animation-timing-function: ease-in-out
}

@keyframes jqSlideDownIn {
    0% {
        opacity: 0;
        transform-origin: 100% 100%;
        transform: scaleY(.8)
    }

    to {
        opacity: 1;
        transform-origin: 100% 100%;
        transform: scaleY(1)
    }
}

@keyframes jqSlideDownOut {
    0% {
        opacity: 1;
        transform-origin: 100% 100%;
        transform: scaleY(1)
    }

    to {
        opacity: 0;
        transform-origin: 100% 100%;
        transform: scaleY(.8)
    }
}

.slide-down-appear,.slide-down-enter-active,.slide-down-leave-active {
    animation-duration: .3s;
    animation-fill-mode: both;
    animation-play-state: paused
}

.slide-down-appear,.slide-down-enter-active {
    animation-name: jqSlideDownIn;
    animation-play-state: running
}

.slide-down-leave-active {
    animation-name: jqSlideDownOut;
    animation-play-state: running
}

.slide-down-appear,.slide-down-enter-active {
    opacity: 0;
    animation-timing-function: ease-in-out
}

.slide-down-leave-active {
    animation-timing-function: ease-in-out
}

</style>