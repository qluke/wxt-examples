## 项目核心功能解释：storybook

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展开发中集成 Storybook，做到组件开发、测试、文档和可视化预览与扩展工程深度结合，实现高效的组件开发体验和可维护性。**

---

## 主要功能与实现原理

### 1. Storybook 与 WXT/React 无缝集成

- 使用 Storybook（基于 Vite 的 builder）对扩展项目内所有 React 组件（`components/`、`stories/`）进行可视化开发和交互测试。
- 通过 `.storybook/vite.config.ts` 复用 WXT 的 Vite 配置（`WxtVitest()`），保证 Storybook 环境和扩展打包环境一致。

### 2. 组件开发与文档自动化

- 组件（如 `Button.tsx`）和其 Story 文件（如 `test-Button.stories.ts`）可在 Storybook UI 中实时预览、交互、切换变体（Primary、Secondary、Large、Small 等）。
- 支持自动文档（Autodocs）、参数控制、交互事件（actions）、布局切换等 Storybook 强大特性。
- 支持 Chromatic、Onboarding、Links、Essentials 等 Storybook 插件，提升设计、协作和测试效率。

### 3. 与扩展主应用代码完全共用

- 组件代码和故事文件直接复用于 popup、内容脚本、主应用等，不存在代码分叉问题。
- 通过自动化测试（例如可集成 Playwright、Vitest 等），实现从组件单测到 UI 可视化集成测试全流程覆盖。

### 4. 现代前端工程体验

- 支持 React 18+、TypeScript、Vite、HMR，开发效率极高。
- 组件样式可用 CSS、Tailwind 或其他方式，Storybook 能完整还原实际效果。
- 一键切换 Storybook、扩展开发、打包、测试环境。

---

## 主要文件结构与说明

- `.storybook/`：Storybook 配置（main.ts、vite.config.ts、preview.ts）
- `components/`：可复用的 React 组件和样式
- `stories/`：自定义或第三方故事文件、MDX 文档
- `entrypoints/`：扩展内容脚本、popup 等主入口
- `wxt.config.ts`：WXT 配置，保证扩展和 Storybook 共用依赖和打包环境

---

## 关键代码与配置说明

### .storybook/main.ts

```typescript
const config: StorybookConfig = {
  stories: [
    "../{stories,components}/**/*.mdx",
    "../{stories,components}/**/*.stories.@(js|jsx|mjs|ts|tsx)",
  ],
  addons: [
    "@storybook/addon-onboarding",
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@chromatic-com/storybook",
    "@storybook/addon-interactions",
  ],
  framework: {
    name: "@storybook/react-vite",
    options: {
      builder: {
        viteConfigPath: ".storybook/vite.config.ts",
      },
    },
  },
};
```

- **重点**：`viteConfigPath` 指向自定义 `.storybook/vite.config.ts`，确保与 WXT 项目配置一致。

---

### .storybook/vite.config.ts

```typescript
import { defineConfig } from "vite";
import { WxtVitest } from "wxt/testing";

export default defineConfig({
  plugins: [
    // 复用 WXT 的 Vite 配置
    WxtVitest(),
  ],
});
```

- Storybook 会自动引入扩展配置，包括 alias、自动导入、模块等。

---

### 组件与 Story 示例

- `components/Button.tsx`：主按钮组件，带多种变体和可配置属性
- `components/test-Button.stories.ts`：Story 文件，定义了不同变体（Primary、Secondary、Large、Small）和交互演示

---

### 运行方式

```sh
pnpm i
pnpm storybook
```

- 打开 http://localhost:6006 即可进入 Storybook UI，实时开发、文档和交互测试。

---

## 亮点与实际价值

- **极致组件开发体验**：可视化、交互、热重载、参数控制、文档自动生成
- **一套代码多端共用**：真实扩展、Storybook、测试、文档完全共用组件代码
- **团队协作友好**：设计、开发、测试、产品可在 Storybook UI 直接讨论和预览
- **与现代工程体系无缝结合**：React、Vite、TypeScript、WXT、自动化测试

---

## 适用场景

- 扩展 UI 组件库开发、复用与维护
- 设计系统/样式统一、文档自动化
- 复杂交互/变体/状态的可视化调试和测试

---

## 总结流程图（文字版）

```
[组件/故事文件]
   ↓
[Storybook (Vite builder) + WXT Vite config]
   ↓
[本地/云端可视化开发、交互、文档、测试]
   ↓
[与扩展应用主代码完全复用]
```

---

如需补充**自定义 Storybook 主题、MDX 文档、与自动化测试集成、团队协作最佳实践**，欢迎继续提问！
