## 项目核心功能解释：inject-script

---

### 一句话总结

**本项目演示如何通过内容脚本，动态将扩展自身的 JS 文件注入到页面上下文，实现扩展与网页原生 JS 环境的直接交互。**

---

## 主要功能与实现原理

### 1. 内容脚本动态注入 JS 文件

- **内容脚本**（`entrypoints/content.ts`）在所有网页注入（`matches: ["*://*/*"]`）。
- 执行时调用 `injectScript("/injected.js", { keepInDom: true })`，将扩展内的 `injected.js` 文件插入到页面 `<script>` 标签中。
- 注入后的脚本在**页面原生 JS 上下文**中执行，可直接访问网页的全局变量和对象（而内容脚本本身是隔离环境）。

### 2. 被注入脚本的内容与行为

- `entrypoints/injected.ts` 被打包为 `/injected.js`，内容简单打印一条日志 `Hello from injected.ts`。
- 实际应用中可以用来注入任意自定义功能，比如 hook 原生 JS、注入第三方库、页面自动化等。

### 3. 脚本文件的可访问性配置

- 在 `wxt.config.ts` 里用 `web_accessible_resources` 显式声明 `injected.js` 文件，使其允许被内容脚本引用并注入到页面。
- `"matches": ["*://*/*"]` 代表所有网页都允许注入该脚本。

### 4. 注入 API 细节

- `injectScript` 提供了 `keepInDom: true` 选项，保证 `<script>` 节点注入后不会立即被移除，适合持久化注入逻辑。
- 注入完成后内容脚本继续执行，可监听和调试注入效果。

---

## 主要代码结构与说明

### content.ts

```typescript
export default defineContentScript({
  matches: ["*://*/*"],
  async main() {
    console.log("Injecting script...");
    await injectScript("/injected.js", {
      keepInDom: true,
    });
    console.log("Done!");
  },
});
```

- 入口内容脚本，负责注入扩展的 JS 文件到网页。

---

### injected.ts

```typescript
export default defineUnlistedScript(() => {
  console.log("Hello from injected.ts");
});
```

- 注入后在页面直接运行的脚本，可直接访问页面 JS 环境。

---

### wxt.config.ts

```typescript
export default defineConfig({
  manifest: {
    web_accessible_resources: [
      {
        resources: ["injected.js"],
        matches: ["*://*/*"],
      },
    ],
  },
  webExt: {
    startUrls: ["https://wxt.dev"],
  },
});
```

- 配置扩展允许 `injected.js` 被内容脚本注入所有网页。

---

## 亮点与实际价值

- **实现扩展与页面 JS 环境的桥接**，可直接读写页面变量、hook 原生 API、与第三方库交互。
- **通用注入模式**：适合绝大多数需要“越权”到页面上下文的扩展场景。
- **严格受控**：只有显式声明为 `web_accessible_resources` 的脚本才能被注入，安全可控。

---

## 适用场景

- 页面自动化、脚本增强、拓展网页原生功能
- hook/patch 页面原生 JS（如拦截 AJAX、修改全局变量、收集事件等）
- 与页面已加载的第三方库直接交互
- 解决内容脚本隔离沙箱无法满足的高级需求

---

## 总结流程图（文字版）

```
[内容脚本注入]
    ↓
[injectScript("/injected.js")]
    ↓
[页面原生上下文执行 injected.js]
    ↓
[页面全局可见操作/日志/变量]
```

---

如需补充**注入与通信双向桥接（如 window.postMessage 方案）、高级注入场景、动态脚本生成技术**等内容，欢迎继续提问！
