## 项目核心功能解释：offscreen-document-domparser

---

### 一句话总结

**本项目演示了如何利用 Chrome 扩展的 Offscreen Document 能力，在后台安全地离屏处理和解析 HTML（如用 DOMParser 分析页面内容），实现与前台页面完全隔离的异步 DOM 操作。**

---

## 主要功能与实现原理

### 1. Offscreen Document 离屏处理

- **Offscreen Document** 是 Chrome 扩展 API 提供的一种无界面隐藏页面，适用于不能直接在 Service Worker 执行的 DOM 操作（如 DOMParser、Canvas、视频处理等）。
- 本项目用 offscreen document 解析 HTML 字符串，统计 `<script>` 标签数量，并通过消息通信回传给后台脚本。

### 2. 主要交互流程

#### 步骤概述

1. 用户在任意网页点击扩展图标。
2. **后台脚本**用 `browser.scripting.executeScript` 拉取当前页面完整 HTML。
3. 后台判断 offscreen document 是否存在，如不存在则自动创建。
4. 后台通过消息（`browser.runtime.sendMessage`）将 HTML 字符串发给 offscreen document。
5. **offscreen document** 用 DOMParser 解析 HTML，数出 `<script>` 标签数量。
6. 解析结果再通过消息回传后台，后台在控制台输出。

#### 关键点

- 所有 DOM 解析都在 offscreen document 中完成，**不会污染扩展主线程或真实页面**。
- 支持多次调用、自动创建/关闭 offscreen document，无内存泄漏风险。

---

## 主要核心代码结构与说明

### background.ts（后台脚本）

- 监听 action 图标点击，获取当前 tab HTML。
- 自动管理 offscreen document（检测、创建、关闭）。
- 通过消息与 offscreen 通信，接收统计结果。

```typescript
browser.action.onClicked.addListener(async (tab) => {
  await closeOffscreenDocument(); // 保证干净环境
  // 抓取页面 HTML
  const [{ result }] = await browser.scripting.executeScript({
    target: { tabId: tab.id },
    func: () => document.documentElement.outerHTML,
  });
  if (result) {
    await sendMessageToOffscreenDocument(OFFSCREEN_KEYS.SCRIPT_COUNTS, result);
  }
});

// 创建/关闭/检测 offscreen document 的辅助函数...
```

---

### offscreen/offscreen.ts（offscreen document 脚本）

- 监听消息，接收 HTML 字符串。
- 用 DOMParser 解析，统计 `<script>` 数量。
- 通过消息回传统计结果。

```typescript
browser.runtime.onMessage.addListener(handleMessages);

async function handleMessages(message: any) {
  if (message?.target !== MESSAGE_TARGET.OFFSCREEN || !message?.type) return;
  switch (message.type as OFFSCREEN_KEYS) {
    case OFFSCREEN_KEYS.SCRIPT_COUNTS:
      scriptCounts(message.data);
      break;
  }
}

function scriptCounts(htmlString: string) {
  const parser = new DOMParser();
  const document = parser.parseFromString(htmlString, "text/html");
  const count = document.querySelectorAll("script").length;
  sendToBackground(OFFSCREEN_KEYS.SCRIPT_COUNTS, count.toString());
}
```

---

### utils/constants.ts

- 定义消息目标（background、offscreen）和消息类型（如 SCRIPT_COUNTS）。
- 统一常量，方便多端通信。

---

### wxt.config.ts & 权限配置

- 需声明 `"offscreen"`、`"activeTab"` 权限。
- `"offscreen"` 允许创建离屏页面，`"activeTab"` 允许注入脚本抓取页面内容。

---

## 亮点与实际价值

- **安全隔离 DOM 操作**：所有敏感解析在独立的 offscreen 环境内，避免污染页面/后台线程。
- **异步、可扩展**：可处理任意 HTML 片段，应用于抓取、转码、标签统计、内容抽取等复杂任务。
- **自动管理生命周期**：后台脚本负责检测/创建/关闭 offscreen，杜绝内存泄漏。
- **兼容性与最佳实践**：完全遵循 [Chrome Offscreen API 官方文档](https://developer.chrome.com/docs/extensions/reference/api/offscreen)。

---

## 典型场景

- 离屏 HTML 内容抓取与分析（如页面内容提取、SEO 分析、批量统计标签等）
- Canvas、音视频处理、OCR 等只能在 DOM 环境运行的扩展功能
- 任何需异步 DOM 操作但又不想影响页面主线程的浏览器扩展

---

## 总结流程图（文字版）

```
[用户点击扩展]
   ↓
[后台脚本抓取当前页面HTML]
   ↓
[如无offscreen，则自动创建]
   ↓
[消息发送HTML到offscreen]
   ↓
[offscreen用DOMParser解析、统计<script>数]
   ↓
[结果回传后台，后台控制台输出]
```

---

如需补充**多类型 DOM 分析、复杂内容抽取方案、Offscreen 生命周期管理技巧**，欢迎继续提问！
