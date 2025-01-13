<script setup>
import {
  PinIcon,
  PinOffIcon,
  PaletteIcon,
  SaveIcon,
  CircleChevronDownIcon,
  MaximizeIcon,
  CirclePlusIcon,
  CircleMinusIcon,
  CircleXIcon,
} from "lucide-vue-next";
import { ref, onMounted, computed, onUnmounted } from "vue";
import { Editor, EditorContent } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import { appWindow, PhysicalSize } from "@tauri-apps/api/window";
import ColorPicker from "./components/ColorPicker.vue";

const containerRef = ref(null);
const isPinned = ref(false);
const filePath = ref("");
const isMaximized = ref(false);
const isHiddenContent = ref(false);
const selectedColor = ref("#4DB6AC");

const editor = new Editor({
  content: "",
  extensions: [StarterKit],
  editorProps: {
    attributes: {
      style:
        "height: calc(100vh - 32px); font-size: 14px;outline: none; overflow-y: auto;",
    },
  },
});

const title = computed(() => {
  return editor.getText({}).slice(0, 20);
});

const togglePin = () => {
  isPinned.value = !isPinned.value;
};

const toggleMaximize = () => {
  isMaximized.value = !isMaximized.value;
  document.body.style.overflow = isMaximized.value ? "hidden" : "";
};

const toggleHiddenContent = () => {
  isHiddenContent.value = !isHiddenContent.value;
};

const toggleSave = () => {
  // 保存逻辑
};

const togglePalette = () => {
  // 切换主题逻辑
};

// 将透明度添加到十六进制颜色中
function addTransparencyToHexColor(hexColor, transparency) {
  // 移除可能存在的 '#' 符号
  hexColor = hexColor.replace("#", "");

  // 确保输入的是有效的6位十六进制颜色
  if (hexColor.length !== 6) {
    throw new Error("Invalid hex color");
  }

  // 确保透明度值在0到1之间
  if (transparency < 0 || transparency > 1) {
    throw new Error("Transparency must be between 0 and 1");
  }

  // 将透明度转换为两位十六进制
  const alpha = Math.round(transparency * 255)
    .toString(16)
    .padStart(2, "0");

  return `#${hexColor}${alpha}`;
}

const handleBlur = () => {
  editor.commands.blur();
};

const unListen = ref(null);

onMounted(() => {
  appWindow.listen("blur", handleBlur).then((listener) => {
    unListen.value = listener;
  });
});

onUnmounted(() => {
  if (unListen.value) {
    unListen.value();
  }
});
</script>

<template>
  <div class="editor">
    <header
      class="header"
      data-tauri-drag-region
      :style="{ backgroundColor: selectedColor }"
    >
      <div class="left-icon">
        <template v-if="isPinned">
          <div class="icon">
            <PinIcon size="14" @click="togglePin" />
          </div>
        </template>
        <template v-else>
          <div class="icon">
            <PinOffIcon size="14" @click="togglePin" />
          </div>
        </template>

        <ColorPicker v-model="selectedColor">
          <template #trigger>
            <div class="icon">
              <PaletteIcon size="14" />
            </div>
          </template>
        </ColorPicker>

        <div class="icon">
          <CirclePlusIcon size="14" />
        </div>
        <div class="icon">
          <SaveIcon size="14" @click="toggleSave" />
        </div>
        <div data-tauri-drag-region v-if="isHiddenContent" class="title-input">
          {{ title }}
        </div>
      </div>
      <div class="right-icon">
        <div class="icon">
          <CircleChevronDownIcon size="14" @click="toggleHiddenContent" />
        </div>
        <div class="icon">
          <MaximizeIcon size="14" @click="toggleMaximize" />
        </div>
        <div class="icon">
          <CircleMinusIcon size="14" />
        </div>
        <div class="icon">
          <CircleXIcon size="14" />
        </div>
      </div>
    </header>
    <main
      class="container"
      v-show="!isHiddenContent"
      ref="containerRef"
      :style="{
        backgroundColor: addTransparencyToHexColor(selectedColor, 0.6),
      }"
    >
      <editor-content :editor="editor" class="editor-content" />
    </main>
  </div>
</template>

<style scoped>
.editor {
  width: 100%;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 24px;
  width: 100%;
  backdrop-filter: blur(10px);
}

.left-icon,
.right-icon {
  display: flex;
  align-items: center;
  gap: 2px;
}

.title-input {
  margin-left: 8px;
  padding: 4px 0;
  border: none;
  font-size: 12px;
  color: #333;
  background: transparent;
  min-width: 80px;
  user-select: none;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  cursor: default;
}

.container {
  padding: 4px 8px;
  overscroll-behavior-y: contain;
  height: calc(100vh - 32px);
}

.icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  cursor: pointer;
  border-radius: 6px;
  color: #1e1e1e;
  transition: all 0.2s ease;
}

.icon:hover {
  background: rgba(0, 0, 0, 0.06);
  color: #333;
}

:deep(.ProseMirror p) {
  margin: 0;
  line-height: 1.5;
  color: #333;
}

:deep(.ProseMirror p:first-child) {
  margin-top: 0;
}
</style>
