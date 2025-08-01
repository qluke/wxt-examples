## 项目核心功能解释：get-started-page

---

### 一句话总结

**本项目实现了扩展安装时自动弹出“欢迎页”（Get Started），引导用户首次体验，并采用 Chrome 官方推荐的 `chrome.tabs.create` 方式打开扩展内页面。**

---

## 主要功能原理与实现细节

### 1. 扩展安装时自动打开欢迎页

- **事件监听**：后台脚本通过 `browser.runtime.onInstalled` 监听扩展的安装事件。
- **只在首次安装时触发**：判断 `reason === "install"`，避免升级或重载时重复弹窗。
- **打开扩展内页面**：调用 `browser.tabs.create`（等价于官方文档的 `chrome.tabs.create`），新建一个标签页，`url` 指向扩展自身的 `/get-started.html`，并设为激活状态。

> 参考 [chrome.tabs.create 官方文档](https://developer.chrome.com/docs/extensions/reference/api/tabs#open_an_extension_page_in_a_new_tab)：“A common pattern for extensions is to open an onboarding page in a new tab when the extension is installed. The following example shows how to do this.”

**示例代码：**

```ts
browser.runtime.onInstalled.addListener(async ({ reason }) => {
  if (reason !== "install") return;
  await browser.tabs.create({
    url: browser.runtime.getURL("/get-started.html"),
    active: true,
  });
});
```

---

### 2. 欢迎页内容与交互

- **`get-started.html` 是一个简单的扩展内页面**，用于展示欢迎内容或引导说明。
- **页面内有“关闭”按钮**，用户点击后调用 `window.close()` 关闭该标签页（仅限扩展打开的标签页可被脚本关闭）。

```html
<p>Get started content...</p>
<button id="closeBtn">Close</button>
<script type="module" src="./main.ts"></script>
```

```ts
closeBtn.onclick = () => {
  window.close();
};
```

---

### 3. 无需声明特殊权限

- 打开扩展内页面时**不需要声明任何额外权限**，仅需 background/service worker 权限。
- 该模式不会触发权限警告，符合 Chrome Web Store 的最佳实践。

---

## 亮点与实际价值

- **用户体验友好**：安装即展示欢迎/上手说明，提升新用户引导和转化。
- **开发规范**：代码完全符合 Chrome 官方扩展开发建议。
- **使用简单**：只需一行 `tabs.create`，无需复杂配置。

---

## 典型应用场景

- 扩展首次安装欢迎页、功能介绍
- 新版本升级公告页（可扩展 onInstalled 的 reason 为 "update"）
- 引导用户进行首次配置、授权操作

---

## 总结流程图

```
[扩展首次安装]
    ↓
[background onInstalled事件]
    ↓
[tabs.create 打开 /get-started.html]
    ↓
[用户阅读、可点击“Close”关闭标签页]
```

---

## 参考官方示例

> ```javascript
> chrome.runtime.onInstalled.addListener(({ reason }) => {
>   if (reason === "install") {
>     chrome.tabs.create({ url: "onboarding.html" });
>   }
> });
> ```
>
> — [Chrome Developer Docs: Open an extension page in a new tab](https://developer.chrome.com/docs/extensions/reference/api/tabs#open_an_extension_page_in_a_new_tab)

---

如需扩展“升级弹窗”、“多语言欢迎页”、“统计用户首次打开”等进阶用法，欢迎继续提问！
