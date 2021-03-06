<template>
  <va-input-wrapper
    class="va-input"
    :disabled="c_disabled"
    :success="c_success"
    :messages="messages"
    :error="computedError"
    :error-messages="computedErrorMessages"
    :error-count="errorCount"
  >
    <slot
      name="prepend"
      slot="prepend"
    />
    <div
      class="va-input__container"
      :class="{'va-input__container--textarea': isTextarea}"
      :style="containerStyles"
    >
      <div
        class="va-input__container__content-wrapper"
        :style="{ paddingTop: c_label ? '' : '0'}"
      >
        <label
          :style="labelStyles"
          aria-hidden="true"
          class="va-input__container__label"
        >
          {{ label }}
        </label>
        <textarea
          :id="id"
          :name="name"
          v-if="isTextarea"
          class="va-input__container__input"
          :style="textareaStyles"
          :aria-label="c_label"
          :placeholder="c_placeholder"
          :disabled="c_disabled"
          :readonly="c_readonly"
          :value="c_value"
          v-on="inputListeners"
          v-bind="$attrs"
          ref="input"
          :tabindex="c_tabindex"
        />
        <input
          v-else
          :id="id"
          :name="name"
          class="va-input__container__input"
          :style="{ paddingBottom: c_label ? '0.125rem' : '0.875rem' }"
          :aria-label="c_label"
          :type="c_type"
          :placeholder="c_placeholder"
          :disabled="c_disabled"
          :readonly="c_readonly"
          :value="c_value"
          v-on="inputListeners"
          v-bind="$attrs"
          ref="input"
          :tabindex="c_tabindex"
        >
      </div>
      <div
        v-if="c_success || computedError || $slots.append || (c_removable && hasContent)"
        class="va-input__container__icon-wrapper"
      >
        <va-icon
          v-if="c_success"
          class="va-input__container__icon"
          color="c_success"
          name="check"
        />
        <va-icon
          v-if="computedError"
          class="va-input__container__icon"
          color="danger"
          name="warning"
        />
        <slot name="append" />
        <va-icon
          v-if="c_removable && hasContent"
          @click.native="reset()"
          class="va-input__container__close-icon"
          :color="computedError ? 'danger': 'gray'"
          name="highlight_off"
        />
      </div>
    </div>
  </va-input-wrapper>
</template>

<script>
import VaInputWrapper from '../va-input/VaInputWrapper'
import VaIcon from '../va-icon/VaIcon'
import { getHoverColor } from '../../../services/color-functions'
import calculateNodeHeight from './calculateNodeHeight'
import { ColorThemeMixin } from '../../../services/ColorThemePlugin'
import { makeContextablePropsMixin } from '../../context-test/context-provide/ContextPlugin'
import { FormComponentMixin } from '../../vuestic-mixins/FormComponent/FormComponentMixin'
import { warn } from '../../../services/utils'

const InputContextMixin = makeContextablePropsMixin({
  color: {
    type: String,
    default: '',
  },
  value: {
    type: [String, Number],
    default: '',
  },
  label: {
    type: String,
    default: '',
  },
  placeholder: {
    type: String,
    default: '',
  },
  type: {
    type: String,
    default: 'text',
  },
  removable: {
    type: Boolean,
    default: false,
  },
  tabindex: {
    type: Number,
    default: 0,
  },

  // textarea-specific
  autosize: {
    type: Boolean,
    default: false,
  },
  minRows: {
    type: Number,
    default: null,
    validator: (val) => {
      if (!(val > 0 && (val | 0) === val)) {
        return warn(`\`minRows\` must be a positive integer greater than 0, but ${val} is provided`)
      }
      return true
    },
  },
  maxRows: {
    type: Number,
    validator: (val) => {
      if (!(val > 0 && (val | 0) === val)) {
        return warn(`\`minRows\` must be a positive integer greater than 0, but ${val} is provided`)
      }
      return true
    },
    default: null,
  },
})

export default {
  name: 'VaInput',
  mixins: [ColorThemeMixin, InputContextMixin, FormComponentMixin],
  components: { VaInputWrapper, VaIcon },
  mounted () {
    this.adjustHeight()
  },
  watch: {
    value () {
      this.adjustHeight()
    },
  },
  data () {
    return {
      isFocused: false,
    }
  },
  computed: {
    labelStyles () {
      if (this.computedError) {
        return { color: this.computeColor('danger') }
      }

      if (this.c_success) {
        return { color: this.computeColor('success') }
      }

      return { color: this.colorComputed }
    },
    containerStyles () {
      return {
        backgroundColor:
          this.computedError ? (this.computeColor('danger') ? getHoverColor(this.computeColor('danger')) : '')
            : this.c_success ? (this.computeColor('success') ? getHoverColor(this.computeColor('success')) : '') : '#f5f8f9',
        borderColor:
          this.computedError ? this.computeColor('danger')
            : this.c_success ? this.computeColor('success')
              : this.isFocused ? this.computeColor('dark') : this.computeColor('gray'),
      }
    },
    textareaStyles () {
      return {
        paddingBottom: this.label ? '0.125rem' : '',
        marginTop: this.label ? '0.875rem' : '',
        paddingTop: this.label ? 0 : '',
        minHeight: this.label ? '1.5rem' : '2.25rem',
        marginBottom: 0,
      }
    },
    inputListeners () {
      // TODO Probably not the best idea to stick this in computed.
      return Object.assign(
        {},
        this.$listeners,
        {
          input: event => {
            this.$emit('input', event.target.value)
          },
          click: event => {
            this.$emit('click', event)
          },
          focus: event => {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.isFocused = true

            this.$emit('focus', event)
          },
          blur: event => {
            // eslint-disable-next-line vue/no-side-effects-in-computed-properties
            this.ValidateMixin_onBlur()

            this.$emit('blur', event)
          },
          keyup: event => {
            this.$emit('keyup', event)
          },
          keydown: event => {
            this.$emit('keydown', event)
          },
        },
      )
    },
    hasContent () {
      return ![null, undefined, ''].includes(this.c_value)
    },
    isTextarea () {
      return this.c_type === 'textarea'
    },
  },
  methods: {
    adjustHeight () {
      if (!this.autosize || !this.isTextarea) { return }

      const minRows = this.minRows || 1
      const maxRows = this.maxRows || Number.MAX_SAFE_INTEGER
      const textareaStyles = calculateNodeHeight(this.$refs.input, false, minRows, maxRows)

      // We modify DOM directly instead of using reactivity because the whole adjustHeight method takes place
      // each time the value of textarea is modified, so there's no real need in an additional layer of reactivity.
      // The operation is basically reactive though implicitly.
      Object.assign(this.$refs.input.style, textareaStyles)
    },

    /** @public */
    focus () {
      this.$refs.input.focus()
    },
    /** @public */
    reset () {
      this.$emit('input', '')
    },
  },
}
</script>

<style lang='scss'>
@import '../../vuestic-sass/resources/resources';

.va-input {
  &__container {
    display: flex;
    position: relative;
    width: 100%;
    min-height: 2.375rem;
    border-style: solid;
    border-width: 0 0 thin 0;
    border-top-left-radius: 0.5rem;
    border-top-right-radius: 0.5rem;

    &__content-wrapper {
      display: flex;
      align-items: flex-end;
      width: 100%;

      /* min-width: 100%; */
    }

    &__icon-wrapper {
      display: flex;
      align-items: center;
      margin-right: 0.5rem;
    }

    &__close-icon {
      cursor: pointer;
      margin-left: 0.25rem;
    }

    &__label {
      position: absolute;
      bottom: 0.875rem;
      left: 0.5rem;
      margin-bottom: 0.5rem;
      max-width: calc(100% - 0.25rem);
      color: $vue-green;
      font-size: 0.625rem;
      letter-spacing: 0.0375rem;
      line-height: 1.2;
      font-weight: $font-weight-bold;
      text-transform: uppercase;

      @include va-ellipsis();

      transform-origin: top left;
    }

    &.va-input__container--textarea &__label {
      bottom: auto;
      top: 0.125rem;
    }

    &__input {
      width: 100%;
      height: 1.5rem;
      margin-bottom: 0.125rem;
      padding: 0.25rem 0.5rem;
      color: #34495e;
      background-color: transparent;
      border-style: none;
      outline: none;
      font-size: 1rem;
      font-family: $font-family-sans-serif;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.5;
      letter-spacing: normal;

      &::placeholder {
        color: $brand-secondary;
      }

      &:placeholder-shown {
        padding-bottom: 0.875rem;
      }
    }

    &.va-input__container--textarea &__input {
      resize: vertical;
      height: inherit;
    }
  }
}
</style>
