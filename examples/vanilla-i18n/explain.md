## 项目核心功能解释：vanilla-i18n

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展项目中，利用原生 `browser.i18n` API 和 `_locales` 多语言目录，实现扩展的国际化（i18n）支持，自动根据用户浏览器语言显示本地化 UI 内容。**

---

## 主要功能与实现原理

### 1. 多语言资源文件结构

- 在 `public/_locales/en/messages.json` 下维护所有英文文案，采用标准 Chrome 扩展 `_locales` 目录结构。
- 每个 key 对应一条 message，可包含参数占位符（如 `Hello $1!`）。

### 2. 清单（manifest）本地化

- 在 `wxt.config.ts` 里，`manifest` 字段可用 `"__MSG_extName__"` 这类特殊字符串自动替换为本地化内容。
- Chrome/Firefox 扩展会根据用户浏览器的默认语言自动选择对应 `_locales/{lang}/messages.json`。

### 3. 前端代码国际化调用

- 通过 `browser.i18n.getMessage` 获取本地化字符串，并可传递参数（如用户名）。
- 示例：`browser.i18n.getMessage("helloWorld", "John Smith")` 返回 `"Hello John Smith!"`。
- 支持所有扩展环境（popup、内容脚本、background 等）。

### 4. 自动文案填充

- `document.title = browser.i18n.getMessage("popupTitle")` 自动设置页面标题为本地化内容。
- 通过 DOM 操作设置 UI 元素内容，无需任何第三方库。

### 5. 零依赖、官方兼容

- 仅用原生浏览器 API 和 Chrome 扩展官方推荐目录结构，兼容所有主流浏览器扩展平台。
- 支持多语言切换、参数插值、RTL/LTR、UI 文案和描述的统一管理。

---

## 主要文件结构与说明

- **public/\_locales/en/messages.json**
  - 定义所有英文文案，格式与 Chrome/Firefox 扩展一致。
- **wxt.config.ts**
  - `manifest` 中用 `__MSG_key__` 自动适配本地化。
- **entrypoints/popup/main.ts**
  - 利用 `browser.i18n.getMessage` 获取、插入本地化文案。
- **entrypoints/popup/index.html**
  - 简单页面，所有内容通过 JS 自动本地化填充。

---

## 亮点与实际价值

- **官方标准方案**：兼容所有 Chrome/Firefox/Edge 等主流浏览器扩展国际化要求。
- **开发体验极佳**：多语言文案统一管理，支持参数插值和 UI/描述多处复用。
- **零依赖、体积极小**：无第三方库，所有能力靠浏览器原生支持。
- **扩展易于上架和维护**：多语言支持是许多应用商店的强制审核项。

---

## 适用场景

- 需要多语言/国际市场的浏览器扩展开发
- 希望 UI、描述、弹窗、内容脚本等所有端统一国际化
- 需要支持参数化文案、动态插值的场景

---

## 总结流程图（文字版）

```
[public/_locales/{lang}/messages.json]
     ↓
[manifest:name/description 用 __MSG_key__ 自动本地化]
     ↓
[前端代码用 browser.i18n.getMessage(key, 参数)]
     ↓
[UI、title、描述全自动适配用户浏览器语言]
```

---

如需补充**多语言目录扩展、动态切换语言、内容脚本多语言兼容、复杂参数插值**等内容，欢迎继续提问！
