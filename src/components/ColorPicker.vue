<template>
  <div class="color-picker-container" @mouseleave="closeOnHover">
    <div @mouseenter="openOnHover" @click="togglePicker">
      <slot name="trigger">
        <div
          class="default-trigger"
          :style="{ backgroundColor: modelValue }"
        ></div>
      </slot>
    </div>
    <Transition name="fade">
      <div v-if="isOpen" class="color-picker">
        <div class="color-grid">
          <button
            v-for="color in colors"
            :key="color"
            class="color-option"
            :style="{ backgroundColor: color }"
            @click="selectColor(color)"
          ></button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from "vue";
import { defaultColorList } from "../config.js";

const props = defineProps({
  modelValue: String,
  colors: {
    type: Array,
    default: () => defaultColorList,
  },
  triggerOnHover: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(["update:modelValue"]);

const isOpen = ref(false);

const togglePicker = () => {
  isOpen.value = !isOpen.value;
};

const openOnHover = () => {
  if (props.triggerOnHover) {
    isOpen.value = true;
  }
};

const closeOnHover = () => {
  if (props.triggerOnHover) {
    isOpen.value = false;
  }
};

const selectColor = (color) => {
  emit("update:modelValue", color);
  isOpen.value = false;
};
</script>

<style scoped>
.color-picker-container {
  --picker-size: 80px;
  --border-radius: 8px;
  --transition-speed: 0.3s;
  position: relative;
  display: inline-block;
}

.default-trigger {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: 2px solid #fff;
  box-shadow: 0 0 0 1px #ccc;
  cursor: pointer;
  transition: transform var(--transition-speed);
}

.default-trigger:hover {
  transform: scale(1.1);
}

.color-picker {
  position: absolute;
  top: 100%;
  left: 0;
  width: var(--picker-size);
  height: var(--picker-size);
  background-color: #fff;
  border-radius: var(--border-radius);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  z-index: 9999999999;
}

.color-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(4, 1fr);
  gap: 4px;
  padding: 8px;
  width: 64px;
  height: 64px;
}

.color-option {
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: transform var(--transition-speed);
}

.color-option:hover {
  transform: scale(1.1);
  z-index: 1;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity var(--transition-speed);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
