## 项目核心功能解释

这个项目是一个**浏览器扩展**（基于 [WXT](https://wxt.dev/)），其核心功能是：**只有当用户点击扩展图标（action icon）时，才动态注入 content script 到当前活动标签页（active tab）中**。注入的 content script 会在页面上以 shadow DOM 形式展示一段信息。

---

### 一句话总结

> **只在用户点击扩展图标后，往当前活动的 google.com 网页动态注入脚本和样式，并展示提示内容。**

---

### 工作流程与关键点

1. **权限声明**

   - 需要 `"activeTab"` 和 `"scripting"` 权限，允许在选定页面上动态注入脚本和样式。

2. **点击扩展图标时触发**

   - 监听扩展 action 图标的点击事件。
   - 检查当前 tab 是否为 Google 域名（`*://*.google.com/*`）。

3. **动态注入 content script**

   - 通过 `browser.scripting.executeScript` 方法，在当前 tab 上注入 `/content-scripts/content.js`。
   - content script 没有直接在 manifest 中注册，而是**按需动态注入**，实现了极为精细的权限和注入控制。

4. **content script 作用**

   - 注入后，使用 `createShadowRootUi` 方法，在页面上插入一段 shadow DOM 元素，显示 `"Hello active tab!"`。
   - 注入的样式（`style.css`）只影响该 shadow DOM，内容为全红字黑底。

5. **样式注入**

   - 由于 content script 不是清单 manifest 里直接声明的脚本，CSS 需要作为 `web_accessible_resources` 明确允许被加载。

6. **返回值通信**
   - content script 执行后，会返回 `"Hello world!"`，可在后台（service worker）看到返回值。

---

### 主要文件说明

- `wxt.config.ts`  
  配置扩展权限、web_accessible_resources、启动页面等。

- `entrypoints/background.ts`  
  监听扩展图标点击，动态注入 content script。

- `entrypoints/content/index.ts`  
  被注入页面的脚本，负责渲染 UI 和样式。

- `entrypoints/content/style.css`  
  注入到 shadow DOM 的样式。

- `utils/matches.ts`  
  定义 content script 允许注入的 url 匹配规则。

---

### 典型使用流程

1. 安装扩展，启动开发环境。
2. 打开 [google.com](https://google.com) 页面。
3. 点击扩展 action 图标。
4. 页面上出现红字黑底的 `"Hello active tab!"`，内容只影响本标签页面。
5. 扩展后台可以打印 content script 返回值。

---

### 场景用途

- **按需注入脚本**，避免扩展常驻内容脚本带来的性能和权限问题。
- **只在用户明确触发时运行**，更安全、合规。
- **shadow DOM 隔离 UI**，不会污染原网页样式。

---

### 总结图（流程示意）

1. 用户点击扩展图标 ⟶
2. 检查当前 tab url ⟶
3. 动态注入脚本 ⟶
4. 页面出现扩展自定义 UI（shadow DOM）

---

如需进一步的示意图或流程图，可随时告知！
