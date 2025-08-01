## 项目核心功能解释：wxt-i18n

---

### 一句话总结

**本项目演示如何在 WXT 扩展项目中，使用 [@wxt-dev/i18n](https://github.com/wxt-dev/i18n) 模块和 YAML 语法编写本地化语言包，自动生成类型安全的国际化 API，实现扩展多语言支持和参数插值，开发体验远超原生 browser.i18n。**

---

## 主要功能与实现原理

### 1. 基于 YAML 的现代多语言资源管理

- 所有多语言文案集中存放在 `locales/en.yml`（支持多语言多文件，可扩展为 `locales/zh.yml` 等）。
- YAML 文件结构清晰，支持嵌套、参数插值（如 `hello: Hello $1!`），维护体验远优于 JSON。

### 2. 自动生成类型安全的 i18n API

- 通过 `@wxt-dev/i18n`，WXT 在 `.wxt/i18n/` 目录下自动生成类型定义（如 `structure.d.ts`）及类型安全的 `i18n.t` 函数。
- `i18n.t("popup.hello", ["John Smith"])` 会有类型提示，参数数量、key 拼写错误都会在开发阶段提示，极大提升开发体验和可靠性。

### 3. 替换 manifest 多语言字段

- 在 `wxt.config.ts` 配置中，manifest 字段可直接写为 `"__MSG_extension_name__"` 等，WXT 会自动用当前语言包内容替换，兼容 Chrome/Firefox 国际化标准。

### 4. 前端代码国际化调用

- 通过 `i18n.t(key, substitutions?)` 获取多语言文案，支持参数插值、嵌套 key、类型安全。
- 支持所有扩展端（popup、options、内容脚本、background 等）。

### 5. 零依赖于原生 browser.i18n

- 不再直接依赖 `browser.i18n.getMessage`，而是用更现代、更类型安全的工具链和配置。
- 支持自动导入、热更新、类型推断，开发体验与主流 Web/前端 i18n 方案一致。

---

## 主要文件结构与说明

- **locales/en.yml**
  - 多语言主资源文件，结构清晰、支持嵌套和参数。
- **.wxt/i18n/**
  - 自动生成的类型定义和 i18n API，开发时自动维护。
- **entrypoints/popup/main.ts**
  - 用 `i18n.t("popup.title")` 等方式获取文案，支持参数插值、类型自动提示。
- **wxt.config.ts**
  - 加载 `@wxt-dev/i18n/module`，配置 manifest 多语言占位符与默认语言。
- **tsconfig.json**
  - 支持 `#i18n` 路径别名，代码中可直接 `import { i18n } from "#i18n"`。

---

## 关键代码片段说明

### locales/en.yml

```yaml
extension:
  name: Vanilla I18n Example
  description: Use the @wxt-dev/i18n to localize your extension
popup:
  title: Example Popup Title
  hello: Hello $1!
```

### 使用类型安全的 i18n.t

```typescript
document.title = i18n.t("popup.title");
messageH1.textContent = i18n.t("popup.hello", ["John Smith"]);
```

- 参数数量、类型都自动校验，IDE 智能补全。

---

## 亮点与实际价值

- **类型安全**：所有 key、参数都在开发阶段校验，避免运行时出错。
- **开发体验现代**：YAML 管理多语言，支持嵌套、自动补全、热更新。
- **兼容官方国际化标准**：manifest 字段可直接用 `__MSG_key__`，无需二次维护。
- **方便团队协作**：多语言文案维护、校对、翻译更清晰高效。
- **全端通用**：popup、content script、background、options page 均可用同一 i18n API。

---

## 适用场景

- 需要支持多语言的浏览器扩展开发
- 追求类型安全、现代开发体验、团队协作友好的国际化方案
- 需要参数插值、嵌套、多语言资源统一管理

---

## 总结流程图（文字版）

```
[locales/*.yml] (多语言资源)
   ↓
[WXT + @wxt-dev/i18n 自动生成类型安全 i18n API]
   ↓
[代码用 i18n.t(key, params) 获取文案]
   ↓
[manifest 用 __MSG_key__ 自动替换]
   ↓
[全端一致、多语言支持]
```

---

如需了解**多语言切换、复数支持、复杂嵌套、团队翻译流程、内容脚本国际化最佳实践**，欢迎继续提问！
