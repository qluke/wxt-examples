## 项目核心功能解释：dynamic-content-scripts

---

### 一句话总结

**本项目让浏览器扩展的内容脚本支持“动态授权、动态注入”，用户可随时对任意网站开启/关闭内容脚本，无需在扩展安装时请求全网权限（如`*://*/*`），极大提升了权限安全和用户体验。**

---

## 主要功能与实现原理

### 1. 动态内容脚本注入

- 依赖 [webext-dynamic-content-scripts](https://github.com/fregante/webext-dynamic-content-scripts)，可以让扩展在**没有全网 host 权限**的前提下，允许用户临时开启内容脚本到指定域名。
- 用户通过**右键扩展图标 > “Enable dynamic-content-scripts on this domain”**，即可动态授权和注入内容脚本到当前域名。

### 2. 权限最小化+可控

- 扩展**仅在需要时请求 host 权限**，而不是强制在安装时就申请所有网站权限。
- 配合 [webext-permission-toggle](https://github.com/fregante/webext-permission-toggle)，为用户提供简洁的权限开关界面和逻辑。

### 3. 默认行为与动态行为

- 默认内容脚本**只会在 `https://wxt.dev` 加载**（开发环境下配置见 `wxt.config.ts`）。
- **不会自动注入到 `https://google.com`** 上，除非用户通过上下文菜单主动授权该域名。
- 授权后，内容脚本会被注入到对应页面，并在控制台输出 `Content script loaded!`。

### 4. 核心依赖与配置

- `webext-dynamic-content-scripts`：实现内容脚本的动态注册、注入与回收
- `webext-permission-toggle`：为非开发者用户提供直观的“站点权限管理”入口
- 必要权限：`"storage"`, `"scripting"`, `"activeTab"`, `"contextMenus"`
- `optional_host_permissions: ["*://*/*"]`：允许扩展**在运行时请求任意 host 权限**

---

## 主要代码片段解读

### background.ts

```typescript
import "webext-dynamic-content-scripts";
import addPermissionToggle from "webext-permission-toggle";

export default defineBackground(() => {
  addPermissionToggle();
});
```

- 引入并启用权限切换功能，初始化动态内容脚本支持。

---

### content.ts

```typescript
export default defineContentScript({
  matches: ["*://*.wxt.dev/*"],
  main() {
    console.info("Content script loaded!");
  },
});
```

- 仅在 `*.wxt.dev` 默认注入内容脚本，实际动态注入由 `webext-dynamic-content-scripts` 统一管理。

---

### wxt.config.ts 重点

```typescript
export default defineConfig({
  manifest: {
    permissions: ["storage", "scripting", "activeTab", "contextMenus"],
    optional_host_permissions: ["*://*/*"],
  },
  action: {},
  // 开发时手动补内容脚本配置，保证 dev 热重载可用
  hooks: {
    "build:manifestGenerated": (wxt, manifest) => {
      if (wxt.config.command === "serve") {
        manifest.content_scripts ??= [];
        manifest.content_scripts.push({
          matches: ["*://*.wxt.dev/*"],
          js: ["content-scripts/content.js"],
        });
      }
    },
  },
  webExt: {
    startUrls: ["https://wxt.dev", "https://google.com"],
  },
});
```

- 配置 manifest 支持动态 host 权限，开发环境自动补全内容脚本配置，方便调试。

---

## 典型体验流程

1. 安装扩展，默认内容脚本**只在 `wxt.dev` 生效**。
2. 访问 `google.com`，内容脚本**不会自动注入**。
3. 右键扩展图标，选择“Enable dynamic-content-scripts on this domain”。
4. 刷新页面，内容脚本被注入（在控制台输出日志）。
5. 关闭权限后，内容脚本自动移除，**无需重启扩展**。

---

## 亮点与实际价值

- **极简权限模型**：不再需要安装时要求“全网权限”，提升用户信任。
- **一键授权/回收**：用户可轻松为任意站点动态授权内容脚本。
- **支持企业/私有站点**：可轻松扩展到 GitHub Enterprise、Intranet 等域名，详见 [官方文档](https://github.com/fregante/webext-dynamic-content-scripts/blob/main/how-to-add-github-enterprise-support-to-web-extensions.md)。
- **开发体验优秀**：支持 WXT 热重载、调试、类型安全。

---

## 适用场景

- 不想在安装时请求全网权限的扩展（极大减少“吓人”权限弹窗）
- 用户自定义在哪些网站上运行扩展脚本
- 支持 SaaS、企业内网、私有云等多变域名

---

## 总结流程图（文字版）

```
[用户访问任意网站]
    │
    ├─[未授权] → 内容脚本不会注入
    │
    └─[右键授权当前域] → 内容脚本立即注入
          │
          └─[再次关闭权限] → 内容脚本自动移除
```

---

如需补充“企业站点支持”方案、动态内容脚本机制原理，或具体如何集成到你自己的项目中，欢迎继续提问！
