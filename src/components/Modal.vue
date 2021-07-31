<template>
  <div v-if="isOpen" class="ss-modal">
    <div class="ss-modal__body">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { ref } from "@vue/reactivity";

// defino props, informacion que le entra del exterior
const props = defineProps({
    'is-open': { type: Boolean, default: false }
})

</script>

<style>
.ss-modal {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.5);
}

.ss-modal__body {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
}

.ss-modal__actions {
    display: flex;
    justify-content: center;
}

.ss-modal__actions button {
    font-size: 16px;
    min-height: 44px;
    padding: 16px;
    cursor: pointer;
}
</style>