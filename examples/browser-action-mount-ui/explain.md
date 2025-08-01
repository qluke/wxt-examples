## 项目核心功能解释：browser-action-mount-ui

---

### 一句话总结

**这是一个实现“点击浏览器扩展图标，动态将 React UI 挂载到当前网页（页面内显示 React 应用）”的浏览器扩展。**

---

## 主要功能与交互流程

### 1. 用户在任意网页点击扩展图标

- 扩展没有弹窗（popup），点击图标后会**触发后台脚本中的事件**。

### 2. 后台脚本向当前标签页内容脚本发消息

- 后台监听 action 图标点击事件（`browser.action.onClicked.addListener`）。
- 触发后，向当前 tab 发送 `{ type: "MOUNT_UI" }` 消息。

### 3. 内容脚本接收消息、挂载 UI

- 内容脚本（`content/index.tsx`）已自动注入所有网页。
- 内容脚本收到 `"MOUNT_UI"` 消息后，调用 `ui.mount()`，用 shadow DOM 的方式将 React 应用挂载到网页 `<body>` 内。
- 挂载的 UI 是用 React + ReactDOM 渲染的组件（`App.tsx`），有计数器、logo、文案等。

### 4. UI 组件隔离渲染

- 通过 `createShadowRootUi`，UI 被放在 shadow DOM 容器中，**样式与原网页互不影响**，可自由设计样式和布局。

### 5. UI 可重复挂载/卸载

- 每次点击扩展图标都可以重新挂载 UI，且支持 HMR（开发时热更新）。

---

## 主要文件及核心代码说明

### background.ts（后台脚本）

```typescript
export default defineBackground(() => {
  (browser.action ?? browser.browserAction).onClicked.addListener(
    async (tab) => {
      if (tab.id) {
        await browser.tabs.sendMessage(tab.id, { type: "MOUNT_UI" });
      }
    }
  );
});
```

- 监听图标点击，通知当前 tab 的内容脚本执行 UI 挂载。

---

### content/index.tsx（内容脚本）

```tsx
export default defineContentScript({
  matches: ["*://*/*"],
  async main(ctx) {
    const ui = await createShadowRootUi(ctx, {
      name: "wxt-react-example",
      position: "inline",
      anchor: "body",
      append: "first",
      onMount: (container) => {
        // React 17+ Root 挂载
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

    browser.runtime.onMessage.addListener((event) => {
      if (event.type === "MOUNT_UI") {
        ui.mount();
      }
    });
  },
});
```

- 监听后台发来的 `"MOUNT_UI"` 消息，挂载 shadow DOM UI。

---

### content/App.tsx（React 组件）

```tsx
function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <div>
        <a href="https://wxt.dev" target="_blank">
          <img src={wxtLogo} className="logo" alt="WXT logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>WXT + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the WXT and React logos to learn more
      </p>
    </>
  );
}
```

- 这是显示在网页上的实际 UI，计数器、logo、文案均可自定义。

---

### 其他特性

- **样式文件**：`App.css`、`style.css` 用于 UI 主题和按钮、文字等样式美化。
- **React + Vite + WXT**：开发体验现代，支持热更新。
- **依赖隔离**：UI 放在 shadow DOM，样式与网页隔离。

---

## 典型使用场景

- 在任意网页上“一键挂载”自己的 UI 工具面板，例如：悬浮菜单、页面助手、调试工具、采集面板等。
- 不干扰原网页，也不需要每次都自动展示，**只有用户点击扩展时才出现**，用户可控。

---

## 总结流程图（文字版）

```
[用户点击扩展图标]
        ↓
[后台脚本] --sendMessage--> [内容脚本]
                                ↓
           [在网页中挂载 React UI（shadow DOM）]
```

---

## 亮点

- **主流前端技术栈**（React）与扩展内容脚本完美结合
- **动态挂载/卸载 UI**，不侵入网页原有 UI
- **样式/行为完全隔离**
- **易于扩展成更复杂的页面助手或开发工具**

---

如需补充**流程图、完整代码注释**或更详细的原理剖析，请随时告知！
