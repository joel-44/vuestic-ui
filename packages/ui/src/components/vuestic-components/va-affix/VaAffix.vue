<template>
  <div
    ref="element"
    class="va-affix"
  >
    <div :style="{ visibility: isAffixed ? 'hidden' : 'inherit' }">
      <slot />
    </div>
    <div
      v-if="isAffixed"
      :class="classes"
      :style="styles"
    >
      <slot />
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import { noop } from 'lodash'
import { makeContextablePropsMixin } from '../../context-test/context-provide/ContextPlugin'
import { handleThrottledEvent, useEventsHandlerWithThrottle, getWindowHeight, State } from './VaAffix-utils'

const prefixClass = 'va-affix'

const props = {
  offsetTop: {
    type: Number,
    default: undefined,
  },
  offsetBottom: {
    type: Number,
    default: undefined,
  },
  target: {
    type: [HTMLElement, Window],
    default: () => window,
  },
}

const ContextableMixin = makeContextablePropsMixin(props)

@Component({
  name: 'VaAffix',
})
export default class VaAffix extends Mixins(ContextableMixin) {
  private state: State = {
    isTopAffixed: false,
    isBottomAffixed: false,
  }

  private initialPosition?: undefined | ClientRect
  private clearEventListeners = noop

  get classes () {
    return [
      {
        [`${prefixClass}--affixed`]: this.isAffixed,
      },
    ]
  }

  getTarget () {
    // a custom target may get rendered later than
    // a component gets a property from the context
    const { c_target, target } = this

    return target || c_target
  }

  get styles () {
    const calculateTop = () => {
      const target = this.getTarget()

      if (this.c_offsetTop === undefined) {
        return
      }

      if (target !== window) {
        const { top } = (target as HTMLElement).getBoundingClientRect()
        return top + this.c_offsetTop
      }

      return this.c_offsetTop
    }

    const calculateBottom = () => {
      const target = this.getTarget()
      if (this.c_offsetBottom === undefined) {
        return
      }

      if (target !== window) {
        const { bottom } = (target as HTMLElement).getBoundingClientRect()
        const { offsetHeight, clientHeight } = (target as HTMLElement)
        const scrollBarHeight = offsetHeight - clientHeight
        const windowHeight = getWindowHeight()
        return windowHeight - (bottom - this.c_offsetBottom) + scrollBarHeight
      }

      return this.c_offsetBottom
    }

    const convertToPixels = (calculate: Function) => {
      const result = calculate()

      if (result === undefined) {
        return
      }

      return `${result}px`
    }

    return {
      top: this.state.isTopAffixed ? convertToPixels(calculateTop) : null,
      bottom: this.state.isBottomAffixed ? convertToPixels(calculateBottom) : null,
      width: `${this.state.width}px`,
    }
  }

  get isAffixed (): boolean {
    return this.state.isTopAffixed || this.state.isBottomAffixed
  }

  handleThrottledEvent (eventName: string | null, event?: Event) {
    const context = {
      ...this.$data,
      ...this.$props,
      element: this.$refs.element,
      target: this.getTarget(),
      setState: this.setState.bind(this),
      getState: this.getState.bind(this),
    }

    if (!eventName || eventName === 'resize') {
      handleThrottledEvent(eventName, context)
    } else if (event && event.target) {
      const target = this.getTarget()

      if ((target as HTMLElement) === event.target || target === window) {
        handleThrottledEvent(eventName, context)
      } else {
        // if we have a custom target but keep scrolling on another element,
        // so just disable the affixed state
        this.setState({
          isBottomAffixed: false,
          isTopAffixed: false,
        })
      }
    }
  }

  setState (state: State) {
    this.state = state
    this.$emit('change', this.isAffixed)
  }

  getState () {
    return this.state
  }

  mounted () {
    const events = ['scroll', 'resize']

    this.initialPosition = (this.$refs.element as HTMLElement).getBoundingClientRect()

    this.clearEventListeners = useEventsHandlerWithThrottle(events, {
      handler: this.handleThrottledEvent,
    })

    this.$nextTick(() => {
      // pass `null` here to make sure it is an initial call
      this.handleThrottledEvent(null)
    })
  }

  beforeDestroy () {
    this.clearEventListeners()
  }
}
</script>

<style scoped>
  .va-affix {}

  .va-affix--affixed {
    position: fixed;

    /* TODO: make it a global variable */
    z-index: 10;
  }
</style>
