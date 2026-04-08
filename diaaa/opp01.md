🚀 核心思路

1. ​**CSS Modules 穿透**​：使用 `:global` 覆盖 `el-dialog` 的内部样式。
2. ​**Flex 布局**​：将 `.el-dialog` 设为 `flex` 容器，固定 header 和 footer，让 body 自动伸缩。
3. ​**媒体查询**​：通过 `@media` 区分 PC 端和移动端样式。

---

💻 代码实现

1. 结构与逻辑 (`DialogComponent.vue`)

```typescript
<template>
  <el-dialog
    v-model="visible"
    title="提示标题"
    :class="$style.customDialog"
    :align-center="true"
    destroy-on-close
  >
    <div :class="$style.contentBox">
      <!-- 模拟长内容 -->
      <p v-for="i in 50" :key="i">这是第 {{ i }} 行内容，测试滚动效果。</p>
    </div>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="visible = false">确定</el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const visible = ref(true);
</script>

<style module lang="scss">
/* 使用 :global 穿透覆盖 Element Plus 默认样式 */
.customDialog {
  :global {
    .el-dialog {
      display: flex;
      flex-direction: column;
      padding: 0; /* 清除默认内边距以便自定义 header 背景 */
      background: #fff;
      overflow: hidden;
    }

    .el-dialog__header {
      margin-right: 0;
      padding: 16px 20px;
      background-color: #ffffff; /* 1. 标题栏背景白色 */
      border-bottom: 1px solid #f0f0f0;
    }

    .el-dialog__body {
      flex: 1;
      overflow-y: auto; /* 4. 仅内容部分滚动 */
      padding: 20px;
    }

    .el-dialog__footer {
      padding: 10px 20px;
      border-top: 1px solid #f0f0f0;
    }
  }
}

/* 2. PC 浏览器方式 */
@media (min-width: 768px) {
  .customDialog :global(.el-dialog) {
    height: 600px; /* 2.2 固定高度 */
    width: 50%;
    max-width: 800px;
  }
}

/* 3. Mobile 移动端方式 */
@media (max-width: 767px) {
  .customDialog :global(.el-dialog) {
    margin: 0 !important;
    position: fixed;
    bottom: 0; /* 3.1 从底部开始 */
    left: 0;
    width: 100% !important; /* 3.2 宽度 100% */
    max-height: 80vh; /* 3.3 窗口最大高度限制 */
    height: auto; /* 3.3 根据内容自动高度 */
    border-radius: 12px 12px 0 0;
  }
}

.contentBox {
  /* 确保内部容器配合滚动 */
  height: 100%;
}
</style>
```

---
🔍 关键点解析

🏗️ 布局控制

* ​**`display: flex`**​: 给 `.el-dialog` 增加 flex 布局。
* ​**`flex: 1`**​: 给 `.el-dialog__body` 设置 `flex: 1` 配合 `overflow-y: auto`。
* ​**结果**​: 无论对话框多高，​**标题栏和底部按钮始终固定**​，只有中间内容区会产生滚动条。

📱 移动端适配 (Bottom Sheet 效果)

* ​**`position: fixed` ​+ ​`bottom: 0`**​: 强制对话框吸附在屏幕底部。
* ​**`max-height`**​: 保证内容极多时不会撑满全屏，保留顶部视觉间隙。
* ​**`height: auto`**​: 当内容较少时，对话框高度会随内容收缩，符合移动端交互习惯。

🎨 样式隔离

* ​**`$style.customDialog`**​: 利用 CSS Modules 确保样式只作用于当前组件。
* ​**`:global`**​: 必须配合 `:global` 才能选中 Element Plus 自动生成的 DOM 节点。

---
