<template>
  <div
    class="va-progress-circle"
    ref="progress"
    :style="computedStyle"
    :class="computedClass"
  >
    <svg
      class="va-progress-circle__progress-bar"
      viewBox="0 0 40 40"
    >
      <circle
        class="va-progress-circle__overlay"
        cx="50%"
        cy="50%"
        :r="radius"
        fill="none"
        :stroke="colorComputed"
        :stroke-width="computedThickness"
        :stroke-dasharray="dasharray"
        :stroke-dashoffset="dashoffset"
      />
    </svg>
    <div
      :style="computedStyles"
      class="va-progress-circle__info"
    >
      <slot />
    </div>
  </div>
</template>

<script>
import { progressMixin } from './progressMixin'
import {
  ColorThemeMixin,
  getColor,
} from '../../../../services/ColorThemePlugin'
import { makeContextablePropsMixin } from '../../../context-test/context-provide/ContextPlugin'
import { SizeMixin } from '../../../../mixins/SizeMixin'

const ProgressCircleContextMixin = makeContextablePropsMixin({
  thickness: {
    type: Number,
    default: 0.06,
  },
  color: {
    type: String,
    default: 'primary',
  },
})

export default {
  name: 'VaProgressCircle',
  mixins: [
    progressMixin,
    ColorThemeMixin,
    SizeMixin,
    ProgressCircleContextMixin,
  ],
  computed: {
    radius () {
      return 20 - (20 * this.cappedThickness / 100)
    },
    dasharray () {
      return 2 * Math.PI * this.radius
    },
    dashoffset () {
      return this.dasharray * (1 - this.normalizedValue / 100)
    },
    computedThickness () {
      return `${this.cappedThickness}%`
    },
    computedStyle () {
      return { width: this.sizeComputed, height: this.sizeComputed }
    },
    computedClass () {
      return {
        'va-progress-circle--indeterminate': this.c_indeterminate,
      }
    },
    computedStyles () {
      return {
        color: getColor(this, this.c_color),
      }
    },
    cappedThickness () {
      // value translated to percentage, divided in half, since final maximum value should be 50%
      if (this.c_thickness <= 0) {
        return 0
      } else if (this.c_thickness >= 1) {
        return 50
      } else {
        return this.c_thickness / 2 * 100
      }
    },
  },
}
</script>

<style lang="scss">
@import "../../../vuestic-sass/resources/resources";

.va-progress-circle {
  position: relative;
  overflow: hidden; // Prevents resizing container back and forth.

  &__progress-bar {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    margin: auto;
    transform: rotate(-90deg);
    width: 80%;
    height: 80%;

    @include flex-center();

    @at-root {
      .va-progress-circle--indeterminate & {
        animation: va-progress-circle__progress-bar--indeterminate 2s linear infinite;
      }
    }
  }

  &__overlay {
    transition: all ease 2s;

    @at-root {
      .va-progress-circle--indeterminate & {
        animation: va-progress-circle__overlay--indeterminate 2s ease-in-out infinite;
      }
    }
  }

  &__info {
    font-size: 0.75rem;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}

@keyframes va-progress-circle__progress-bar--indeterminate {
  100% {
    transform: rotate(270deg);
  }
}

@keyframes va-progress-circle__overlay--indeterminate {
  0% {
    stroke-dasharray: 1, 125;
    stroke-dashoffset: 0;
  }

  50% {
    stroke-dasharray: 125, 125;
    stroke-dashoffset: -65px;
  }

  100% {
    stroke-dasharray: 125, 125;
    stroke-dashoffset: -125px;
  }
}
</style>
