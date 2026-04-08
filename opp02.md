​**直接使用的 ​`TestDialog.vue`（Vue3 + TS + Element Plus + CSS Module）**​，已经把你所有需求都整合进去了，包括：

* PC / Mobile 自适应
* 底部弹出（Mobile）
* 固定高度 + 内容滚动（PC）
* 自适应高度 + 最大高度（Mobile）
* 标题固定、内容区滚动
* CSS Module 兼容 Element Plus

---

# ✅ TestDialog.vue（完整版）

```typescript
<template>
  <el-dialog
    v-model="visible"
    :show-close="true"
    :close-on-click-modal="false"
    :class="$style.dialogWrapper"
    :modal-class="$style.modal"
    destroy-on-close
  >
    <!-- Header -->
    <template #header>
      <div :class="$style.header">
        <span>{{ title }}</span>
      </div>
    </template>

    <!-- Body -->
    <div :class="$style.body">
      <!-- 模拟内容 -->
      <div
        v-for="i in 30"
        :key="i"
        :class="$style.item"
      >
        内容项 {{ i }}
      </div>

      <!-- 也支持 slot -->
      <slot />
    </div>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref } from 'vue'

defineProps<{
  title: string
}>()

const visible = ref(true)

// 如果你需要对外控制，可以改成 defineExpose
// defineExpose({ visible })
</script>

<style module>
/* =========================
   通用基础样式
========================= */
.dialogWrapper :global(.el-dialog) {
  padding: 0;
  border-radius: 12px;
  overflow: hidden;

  display: flex;
  flex-direction: column;
}

/* 遮罩层（可选） */
.modal {
  /* 可自定义遮罩透明度 */
}

/* =========================
   Header（固定不滚动）
========================= */
.header {
  background: #fff;
  padding: 16px;
  font-size: 16px;
  font-weight: bold;
  border-bottom: 1px solid #eee;

  flex-shrink: 0; /* 防止被压缩 */
}

/* =========================
   Body（唯一滚动区域）
========================= */
.body {
  padding: 16px;
  overflow-y: auto;

  /* 提升滚动体验 */
  -webkit-overflow-scrolling: touch;
}

/* 示例内容 */
.item {
  padding: 12px;
  border-bottom: 1px solid #f0f0f0;
}

/* =========================
   PC 模式
========================= */
@media (min-width: 768px) {
  .dialogWrapper :global(.el-dialog) {
    width: 600px;
    height: 500px; /* 固定高度 */

    margin: auto; /* 居中（默认其实也会居中） */
  }

  .body {
    flex: 1; /* 填充剩余空间 */
    overflow-y: auto;
  }
}

/* =========================
   Mobile 模式（重点）
========================= */
@media (max-width: 767px) {
  .dialogWrapper :global(.el-dialog) {
    width: 100%;
    margin: 0 !important;

    position: fixed;
    bottom: 0;
    left: 0;

    border-radius: 16px 16px 0 0;

    max-height: 80vh; /* 最大高度限制 */
    height: auto;

    display: flex;
    flex-direction: column;

    /* 动画更自然 */
    animation: slideUp 0.25s ease-out;
  }

  .body {
    overflow-y: auto;
    max-height: calc(80vh - 60px); /* header高度预留 */
  }
}

/* =========================
   动画（Mobile）
========================= */
@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}
</style>
```

---

# ✅ 使用方式

```typescript
<TestDialog title="测试弹窗">
  <div>这里是自定义内容</div>
</TestDialog>
```

---

# ✅ 这版已经满足你的所有要求

### ✔ 1. 标题栏白色

✔ `.header { background: #fff }`

---

### ✔ 2. PC

* 居中（默认）
* 固定高度 `500px`
* 内容滚动（body）

---

### ✔ 3. Mobile

* 从底部弹出 ✅
* 宽度 100% ✅
* 高度自适应 + 最大 80vh ✅
* 超出滚动 ✅

---

### ✔ 4. 滚动只在内容区

👉 通过：

```css
.el-dialog {
  display: flex;
  flex-direction: column;
}

.body {
  overflow-y: auto;
}
```

实现

---

# ✅ 可选增强（你可能会想要）

如果你后面想升级成更“像 App”的效果，我可以帮你加：

### 🚀 拖拽关闭（很高级）

类似 iOS bottom sheet

### 🚀 多段高度（50% / 80%）

类似：

* 半屏
* 全屏

### 🚀 动态内容高度监听（自动撑开）
