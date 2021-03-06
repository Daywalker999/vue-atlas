<template>
  <transition :name="(this.placement === 'left') ? 'slideleft' : 'slideright'">
    <div :class="classObj" :style="{width:width}" ref="aside" v-show="show">
      <div :class="`${classPrefix}-aside-dialog`">
        <div :class="`${classPrefix}-aside-content`">
          <div :class="`${classPrefix}-aside-header`" v-if="header">
            <button
              :class="`${classPrefix}-close`"
              @click="close"
              type="button"
            >
              <span>&times;</span>
            </button>
            <div :class="`${classPrefix}-aside-title`">{{title}}</div>
          </div>
          <div :class="`${classPrefix}-aside-body`">
            <slot/>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import EventListener from '../utils/EventListener'
import createFocusTrap from 'focus-trap'
import element from '../utils/element'

export default {
  name: 'VaAside',
  props: {
    placement: {
      type: String,
      default: 'left',
      required: false
    },
    title: {
      type: String,
      default: '',
      required: false
    },
    header: {
      type: Boolean,
      default: false,
      required: false
    },
    width: {
      type: String,
      default: '304px',
      required: false
    },
    classPrefix: {
      type: String,
      default: 'va',
      required: false
    }
  },
  data () {
    return {
      show: false,
      focusTrap: null
    }
  },
  computed: {
    classObj () {
      let { classPrefix, placement } = this
      let classes = {}

      classes[classPrefix + '-aside'] = true
      classes[classPrefix + '-aside-left'] = placement === 'left'
      classes[classPrefix + '-aside-right'] = placement === 'right'

      return classes
    }
  },
  created () {
    const escapeHandler = e => {
      if (e.key === 'Escape' && this.show) {
        this.close()
      }
    }
    document.addEventListener('keydown', escapeHandler)
    this.$once('hook:destroyed', () => {
      document.removeEventListener('keydown', escapeHandler)
    })
  },
  mounted () {
    document.querySelector('body').appendChild(this.$refs.aside)
    this.$once('hook:beforeDestroy', () => {
      document.querySelector('body').removeChild(this.$refs.aside)
    })

    this.focusTrap = createFocusTrap(this.$refs.aside, {
      clickOutsideDeactivates: true,
      returnFocusOnDeactivate: true,
      fallbackFocus: this.$refs.aside
    })
  },
  beforeDestroy () {
    this.performClose()
  },
  watch: {
    show (val) {
      let backdrop = document.createElement('div')
      let classPrefix = this.classPrefix
      const body = document.body
      backdrop.className = classPrefix + '-aside-backdrop'

      if (val) {
        body.appendChild(backdrop)
        element.addClass(body, classPrefix + '-modal-open')

        // This timeout is included to allow for opacity transition.
        setTimeout(() => {
          backdrop.className += ' ' + classPrefix + '-aside-in'
          this._clickEvent = EventListener.listen(backdrop, 'click', this.close)
          this.$emit('show')
        }, 20)

        this.focusTrap.activate()
      } else {
        this.focusTrap.deactivate()
        this.performClose()
      }
    }
  },
  methods: {
    open () {
      this.show = true
    },
    close () {
      this.show = false
    },
    performClose () {
      if (this._clickEvent) this._clickEvent.remove()

      const body = document.body
      let classPrefix = this.classPrefix
      let backdrop = document.querySelector(
        '.' + classPrefix + '-aside-backdrop'
      )

      if (backdrop) {
        backdrop.className = classPrefix + '-aside-backdrop'
        setTimeout(() => {
          element.removeClass(body, classPrefix + '-modal-open')
          body.removeChild(backdrop)
        }, 300)
        this.$emit('hide')
      }
    }
  }
}
</script>

<style lang="scss">
.#{$class-prefix}-aside-open {
  transition: transform 0.3s;
}

.#{$class-prefix}-aside {
  position: fixed;
  top: 0;
  bottom: 0;
  z-index: 10;
  overflow: auto;
  background: $N0;
  &-left {
    left: 0;
    right: auto;
  }
  &-right {
    right: 0;
    left: auto;
  }
  &:focus {
    outline: 0;
  }
  &-dialog {
    .#{$class-prefix}-aside-header {
      .#{$class-prefix}-close {
        font-size: 24px;
        color: $N300;
      }
      .#{$class-prefix}-aside-title {
        padding-top: 8px;
        font-size: 24px;
        color: $N300;
      }
    }
  }
  &-backdrop {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 5;
    opacity: 0;
    transition: opacity 0.3s ease-in-out;
    background: rgba(9, 30, 66, 0.54);
  }
  &-in {
    opacity: 1;
  }
}
</style>
