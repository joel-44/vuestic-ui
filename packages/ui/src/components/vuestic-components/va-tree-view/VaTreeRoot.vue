<template>
  <div class="va-tree-root">
    <slot />
  </div>
</template>

<script>
import { ColorThemeMixin } from '../../../services/ColorThemePlugin'
import Vue from 'vue'

export default {
  name: 'VaTreeRoot',
  mixins: [ColorThemeMixin],
  provide () {
    // eslint-disable-next-line @typescript-eslint/no-this-alias
    const parent = this
    return {
      va: new Vue({
        computed: {
          color () {
            return parent.color
          },
        },
      }),
    }
  },
  methods: {
    /**
     * @public
     */
    collapse () {
      this.$nextTick(() => {
        this.$children.forEach(child => {
          if (child.$options.name === 'va-tree-category') {
            child.collapse()
          }
        })
      })
    },
    /**
     * @public
     */
    expand () {
      this.$nextTick(() => {
        this.$children.forEach(child => {
          if (child.$options.name === 'va-tree-category') {
            child.expand()
          }
        })
      })
    },
  },
}
</script>

<style lang="scss">
.va-tree-root {
  padding: 0.3125rem;
}
</style>
