## 项目核心功能解释：react-content-script-ui

---

### 一句话总结

**本项目演示了如何在浏览器扩展的内容脚本中，利用 React + WXT 框架，将完整的 React 组件 UI 动态挂载到任意网页（通过 Shadow DOM 隔离），实现扩展级页面内交互和样式隔离。**

---

## 主要功能与实现原理

### 1. 内容脚本中挂载 React 应用

- 通过内容脚本入口（`entrypoints/content/index.tsx`），自动注入到所有网页（`matches: ["*://*/*"]`）。
- 使用 `createShadowRootUi` 工具，将 UI 挂载到页面 `<body>`，并包裹在独立的 Shadow Root 中，**确保扩展 UI 与页面原有样式完全隔离**。
- 在 `onMount` 钩子中，用 ReactDOM 渲染自定义组件（`App.tsx`）。

### 2. React 组件与交互

- 组件示例（`App.tsx`）是一个带状态的计数器，点击按钮可自增，演示了 React 状态和事件在内容脚本中的兼容性。
- 你可以自由扩展为复杂表单、菜单、浮窗、弹窗等任意 React UI。

### 3. 样式隔离与注入

- 样式文件（`style.css`）通过 `cssInjectionMode: "ui"`，自动注入到 Shadow DOM 内，保证扩展样式不污染原网页，也不受网页样式影响。
- 支持自定义主题、reset 等任意 CSS。

### 4. 开发与依赖

- 依赖 `@wxt-dev/module-react`，自动启用 React 相关的打包与类型支持。
- 兼容 React 18+，支持现代 React API 与类型推断。
- 支持 TypeScript 全类型检查和 JSX 语法。

---

## 主要文件与代码说明

### entrypoints/content/index.tsx

```tsx
import "./style.css";
import ReactDOM from "react-dom/client";
import App from "./App.tsx";

export default defineContentScript({
  matches: ["*://*/*"],
  cssInjectionMode: "ui",

  async main(ctx) {
    const ui = await createShadowRootUi(ctx, {
      name: "wxt-react-example",
      position: "inline",
      anchor: "body",
      append: "first",
      onMount: (container) => {
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

- 重点：**createShadowRootUi + ReactDOM.createRoot** 实现内容脚本内的 React UI 动态挂载与卸载。

---

### entrypoints/content/App.tsx

```tsx
import { useState } from "react";

export default () => {
  const [count, setCount] = useState(1);
  return (
    <div>
      <p>This is React. {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
};
```

- 演示了内容脚本内 React 状态与事件的标准写法。

---

### style.css

```css
* {
  padding: 0;
  margin: 0;
}
body {
  background-color: black;
  padding: 16px;
}
```

- 所有样式仅作用于 Shadow DOM 下的 React UI，不影响原网页。

---

## 亮点与实际价值

- **极致隔离**：Shadow DOM 保证扩展 UI 和网页互不干扰，兼容所有网站。
- **现代 React 能力**：内容脚本可用 hooks、状态、事件、组件拆分等完整 React 生态。
- **动态挂载与卸载**：无需修改网页结构，随时可添加/移除 UI。
- **易于扩展**：可用于悬浮工具条、弹窗、辅助面板、自动化交互等任意场景。

---

## 适用场景

- 网页助手、自动化脚本、内容增强扩展
- 需要复杂 UI 但又要保证与原页面“零耦合”的功能
- 跨平台内容脚本开发（Chrome、Firefox、Edge 等）

---

## 总结流程图（文字版）

```
[内容脚本注入]
    ↓
[createShadowRootUi 创建 Shadow DOM]
    ↓
[ReactDOM 渲染 React 组件到 Shadow DOM]
    ↓
[样式与 UI 隔离，支持动态交互]
```

---

如需探索**与原网页通信、复杂组件集成、跨域样式兼容、消息桥接等高级用法**，欢迎继续提问！
