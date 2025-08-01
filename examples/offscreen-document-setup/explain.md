## 项目核心功能解释：offscreen-document-setup

---

### 一句话总结

**本项目演示了如何在 Chrome 扩展中“最小成本”地创建和初始化一个 Offscreen Document，为后续在后台安全使用 DOM、剪贴板等浏览器能力打下基础。**

---

## 主要功能与实现原理

### 1. Offscreen Document 的基本创建

- **Offscreen Document** 是 Chrome 扩展专有的无界面后台页面，允许在 Service Worker 或后台环境下安全运行需要 DOM、剪贴板等 API 的 JS 逻辑。
- 本项目通过后台脚本，调用 `browser.offscreen.createDocument`，在扩展运行时创建离屏（offscreen）的 HTML 文档。

### 2. 权限和配置

- `manifest` 需声明 `"offscreen"` 权限，允许扩展使用该 API。
- 创建文档时需注明目的（如 `"CLIPBOARD"`），并填写 `justification`（出于什么理由需要后台页面，Chrome 审核要求）。

### 3. Offscreen 页面的内容与通信

- `entrypoints/offscreen/index.html` 是被创建的 offscreen 页，加载 `main.ts`。
- `main.ts` 仅做简单演示：`console.log("Offscreen document is working!", window);`，可用于调试和查看可用 API。
- 实际项目可在此页面中使用 DOMParser、Canvas、剪贴板等前台无法直接访问的特性。

### 4. 最小复现与扩展性

- 这是一个**只演示如何创建 offscreen document 的最小可运行模板**，后续可在 offscreen 环境实现任意 DOM 操作、数据处理等需求。
- 只需简单几行代码即可集成到任何需要后台 DOM 能力的扩展项目。

---

## 主要代码结构与说明

### background.ts

```typescript
export default defineBackground(() => {
  // 创建 offscreen document，理由为 CLIPBOARD
  // @ts-expect-error: MV3 only API not typed
  browser.offscreen.createDocument({
    url: "/offscreen.html",
    reasons: ["CLIPBOARD"],
    justification: "<your justification here>",
  });
});
```

---

### offscreen/index.html & main.ts

```html
<!-- offscreen document HTML，仅包含 main.ts 脚本 -->
<body>
  <script type="module" src="./main.ts"></script>
</body>
```

```typescript
// offscreen/main.ts
console.log("Offscreen document is working!", window);
```

- 加载后即可在 Service Worker 控制台看到 offscreen 页面的 log。

---

## 亮点与实际价值

- **零业务依赖**，只关注 offscreen document 的创建与环境初始化，便于新手和团队快速上手。
- **为 DOMParser、剪贴板、Canvas、音视频处理等后台能力打下基础**，以后可直接在 offscreen 页实现复杂逻辑。
- **符合 Chrome 扩展 MV3 最佳实践**，严格权限、理由说明，便于通过 Chrome Web Store 审核。

---

## 典型应用场景

- 后台剪贴板读写、图片/文件处理
- 离屏 DOM 解析、内容抓取、HTML 转换
- 扩展后台自动化、异步数据处理等

---

## 总结流程图（文字版）

```
[后台脚本]
    ↓
[调用 browser.offscreen.createDocument]
    ↓
[创建并加载 offscreen.html]
    ↓
[offscreen/main.ts 运行（可用 DOM、剪贴板等 API）]
```

---

如需进一步扩展**offscreen document 的高级用法、与后台/内容脚本通信、实际 DOM 处理案例**，欢迎继续提问！
