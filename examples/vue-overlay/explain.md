## 项目核心功能解释：vue-overlay

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展中，利用 Vue 3 + Shadow DOM 动态在所有网页右上角显示自定义浮层（Overlay），实现强隔离、跨页面、可复用的扩展 UI 组件。**

---

## 主要功能与实现原理

### 1. 内容脚本动态挂载 Vue UI 到 Shadow DOM

- 通过 `defineContentScript`，内容脚本自动注入所有网页（`*://*/*`），动态挂载一个 Vue 应用。
- 利用 `createShadowRootUi`，在页面顶层创建一个 shadow DOM 容器，UI 被完全包裹、样式与原页面隔离。
- Shadow root 的宿主元素设置 `pointer-events: none`，但内部元素（如 overlay 容器）可用 `pointer-events: auto` 控制交互区。

### 2. Vue 3 单文件组件开发体验

- UI 由 SFC（单文件组件，`.vue` 文件）实现，支持 `<script setup>`, `<template>`, `<style scoped>`，开发体验和主流 Vue 项目一致。
- 例如 `App.vue` 中的 overlay 样式、logo、文本完全自定义，可自由扩展为表单、菜单、悬浮工具栏等。

### 3. 支持页面跳转自动重挂载

- 监听 `wxt:locationchange` 事件，每当页面发生 SPA 跳转或动态切换，自动重新挂载 overlay，保证浮层在页面生命周期内始终可见且样式一致。

### 4. 样式隔离与易用性

- 所有样式通过 shadow DOM 隔离，不受原网页干扰，也不会污染原网页。
- 可用 CSS（如 `reset.css`）进一步消除默认样式差异。

### 5. 现代工程体验

- 依赖 `@wxt-dev/module-vue`，支持 TypeScript、类型检查、热重载（HMR）、自动构建。
- 可与 WXT 的内容脚本 API、自动导入等特性无缝集成。

---

## 主要代码结构与说明

### entrypoints/content/index.ts

```typescript
import App from "./App.vue";
import { createApp } from "vue";
import "./reset.css";

export default defineContentScript({
  matches: ["*://*/*"],
  cssInjectionMode: "ui",
  async main(ctx) {
    const ui = await defineOverlay(ctx);
    ui.mount();
    ctx.addEventListener(window, "wxt:locationchange", (event) => {
      ui.mount();
    });
  },
});

function defineOverlay(ctx: ContentScriptContext) {
  return createShadowRootUi(ctx, {
    name: "vue-overlay",
    position: "modal",
    zIndex: 99999,
    onMount(container, _shadow, shadowHost) {
      const app = createApp(App);
      app.mount(container);
      shadowHost.style.pointerEvents = "none"; // 宿主不可交互
      return app;
    },
    onRemove(app) {
      app?.unmount();
    },
  });
}
```

- 挂载与卸载完全自动，兼容 SPA、传统网页。

---

### entrypoints/content/App.vue

```vue
<script lang="ts" setup>
import vueLogo from "~/assets/vue.svg";
</script>

<template>
  <div class="container">
    <img :src="vueLogo" />
    <p>Hello World</p>
  </div>
</template>

<style scoped>
.container {
  /* 右上浮层样式，可自定义 */
}
</style>
```

- 组件样式、结构、交互都可以自定义，适合开发各种浮层、工具条。

---

### wxt.config.ts

```typescript
export default defineConfig({
  modules: ["@wxt-dev/module-vue"],
  webExt: {
    startUrls: ["https://duckduckgo.com"],
  },
});
```

- 配置 WXT 支持 Vue 自动构建和开发。

---

## 亮点与实际价值

- **真正的浮层/Overlay UI**：适合扩展悬浮按钮、页面助手、全局快捷工具等。
- **Vue 3 全生态支持**：单文件组件、Composition API、TypeScript、热重载。
- **样式/DOM 彻底隔离**：不会和网页任何元素冲突，兼容所有网站。
- **跨页面、SPA 兼容**：自动在页面跳转时恢复或重挂载 UI。
- **可拓展性极强**：可用 Vue 生态任意组件、插件、动画、状态管理等。

---

## 适用场景

- 扩展内全局浮窗、悬浮菜单、快捷入口、页面分析助手、内容采集等场景
- 希望 UI/样式完全隔离网页且高度可定制的扩展开发
- 需要 Vue 生态能力、团队协作、组件化复用

---

## 总结流程图（文字版）

```
[内容脚本注入]
    ↓
[createShadowRootUi 创建 shadow DOM]
    ↓
[Vue 3 应用挂载 shadow root]
    ↓
[页面右上角显示自定义浮层]
    ↓
[SPA 跳转自动重挂载]
```

---

如需补充**复杂浮窗管理、多页面通信、与 popup/后台协作、Vuex/Pinia 状态共享**等内容，欢迎继续提问！
