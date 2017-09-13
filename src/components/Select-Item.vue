<template>
    <li :class="classes" @click.stop="select" @mouseout.stop="blur" v-show="!hidden">
    	<slot>{{ showLabel }}</slot>
    	<i class="second-select-icon" v-if="showIcon"></i>
    </li>
</template>

<script>
    import Emitter from '../utils/emitter';
    import { findComponentUpward } from '../utils/assist';
    const prefixCls = 'jq-select-item';

    export default {
        name: 'SelectItem',
        mixins: [ Emitter ],
        componentName: 'select-item',
        props: {
            value: {
                type: [String, Number],
                required: true
            },
            label: {
                type: [String, Number]
            },
            disabled: {
                type: Boolean,
                default: false
            },
            secondSelect: {
                type: Boolean,
                default: false
            },
            showIcon: {
                type: Boolean,
                default: false
            }
        },
        data () {
            return {
                selected: false,
                isFocus: false,
                hidden: false,   
                searchLabel: ''
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-disabled`]: this.disabled,
                        [`${prefixCls}-selected`]: this.selected,
                        [`${prefixCls}-focus`]: this.isFocus
                    }
                ];
            },
            showLabel () {
                return (this.label) ? this.label : this.value;
            }
        },
        methods: {
            select () {
                if (this.disabled) {
                    return false;
                }
                this.dispatch('Select', 'on-select-selected', this.value);
            },
            blur () {
                this.isFocus = false;
            },
            queryChange (val) {
                const parsedQuery = val.replace(/(\^|\(|\)|\[|\]|\$|\*|\+|\.|\?|\\|\{|\}|\|)/g, '\\$1');

                if (this.$parent.$options.name === "DropdownMenu") {
                    this.hidden = false;
                    return;
                }

                this.hidden = !new RegExp(parsedQuery, 'i').test(this.searchLabel);
            }
        },
        mounted () {
            this.searchLabel = this.$el.textContent;
            this.dispatch('Select', 'append');

            this.$on('on-select-close', () => {
                this.isFocus = false;
            });

            this.$on('on-query-change', (val) => {
                this.queryChange(val);
            });
        },
        beforeDestroy () {
            this.dispatch('Select', 'remove');
        }
	}
</script>   