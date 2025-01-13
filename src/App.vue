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
import { ref, onMounted, computed } from "vue";
import { Editor } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";

const isPinned = ref(false);
const filePath = ref("");
const isMaximized = ref(false);
const isHiddenContent = ref(false);
const title = ref("title");

const editor = new Editor({
  content: "<p>tiptap</p>",
  extensions: [StarterKit],
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
</script>

<template>
  <div class="editor">
    <header class="header" data-tauri-drag-region>
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
        <div class="icon">
          <PaletteIcon size="14" @click="togglePalette" />
        </div>
        <div class="icon">
          <CirclePlusIcon size="14" />
        </div>
        <div class="icon">
          <SaveIcon size="14" @click="toggleSave" />
        </div>
        <input
          v-if="isHiddenContent"
          v-model="title"
          class="title-input"
          type="text"
          placeholder="title"
        />
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
    <main class="container" v-show="!isHiddenContent">
      <editor-content :editor="editor" />
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
  background: #f5e942;
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
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  font-size: 13px;
  color: #333;
  background: transparent;
  width: 80px;
}

.title-input:focus {
  outline: none;
  background: rgba(0, 0, 0, 0.05);
}

.title-input::placeholder {
  color: #999;
}

.icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  cursor: pointer;
  border-radius: 6px;
  color: #666;
  transition: all 0.2s ease;
}

.icon:hover {
  background: rgba(0, 0, 0, 0.06);
  color: #333;
}

:deep(.ProseMirror) {
  outline: none;
  min-height: 300px;
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
