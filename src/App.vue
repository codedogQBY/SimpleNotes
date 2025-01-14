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
import { ref, onMounted, onUnmounted } from "vue";
import { Editor, EditorContent } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import { appWindow, WebviewWindow, PhysicalSize } from "@tauri-apps/api/window";
import ColorPicker from "./components/ColorPicker.vue";
import { defaultColorList } from "./config.js";
import { writeFile, BaseDirectory } from "@tauri-apps/api/fs";
import { save, ask } from "@tauri-apps/api/dialog";
import { Markdown } from "tiptap-markdown";

const containerRef = ref(null);
const isPinned = ref(true);
const filePath = ref("");
const isMaximized = ref(false);
const isHiddenContent = ref(false);
// 默认随机颜色
const selectedColor = ref(
  defaultColorList.filter(() => Math.random() > 0.5)[0]
);

const editor = new Editor({
  content: "",
  extensions: [StarterKit, Markdown],
  editorProps: {
    attributes: {
      style:
        "height: calc(100vh - 32px); font-size: 14px;outline: none; overflow-y: auto;",
    },
  },
});

const title = ref("");

editor.on("update", () => {
  title.value = editor.getText({}).slice(0, 10);
  appWindow.setTitle(title.value);
});

async function setAlwaysOnTop(alwaysOnTop) {
  try {
    // 将窗口固定到最高层
    await appWindow.setAlwaysOnTop(alwaysOnTop);
  } catch (error) {}
}

const togglePin = () => {
  isPinned.value = !isPinned.value;
  setAlwaysOnTop(isPinned.value);
};

const toggleMaximize = () => {
  isMaximized.value = !isMaximized.value;
  appWindow.setFullscreen(isMaximized.value);
};

// 保存原始窗口大小
const originalSize = ref({
  width: 300,
  height: 200,
});

const toggleHiddenContent = async () => {
  isHiddenContent.value = !isHiddenContent.value;
  const factor = await appWindow.scaleFactor();
  if (isHiddenContent.value) {
    const { width, height } = await appWindow.outerSize();
    originalSize.value = {
      width,
      height,
    };
    await appWindow.setSize(new PhysicalSize(width, 24 * factor));
  } else {
    await appWindow.setSize(
      new PhysicalSize(originalSize.value.width, originalSize.value.height)
    );
  }
};

const saveFile = async (path) => {
  try {
    if (path) {
      const markdownOutput = editor.storage.markdown.getMarkdown();
      await writeFile(path, markdownOutput, {
        dir: BaseDirectory.Document,
        append: true,
      });
    }
  } catch (error) {
    console.error(error);
  }
};

const toggleSave = async () => {
  try {
    if (!filePath.value) {
      const result = await save({
        defaultPath: `${title.value}.md`,
        filters: [
          {
            name: "Markdown",
            extensions: ["md"],
          },
        ],
      });
      if (result) {
        filePath.value = result;
        await saveFile(result);
      }
    } else {
      await saveFile(filePath.value);
    }
  } catch (error) {
    console.error(error);
  }
};

const createNewNote = async () => {
  // 生成16位随机数
  const randomId = Math.random().toString(16).slice(2, 8).toString();

  // 新窗口初始位置在当前窗口的右下角（30，30）处
  const { x, y } = await appWindow.outerPosition();

  const newWindow = new WebviewWindow(randomId, {
    url: "/", // 可以是本地文件或远程 URL
    fullscreen: false,
    resizable: true,
    width: 300,
    height: 200,
    minWidth: 270,
    minHeight: 24,
    decorations: false,
    x: x + 30,
    y: y + 30,
  });
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

const toggleClose = async () => {
  const answer = await ask("是否保存当前内容后关闭？", {
    title: "保存当前内容",
    type: "warning",
    okLabel: "保存",
    cancelLabel: "不保存",
  });
  if (answer) {
    await toggleSave();
  }
  await appWindow.close();
};

const unListen = ref(null);

onMounted(() => {
  appWindow.listen("blur", handleBlur).then((listener) => {
    unListen.value = listener;
  });
  setAlwaysOnTop(isPinned.value);
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
          <CirclePlusIcon size="14" @click="createNewNote" />
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
          <CircleMinusIcon size="14" @click="appWindow.minimize()" />
        </div>
        <div class="icon">
          <CircleXIcon size="14" @click="toggleClose" />
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
  margin-left: 4px;
  border: none;
  font-size: 12px;
  color: #333;
  background: transparent;
  width: 80px;
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
  position: relative;
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
