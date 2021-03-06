<template>
  <div
    v-click-outside.capture="handleClose"
    @click="handleOpen"
    :class="classes">
    <div :class="valueClasses">
      <span v-if="!selected"><slot></slot></span>
      {{ selected }}

      <o-icon
        @on-click.stop="handleClear"
        :class="clearClasses"
        v-if="canClear"
        type="clear">
      </o-icon>

      <o-icon type="down" :class="`${ prefixCls }-icon arrows`" :size="12"></o-icon>

    </div>
    <transition :name="`${ $PrefixCls }-drop`">
      <div :class="`${ prefixCls }-list`" v-show="visible">
        <div
          :class="[`${ prefixCls }-item`, { active: item.value === value }]"
          @click="handleChange(item.value)"
          v-for="item in list">
          {{ item.label }}
        </div>
        <div v-if="!list.length" :class="`${ prefixCls }-text`">{{ notFoundText }}</div>
      </div>
    </transition>
  </div>
</template>

<script>
  import oIcon from '~/components/Icon';

  import { directive as clickOutside } from 'v-click-outside-x';

  export default {
    name: 'oSelect',
    components: { oIcon },
    directives: { clickOutside },
    model: {
      prop: 'value',
      event: 'change'
    },
    props: {
      size: {
        validator (value) {
          return ['large', 'small', 'mini', ''].includes(value);
        },
        default: ''
      },
      placement: {
        validator (value) {
          return ['top', 'bottom'].includes(value);
        },
        default: 'bottom'
      },
      options: {
        type: Array,
        default () { return [] }
      },
      value: {
        type: [Number, String],
        default: ''
      },
      notFoundText: {
        type: String,
        default: '无匹配数据'
      },
      clearable: {
        type: Boolean,
        default: false
      },
      disabled: {
        type: Boolean,
        default: false
      }
    },
    inject: {
      form: { default: null }
    },
    data () {
      return {
        prefixCls: `${ this.$PrefixCls }-select`,

        visible: false
      }
    },
    computed: {
      classes () {
        return [
          this.prefixCls,
          this.placement,
          this.form ? this.form.size : this.size,
          {
            open    : this.visible,
            selected: this.selected && !this.disabled
          }
        ]
      },
      valueClasses () {
        return [
          `${ this.prefixCls }-value`,
          {
            disabled: this.disabled
          }
        ]
      },
      clearClasses () {
        return [
          `${ this.prefixCls }-icon`,
          {
            clear: this.canClear
          }
        ]
      },
      list () {
        let options = this.options;

        return !options.some(option => typeof option !== 'object')
          ? options
          : options.map(option => {
              return { label: option, value: option }
            })
      },
      selected () {
        let selected = '';

        this.list.forEach(item => {
          if (item.value === this.value) selected = item.label
        });

        return selected
      },
      canClear () {
        return !this.disabled && this.clearable && this.selected !== ''
      }
    },
    methods: {
      handleOpen () {
        if (!this.disabled) this.visible = !this.visible
      },
      handleClose () {
        if (this.visible) this.visible = false
      },
      handleClear () {
        if (this.canClear) {
          this.handleChange('');
          this.$emit('on-clear')
        }
      },
      handleChange (val) {
        if (!this.disabled) {
          this.$emit('change', val);
          this.$emit('on-change', val)
        }
      }
    }
  }
</script>