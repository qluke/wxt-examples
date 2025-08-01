## 项目核心功能解释：esm-content-script-ui

---

### 一句话总结

**本项目演示如何将内容脚本预打包为 ESM（原生模块），在浏览器扩展中实现代码分割、动态加载和样式隔离的 UI 挂载。**

---

## 详细功能说明

### 1. ESM 内容脚本分包与动态加载

- **内容脚本主入口**（`entrypoints/content/index.ts`）并不直接承载所有逻辑，而是运行时动态 import 另一个 ESM 文件（`/content-scripts/esm/content.js`）。
- 动态加载的模块（`esm-index.ts`）包含 UI 挂载和样式处理的核心逻辑。
- 这种结构支持**代码分割**，便于按需加载、减少初始体积。

### 2. 动态挂载 UI 到页面（Shadow DOM）

- 动态加载模块后，调用 `createShadowRootUi`，在页面插入 shadow DOM 容器。
- UI 内容全部渲染在 shadow root，**与原网页完全隔离**。
- 挂载的内容为“ESM UI!”文本，实际可替换为任意复杂 UI（如 React/Vue/Svelte）。

### 3. 动态加载、插入作用域 CSS

- ESM 脚本通过 `fetch` 获取预打包好的 CSS 文件内容（`/content-scripts/esm/content.css`）。
- 运行时将 CSS 的 `:root` 选择器替换为 `:host`，保证样式只在 shadow DOM 内生效，实现**样式作用域隔离**。
- 动态创建 `<style>` 标签插入 shadow DOM `<head>`，安全且高兼容。

### 4. 现代开发体验

- 依赖 [WXT](https://wxt.dev) 与 [Vite](https://vitejs.dev) 打包，开发/构建体验快、支持热重载。
- 可用 `pnpm dev` 一键启动开发和调试环境。

---

## 主要源码结构与核心逻辑

### `entrypoints/content/index.ts`

```typescript
export default defineContentScript({
  matches: ["*://*/*"],
  async main(ctx) {
    /* @vite-ignore */
    const mod = await import(
      browser.runtime.getURL("/content-scripts/esm/content.js")
    );
    return await mod.default(ctx);
  },
});
```

- 在每个网页动态加载预打包好的 ESM 内容脚本模块。

---

### `entrypoints/content/esm-index.ts`

```typescript
import { ContentScriptContext } from "wxt/utils/content-script-context";
import "./styles.css";

export default async (ctx: ContentScriptContext) => {
  const { createShadowRootUi } = await import(
    "wxt/utils/content-script-ui/shadow-root"
  );
  const stylesText = await fetch(
    browser.runtime.getURL("/content-scripts/esm/content.css")
  ).then((res) => res.text());
  const ui = await createShadowRootUi(ctx, {
    name: "esm-ui-example",
    position: "inline",
    append: "first",
    onMount(uiContainer, shadow) {
      // 注入作用域 CSS
      const style = document.createElement("style");
      style.textContent = stylesText.replaceAll(":root", ":host");
      shadow.querySelector("head")!.append(style);

      uiContainer.textContent = "ESM UI!";
    },
  });
  ui.mount();
};
```

- 实现动态分包、shadow DOM 挂载、CSS 动态注入与作用域隔离。

---

### `entrypoints/content/styles.css`

```css
:root {
  color: red;
}
```

- 预打包样式，运行时自动只影响 shadow DOM 内内容。

---

## 亮点与实际价值

- **现代 ESM 架构**：支持原生模块化、异步分包、按需加载，适合大型/复杂扩展项目。
- **UI/样式隔离**：通过 shadow DOM 和作用域 CSS，内容脚本 UI 不会污染原网页。
- **灵活易集成**：可与任何前端框架结合，易于扩展为复杂页面助手、分析工具等。
- **开发体验优异**：集成 WXT+Vite，效率高、类型安全、支持热重载。

---

## 适用场景

- 需要内容脚本复杂 UI 且追求性能和隔离性的扩展。
- 多页面共用内容脚本，但需动态加载不同逻辑的场景。
- 现代大型浏览器扩展工程的最佳实践样板。

---

## 总结流程图

```
[内容脚本 index.ts]
    ↓ (动态 import)
[ESM 子包 esm-index.ts]
    ↓ (动态加载工具/样式)
[createShadowRootUi 挂载 UI]
    ↓
[Shadow DOM 注入作用域 CSS + 显示 UI]
```

---

如需进一步了解 ESM 内容脚本的工程化细节、复杂 UI 集成方式等，欢迎继续提问！
