## 项目核心功能解释：react-mantine

---

### 一句话总结

**本项目演示了如何在浏览器扩展（WXT）中，结合 React 与 [Mantine](https://mantine.dev) UI 框架，在内容脚本和弹窗（popup）中实现现代化、主题可配置、组件丰富的 UI，并正确解决 shadow DOM 场景下的样式与变量隔离。**

---

## 主要功能与实现原理

### 1. 内容脚本中挂载 Mantine + React UI

- 通过内容脚本，利用 `createShadowRootUi` 将 Mantine 组件动态挂载到任意网页的 Shadow DOM 内（如网页 `<body>`）。
- 采用 `MantineProvider`，传递自定义 `theme` 和 shadow DOM 定位相关配置参数，使得 Mantine 的 CSS 变量和样式作用于 shadow DOM，不受原网页影响。

### 2. Mantine 的 Shadow DOM 适配

- Mantine 默认不兼容 shadow DOM（内容脚本 UI 会出现主题变量失效、样式覆盖等问题）。
- 本项目通过设置 `cssVariablesSelector="html"` 和 `getRootElement={() => shadow.querySelector("html")!}`，让 Mantine 的 CSS 变量注入 shadow root 内部的 `<html>`，实现主题与样式隔离，UI 100%一致。

### 3. 组件库与主题系统

- 集成 Mantine 的 Button、Container 等所有组件（见 `components/Counter.tsx`），可在内容脚本或 popup 复用。
- 支持自定义主题（`utils/theme.ts`），可一处维护多端统一风格。

### 4. 开发体验与打包

- 依赖 `@wxt-dev/module-react`，兼容 React 18+，支持 TypeScript、自动类型推断、HMR 热重载。
- 使用 PostCSS 集成 `postcss-preset-mantine` 和变量插件，确保 Mantine 样式完整、主题断点生效。

### 5. 弹窗（popup）与内容脚本一致体验

- `popup` 页面直接用 MantineProvider 包裹，UI 与内容脚本完全一致，便于统一开发和维护。
- 组件可在多个上下文复用（popup、内容脚本、options page 等）。

---

## 主要文件结构与说明

- **entrypoints/content.tsx**

  - 通过 `createShadowRootUi` 动态挂载 Mantine + React UI 到网页
  - 关键：在 `onMount` 中设置 MantineProvider 的 shadow DOM 定位参数

- **components/Counter.tsx**

  - Mantine 计数器组件，演示 Mantine 按钮、主题变量、交互效果

- **utils/theme.ts**

  - 自定义 Mantine 主题，可配置全局色彩、字体、断点等

- **postcss.config.cjs**
  - Mantine + PostCSS 断点变量配置，保证响应式样式在 shadow DOM、popup 等场景下生效

---

## 关键代码片段说明

### 内容脚本 UI 挂载（shadow DOM 适配）

```tsx
function createUi(ctx: ContentScriptContext) {
  return createShadowRootUi(ctx, {
    name: "mantine-example",
    position: "inline",
    append: "first",
    onMount(uiContainer, shadow) {
      const app = document.createElement("div");
      uiContainer.append(app);

      const root = ReactDOM.createRoot(app);
      root.render(
        <React.StrictMode>
          <MantineProvider
            theme={theme}
            cssVariablesSelector="html"
            getRootElement={() => shadow.querySelector("html")!}
          >
            <Counter />
          </MantineProvider>
        </React.StrictMode>
      );
      return root;
    },
    onRemove(root) {
      root?.unmount();
    },
  });
}
```

---

### popup/main.tsx

```tsx
ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <MantineProvider theme={theme}>
      <Counter />
    </MantineProvider>
  </React.StrictMode>
);
```

- Popup 直接用 MantineProvider，开发体验与内容脚本一致。

---

### postcss.config.cjs

```js
module.exports = {
  plugins: {
    "postcss-preset-mantine": {},
    "postcss-simple-vars": {
      variables: {
        "mantine-breakpoint-xs": "36em",
        "mantine-breakpoint-sm": "48em",
        "mantine-breakpoint-md": "62em",
        "mantine-breakpoint-lg": "75em",
        "mantine-breakpoint-xl": "88em",
      },
    },
  },
};
```

- 保证响应式断点样式正确注入 shadow DOM。

---

## 亮点与实际价值

- **主流企业级 React 组件库**可直接用于扩展内容脚本，极大丰富 UI 选择
- **主题与样式隔离**，UI 不受网页原有样式污染，支持多皮肤、多主题
- **开发效率提升**，组件可复用于 popup、内容脚本等多端，统一体验
- **现代 React 生态、热重载、类型安全、最佳工程实践**

---

## 适用场景

- 需要复杂 UI（如侧边栏、弹窗、表单、设置面板等）的扩展开发
- 企业级浏览器扩展、团队协作、统一设计系统
- 主题/风格切换、多端 UI 复用

---

## 总结流程图（文字版）

```
[内容脚本注入]
    ↓
[createShadowRootUi 挂载 Shadow DOM]
    ↓
[MantineProvider + 自定义主题，适配 shadow DOM]
    ↓
[组件/UI 隔离，风格一致，支持响应式和多端复用]
```

---

如需补充**Mantine 全量主题扩展、内容脚本与网页交互、复杂表单/弹窗集成**等内容，欢迎继续提问！
