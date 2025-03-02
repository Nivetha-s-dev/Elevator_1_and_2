<!-- button.vue -->
<template>
  <button
      class="elevator-button"
      :class="{
            'current': isCurrent,
            'queued': isQueued
        }"
      :disabled="disabled"
      @click="$emit('click')"
  >
    {{ number }}
  </button>
</template>

<script>
export default {
  name: 'Button',
  props: {
    number: {
      type: Number,
      required: true
    },
    isCurrent: {
      type: Boolean,
      default: false
    },
    isQueued: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    }
  }
}
</script>

<style scoped>
.elevator-button {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 2px solid #333;
  background: #fff;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.elevator-button:hover:not(:disabled) {
  background: #eee;
}

</style>
