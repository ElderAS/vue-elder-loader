<template>
  <div
    class="elder-loader"
    :class="[
      'elder-loader--' + content.theme,
      { 'elder-loader--active': isLoading, ['elder-loader--' + state]: state },
    ]"
  >
    <slot v-if="showDefaultSlot" />
    <slot
      v-if="showSuccessSlot && content.theme === 'default'"
      name="success"
      :reset="reset"
    >
      <SlotHandler :value="content.messages.success" :reset="reset" />
    </slot>
    <slot
      v-if="showErrorSlot && content.theme === 'default'"
      name="error"
      :error="error"
      :reset="reset"
    >
      <SlotHandler :value="content.messages.error" :reset="reset" :error="error" />
      ></slot
    >

    <div v-if="showLoaderElement" class="elder-loader__element">
      <div class="elder-loader__element-content">
        <div class="elder-loader__element-content-inner">
          <slot
            v-if="showSuccessSlot && content.theme === 'overlay'"
            name="success"
            :reset="reset"
          >
            <SlotHandler :value="content.messages.success" :reset="reset" />
          </slot>
          <slot
            v-if="showErrorSlot && content.theme === 'overlay'"
            name="error"
            :error="error"
            :reset="reset"
          >
            <SlotHandler :value="content.messages.error" :reset="reset" :error="error" />
          </slot>
          <slot name="loader" v-if="isLoading" :content="content">
            <Icon class="elder-loader__loading-icon" v-bind="content.icon" spin />
            <div class="elder-loader__loading-message">
              <slot name="message">{{ content.messages.loading }}</slot>
            </div>
          </slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Icon from '@kvass/vue2-icon'
import { Options } from '../index'
import { iconBinding, isPromise } from './utils'
import SlotHandler from './SlotHandler'

export default {
  props: {
    value: [Boolean, Promise],
    theme: {
      type: String,
      enum: ['overlay', 'default'],
    },
    message: String,
    icon: [String, Object, Array],
    delay: Number,
  },
  data() {
    return {
      isLoading: false,
      state: null,
      error: null,
      timeout: null,
    }
  },
  watch: {
    value: {
      handler(value) {
        if (isPromise(value)) {
          this.start()

          value
            .then(() => this.stop('success'))
            .catch(err => {
              this.error = err
              this.stop('error')
              throw err
            })
        } else {
          if (!value) this.stop('success')
          else this.start()
        }
      },
      immediate: true,
    },
  },
  methods: {
    reset() {
      this.state = undefined
      this.error = undefined
      this.isLoading = false
    },
    start() {
      this.reset()

      if (!this.content.delay) this.isLoading = true
      else {
        this.timeout = clearTimeout(this.timeout)
        this.timeout = setTimeout(() => (this.isLoading = true), this.content.delay)
      }
    },
    stop(result) {
      this.timeout = clearTimeout(this.timeout)
      this.state = result
      this.isLoading = false
    },
  },
  computed: {
    showLoaderElement() {
      if (this.isLoading) return true
      if (this.showStateSlot && this.content.theme === 'overlay') return true
      return false
    },
    showDefaultSlot() {
      if (this.content.theme === 'overlay') return true
      return !this.isLoading && !this.showSuccessSlot && !this.showErrorSlot
    },
    showStateSlot() {
      return this.showErrorSlot || this.showSuccessSlot
    },
    showSuccessSlot() {
      return Boolean(
        this.state === 'success' &&
          (this.$slots.success || this.content.messages.success),
      )
    },
    showErrorSlot() {
      return Boolean(
        this.state === 'error' &&
          (this.$slots.error || this.$scopedSlots.error || this.content.messages.error),
      )
    },
    content() {
      return {
        icon: iconBinding(this.icon || Options.icon),
        messages: {
          loading: this.message || Options.messages.loading,
          error: Options.messages.error,
          success: Options.messages.success,
        },
        theme: this.theme || Options.theme,
        delay: this.delay || Options.delay,
      }
    },
  },
  components: {
    Icon,
    SlotHandler,
  },
}
</script>

<style lang="scss">
$variables: (
  'primary': #3a9acd,
  'border-radius': 3px,
  'border-color': #eaeaea,
  'loader-animation-duration': 500ms,
  'loader-theme-overlay-transparency': 0.6,
);

@function GetVariable($key) {
  @return var(--vue-elder-#{$key}, map-get($variables, $key));
}

.elder-loader {
  flex-grow: 1;

  &--default {
    &.elder-loader--active {
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }

  &--overlay {
    position: relative;

    & > .elder-loader__element {
      position: absolute;
      z-index: 2;
      top: 0;
      left: 0;

      width: 100%;
      height: 100%;

      background-color: rgba(white, GetVariable('loader-theme-overlay-transparency'));

      & > .elder-loader__element-content {
        position: sticky;
        top: 0;

        display: flex;
        align-items: center;
        flex-direction: column;
        justify-content: center;

        height: 100%;
        max-height: 95vh;

        & > .elder-loader__element-content-inner {
          border: 1px solid GetVariable('border-color');
          background-color: white;
        }
      }
    }
  }

  &__element {
    &-content {
      &-inner {
        min-width: 200px;
        margin: 2rem 0;
        padding: 1rem 2rem;

        text-align: center;

        border-radius: GetVariable('border-radius');
      }
    }
  }

  &__loading-icon {
    --kvass-icon-spin-duration: #{GetVariable('loader-animation-duration')};
    color: GetVariable('primary');
  }

  &__loading-message {
    margin-top: 0.5rem;
  }
}
</style>
