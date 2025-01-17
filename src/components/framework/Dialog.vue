<template>
  <transition name="scale">
    <div
      v-if="open"
      class="dialog"
      @click="clickOutside"
      :class="{ '-open': open, '-medium': medium, '-large': large, '-small': small, '-mask': mask }"
    >
      <div
        ref="dialogContent"
        class="dialog-content"
        @click.stop
        :style="`transform: translate(${delta.x}px, ${delta.y}px)`"
        @mousedown="onMouseDown"
      >
        <header @mousedown="handleDrag" @touchstart="handleDrag">
          <slot name="header"></slot>
          <div class="dialog-controls">
            <slot name="controls"></slot>

            <a href="javascript:void(0);" class="dialog-controls__close -link" @click="$emit('clickOutside')">
              <i class="icon-cross"></i>
            </a>
          </div>
        </header>
        <div class="dialog-body custom-scrollbar">
          <slot></slot>
        </div>
      </div>
    </div>
  </transition>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import { getEventCords } from '../../utils/picker'

@Component({
  name: 'Dialog',
  props: {
    open: {
      type: Boolean
    },
    small: {
      type: Boolean
    },
    medium: {
      type: Boolean
    },
    large: {
      type: Boolean
    },
    mask: {
      type: Boolean,
      default: true
    }
  }
})
export default class extends Vue {
  delta = { x: 0, y: 0 }
  target = { x: 0, y: 0 }
  animating = false
  private clickOutsideClose: boolean
  private _handleRelease: () => void
  private _handleDragging: (evnt: any) => void

  $refs!: {
    dialogContent: HTMLElement
  }

  created() {
    this.clickOutsideClose = true
  }

  beforeDestroy() {
    if (this._handleRelease) {
      this._handleRelease()
    }
  }

  handleDrag(event) {
    if (event.button === 2) {
      return
    }
    if (event.target.classList.contains('-no-grab')) {
      return
    }
    event.preventDefault()
    const lastMove = Object.assign({}, this.delta)
    const startPosition = getEventCords(event)

    const startOffset = this.$refs.dialogContent.offsetTop
    this._handleDragging = evnt => {
      const endPosition = getEventCords(evnt)

      const x = lastMove.x + endPosition.x - startPosition.x
      const y = Math.max(startOffset * -1, lastMove.y + endPosition.y - startPosition.y)

      this.target.x = x
      this.target.y = y

      this.animate()
    }
    this._handleRelease = () => {
      document.removeEventListener('mousemove', this._handleDragging)
      document.removeEventListener('mouseup', this._handleRelease)
      document.removeEventListener('touchmove', this._handleDragging)
      document.removeEventListener('touchup', this._handleRelease)

      delete this._handleRelease
    }
    document.addEventListener('mousemove', this._handleDragging)
    document.addEventListener('mouseup', this._handleRelease)
    document.addEventListener('touchmove', this._handleDragging)
    document.addEventListener('touchup', this._handleRelease)
  }

  onMouseDown() {
    this.clickOutsideClose = false

    const handler = () => {
      document.removeEventListener('mouseup', handler)

      setTimeout(() => {
        this.clickOutsideClose = true
      }, 100)
    }

    document.addEventListener('mouseup', handler)
  }

  animate() {
    this.delta.x = this.target.x
    this.delta.y = this.target.y
    const distance = Math.abs(this.delta.x - this.target.x) + Math.abs(this.delta.y - this.target.y)

    if (distance > 10 && !this.animating) {
      this.animating = true
      return requestAnimationFrame(this.animate)
    }

    this.animating = false
  }

  clickOutside() {
    if (!this.clickOutsideClose) {
      return false
    }

    this.$emit('clickOutside')
  }
}
</script>
<style lang="scss" scoped>
.scale-enter-active,
.scale-leave-active {
  transition: opacity 0.2s $ease-out-expo, transform 0.2s $ease-out-expo;
}
.scale-enter, .scale-leave-to /* .scale-leave-active below version 2.1.8 */ {
  opacity: 0;
  transform: scale(0.8);
}
</style>
