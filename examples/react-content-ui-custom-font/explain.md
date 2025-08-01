## 项目核心功能解释：react-content-ui-custom-font

---

### 一句话总结

**本项目演示如何在浏览器扩展的内容脚本中，使用 React + WXT + Shadow DOM 动态挂载 UI，并通过 web_accessible_resources 动态注入自定义字体文件（如 ttf），实现内容脚本内 React 组件的“真·自定义字体”显示。**

---

## 主要功能与实现原理

### 1. 内容脚本中 React UI 动态挂载

- 通过内容脚本（`entrypoints/content/index.tsx`），将 React 组件挂载到任意网页（本例限定 google.com），UI 用 shadow DOM 隔离，避免样式冲突。
- 支持 React 19 语法，组件可以自由扩展交互与样式。

### 2. 动态注入自定义字体

- 通过 WXT 配置（`wxt.config.ts`）将 `/fonts/jbmono.ttf` 声明为 `web_accessible_resources`，允许内容脚本在页面上下文安全加载。
- 在 `onMount` 钩子中，通过 `browser.runtime.getURL("/fonts/jbmono.ttf")` 获取字体绝对 URL，构造 `@font-face` 样式并注入 `<style>`，使 shadow DOM 内的 React UI 可用自定义字体。

### 3. 组件中应用字体

- 在 React 组件（`App.tsx`）里，直接 `style={{ fontFamily: "JB Mono" }}`，即刻生效，无需担心网页原有字体影响。
- 支持多字体、多权重、多格式（ttf/woff/woff2）扩展。

---

## 主要代码结构与说明

### entrypoints/content/index.tsx

```tsx
import ReactDOM from "react-dom/client";
import App from "./App.tsx";

export default defineContentScript({
  matches: ["*://*.google.com/*"],
  cssInjectionMode: "ui",

  async main(ctx) {
    const ui = await createShadowRootUi(ctx, {
      name: "custom-font",
      position: "inline",
      anchor: "body",
      append: "first",
      onMount: (container) => {
        // 获取字体文件URL
        const fontUrl = browser.runtime.getURL("/fonts/jbmono.ttf");
        // 动态注入 @font-face
        const fontStyle = document.createElement("style");
        fontStyle.textContent = `
                    @font-face {
                      font-family: 'JB Mono';
                      src: url('${fontUrl}') format('truetype');
                      font-weight: 400;
                      font-style: normal;
                    }
                `;
        document.head.appendChild(fontStyle);

        const wrapper = document.createElement("div");
        container.append(wrapper);

        const root = ReactDOM.createRoot(wrapper);
        root.render(<App />);
        return { root, wrapper };
      },
      onRemove: (elements) => {
        elements?.root.unmount();
        elements?.wrapper.remove();
      },
    });
    ui.mount();
  },
});
```

- 重点：**动态注入 @font-face，Shadow DOM 内的 React 组件可安全使用自定义字体。**

---

### entrypoints/content/App.tsx

```tsx
export default function App() {
  return (
    <div
      style={{
        fontFamily: "JB Mono",
        position: "fixed",
        top: 0,
        left: 0,
        zIndex: 9999,
      }}
    >
      <h1>Hello Custom Font</h1>
    </div>
  );
}
```

- 组件内容采用自定义字体“JB Mono”，样式效果独立于网页。

---

### wxt.config.ts

```typescript
export default defineConfig({
  modules: ["@wxt-dev/module-react"],
  manifest: {
    web_accessible_resources: [
      { resources: ["fonts/*"], matches: ["*://*.google.com/*"] },
    ],
  },
});
```

- 声明 `/fonts/*` 文件在 google.com 上可被内容脚本访问和注入。

---

## 亮点与实际价值

- **彻底解决内容脚本 UI 字体自定义难题**：即使网页全局被强制字体，扩展 UI 依然独立显示自定义字体。
- **shadow DOM + @font-face 动态注入**，不影响网页原有字体和样式，适合所有复杂 UI 场景。
- **兼容任意字体文件类型和多域名**，支持多种字体和多页面。
- **开发体验现代**，React 19、TypeScript、自动类型推断、WXT 热重载。

---

## 适用场景

- 需要扩展 UI 一致字体标识（如代码助手、浮窗、UI 设计稿展现）
- 字体风格与网页强隔离（如对比、审美、品牌需求）
- 多语言/特殊字符/符号支持

---

## 总结流程图（文字版）

```
[内容脚本注入]
    ↓
[createShadowRootUi 创建 Shadow DOM]
    ↓
[onMount 动态注入 @font-face + 加载字体文件]
    ↓
[React 组件设置 fontFamily: 'JB Mono']
    ↓
[扩展 UI 独立字体，网页字体无影响]
```

---

如需了解**多字体切换、字体预加载优化、web_accessible_resources 跨域安全、React 复杂 UI 与字体适配**等，欢迎继续提问！
