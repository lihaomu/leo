<template>
  <div :class="`${ prefixCls } ${ placement }`" v-carousels="carousels">
    <div :class="`${ prefixCls }-track`" :style="trackStyles">
      <slot></slot>
    </div>
    <template v-if="placement === 'center'">
      <div :class="`${ prefixCls }-icon left`">

        <o-button
          @on-click="carousels(true, width)"
          :disabled="translateX >= 0"
          iconSize="30"
          icon="left"
          type="text">
        </o-button>

      </div>
      <div :class="`${ prefixCls }-icon right`">

        <o-button
          :disabled="translateX + trackWidth <= width + gutter"
          @on-click="carousels(false, width)"
          iconSize="30"
          icon="right"
          type="text">
        </o-button>

      </div>
    </template>
    <div v-else :class="`${ prefixCls }-arrows`">
      <div :class="`${ prefixCls }-icon`">

        <o-button
          @on-click="carousels(true, width)"
          :disabled="translateX >= 0"
          icon="left"
          size="mini"
          type="text">
        </o-button>

      </div>
      <div :class="`${ prefixCls }-icon`">

        <o-button
          :disabled="translateX + trackWidth <= width + gutter"
          @on-click="carousels(false, width)"
          icon="right"
          size="mini"
          type="text">
        </o-button>

      </div>
    </div>
  </div>
</template>

<script>
  import oButton from '~/components/Button';

  import { getStyle } from '~/utils/assist';

  export default {
    name: 'oCarousels',
    components: { oButton },
    props: {
      placement: {
        validator (value) {
          return ['top-left', 'top-right', 'center', 'bottom-right', 'bottom-left'].includes(value);
        },
        default: 'center'
      },
      gutter: {
        type: Number,
        default: 0
      },
      value: {
        type: Number,
        default: 60
      }
    },
    provide () {
      return { gutter: this.gutter };
    },
    data () {
      return {
        prefixCls: `${ this.$PrefixCls }-carousels`,

        width: 0,

        translateX: 0,
        trackWidth: 0
      }
    },
    computed: {
      trackStyles () {
        let style = {
          width    : this.trackWidth + 'px',
          transform: 'translateX('+ this.translateX +'px)'
        };

        return this.gutter === 0
          ? style
          : Object.assign(style, {
              marginLeft : - this.gutter / 2 + 'px',
              marginRight: - this.gutter / 2 + 'px'
            })
      }
    },
    directives: {
      carousels: {
        bind (el, { value }, vnode) {
          let self = vnode.context;

          const mousewheelHandler = e => {
            e.preventDefault();
            value(e.wheelDelta ? e.wheelDelta > 0 : e.detail < 0)
          };
          const resizeHandler = () => {
            self.width = parseInt(getStyle(el, 'width'));
            self.translateX = Math.max(self.width + self.gutter - self.trackWidth, self.translateX)
          };

          el.__resize     = resizeHandler;
          el.__mousewheel = mousewheelHandler;

          typeof el.onmousewheel === "undefined"
            ? el.addEventListener('DOMMouseScroll', mousewheelHandler)
            : el.addEventListener('mousewheel', mousewheelHandler);
          window.addEventListener('resize', resizeHandler)
        },
        unbind (el, {}) {
          typeof el.onmousewheel === "undefined"
            ? el.removeEventListener('DOMMouseScroll', el.__mousewheel)
            : el.removeEventListener('mousewheel', el.__mousewheel);
          window.removeEventListener('resize', el.__resize);

          delete el.__resize;
          delete el.__mousewheel
        }
      }
    },
    methods: {
      slotChange (val) {
        this.$nextTick(() => { this.trackWidth += val })
      },
      carousels (isLeft, num) {
        num = num || this.value || this.width;

        if (isLeft) {
          let value = this.translateX + num;
          this.translateX = Math.min(0, value);
        } else {
          let value = this.translateX - num;
          this.translateX = Math.max(this.width + this.gutter - this.trackWidth, value);
        }
      }
    },
    mounted () {
      this.width = parseInt(getStyle(this.$el, 'width'));
    }
  }
</script>