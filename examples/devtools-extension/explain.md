## 项目核心功能解释：devtools-extension

---

### 一句话总结

**本项目实现了一个为浏览器 DevTools（开发者工具）添加自定义面板（tab）和侧边栏（pane）的扩展样板，演示如何用扩展 API 向 DevTools 插件化地注入自定义 UI。**

---

## 主要功能与交互流程

### 1. 扩展为开发者工具添加自定义入口

- **自定义面板（Panel）**  
  在 DevTools 顶部 tab 区域新增一个 `"Example Panel"`，点击后显示自定义页面（`devtools-panel.html`）。
- **自定义侧边栏（Sidebar Pane）**  
  在 Elements（元素）面板右侧，新增 `"Example Pane"`，展示自定义内容（`devtools-pane.html`）。

### 2. 实现方式

- **devtools entrypoint (`entrypoints/devtools/main.ts`)**

  ```typescript
  browser.devtools.panels.create(
    "Example Panel", // 面板标题
    "icon/128.png", // 图标
    "devtools-panel.html" // 面板内容页面
  );

  browser.devtools.panels.elements.createSidebarPane("Example Pane", (pane) => {
    pane.setPage("devtools-pane.html");
  });
  ```

  - 上述 API 分别插入了一个新的 DevTools tab 和一个新的元素面板侧边栏。

- **自定义页面内容**
  - `entrypoints/devtools-panel/index.html`：面板内容页面，可自定义 HTML/JS 逻辑
  - `entrypoints/devtools-pane/index.html`：侧边栏内容页面，可自定义 HTML/JS 逻辑

### 3. 扩展加载与效果

- 开发时执行 `pnpm dev`，用 WXT 启动扩展环境
- 在任意网页打开 DevTools，顶部会多出 `Example Panel`，Elements 面板右侧会多出 `Example Pane`
- 这两个面板都能加载扩展自己定义的 HTML/JS 页面

---

## 主要文件结构和说明

- `entrypoints/devtools/main.ts`：注册自定义面板和侧边栏的脚本
- `entrypoints/devtools-panel/index.html` + `main.ts`：自定义面板的内容页面
- `entrypoints/devtools-pane/index.html` + `main.ts`：自定义侧边栏的内容页面
- `public/icon/128.png`：面板图标

---

## 典型开发体验流程

1. 安装依赖，启动 `pnpm dev`
2. 在浏览器扩展环境下访问网页，打开开发者工具（F12）
3. 顶部多出 `Example Panel`，点击可见自定义内容
4. 在 Elements 元素检查页右侧多出 `Example Pane`，可展示调试信息、工具栏等 UI

---

## 亮点与实际价值

- **DevTools 插件开发基础样板**：快速搭建自定义调试/分析/辅助工具入口
- **面板与侧边栏**：可独立加载 HTML/JS，功能扩展灵活
- **代码结构清晰、WXT 支持热重载**：便于本地调试和快速开发

---

## 适用场景

- 页面调试增强/辅助 UI（如自动化测试、性能分析、元素抓取等）
- 页面数据分析工具
- 专业开发流程插件（如样式/DOM/网络监控自定义面板）

---

## 总结流程图（文字版）

```
[扩展注册 DevTools 面板/侧边栏]
      ↓
[打开 DevTools]
   ├─ [顶部多出 Example Panel → 显示扩展自定义页面]
   └─ [Elements 右侧多出 Example Pane → 显示扩展自定义内容]
```

---

如需补充**API 说明、示例扩展示意图、复杂通信场景（与 background/content script 结合）**，请随时继续提问！
