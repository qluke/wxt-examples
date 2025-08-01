## 项目核心功能解释：vitest-unit-testing

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展项目中，使用 [Vitest](https://vitest.dev/) 进行单元测试，并通过 WXT 提供的测试工具 (`WxtVitest`、`fakeBrowser`) 实现浏览器扩展 API 的模拟和隔离测试，保障扩展逻辑的可靠性和开发体验。**

---

## 主要功能与实现原理

### 1. Vitest 集成与配置

- 使用 Vitest 作为测试运行器，支持 TypeScript、现代断言、快照、模拟等全部主流特性。
- 在 `vitest.config.ts` 里集成 `WxtVitest()` 插件，自动模拟 WebExtension 环境（如 `browser`、`storage` 等 API），无需手动 polyfill 或 mock。
- 支持 `mockReset`、`restoreMocks` 等测试隔离选项，保证每个测试用例环境干净。

### 2. 浏览器扩展 API 的自动 mock

- 通过 `fakeBrowser`（WXT 测试工具），可以模拟 `browser.runtime`、`browser.storage` 等扩展 API 的事件和数据。
- 支持事件触发、断言、状态重置等全部常用测试操作。

### 3. 真实业务逻辑的单元测试

- 以 `entrypoints/background.ts` 为例：
  - 监听扩展安装事件（`onInstalled`），仅在 reason 为 `"install"` 时写入安装日期到本地存储。
- 在测试中可用 `vi.setSystemTime` 模拟时间，`fakeBrowser.runtime.onInstalled.trigger` 模拟扩展事件，断言存储的值是否如预期。

### 4. 遵循最佳工程实践

- 单元测试文件放在 `entrypoints/__tests__/` 下，命名规范、易于发现。
- 每个测试前重置 fakeBrowser 保证无状态污染。
- 可用 `it.each` 测试多参数分支，断言不同安装场景下的行为。

---

## 主要代码结构与说明

### vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import { WxtVitest } from "wxt/testing";

export default defineConfig({
  test: {
    mockReset: true,
    restoreMocks: true,
  },
  plugins: [WxtVitest()],
});
```

- 关键：`plugins: [WxtVitest()]` 自动注入所有 WebExtension mock，无需手动处理。

---

### entrypoints/**tests**/background.test.ts

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import background from "../background";
import { fakeBrowser } from "wxt/testing";

describe("Background Entrypoint", () => {
  beforeEach(() => {
    fakeBrowser.reset();
  });

  it("should store the current date on install", async () => {
    const expected = "2023-12-22T15:27:25.950Z";
    vi.setSystemTime(expected);

    background.main();
    await fakeBrowser.runtime.onInstalled.trigger({
      reason: "install",
      temporary: false,
    });
    const actual = await storage.getItem("local:installDate");

    expect(actual).toBe(expected);
  });

  it.each<Runtime.OnInstalledReason>(["update", "browser_update"])(
    "should not store the current date on %s",
    async (reason) => {
      // ...省略，其逻辑为：模拟非首次安装时 storage 不变
    }
  );
});
```

- 重点：**模拟事件、断言存储、分支覆盖**，开发体验与主流前端工程完全一致。

---

### entrypoints/background.ts

```typescript
export default defineBackground(() => {
  browser.runtime.onInstalled.addListener(({ reason }) => {
    if (reason === "install") {
      storage.setItem("local:installDate", new Date().toISOString());
    }
  });
});
```

- 业务逻辑与真实扩展完全一致，测试用例可直接覆盖。

---

## 亮点与实际价值

- **极速测试反馈**：Vitest 热重载、秒级运行、内存隔离，极大提升开发效率。
- **零手动 mock**：WXT 提供的 `WxtVitest` 与 `fakeBrowser` 全自动模拟 WebExtension 环境。
- **业务、测试一体化**：扩展逻辑与测试代码共用同一套 API，无需“外部化”或分离测试逻辑。
- **适合 CI/CD 自动化**：所有测试可集成到 CI 流程，保证每次构建都能被验证。

---

## 适用场景

- 扩展业务核心逻辑（如事件响应、存储、配置、选项等）的单元测试
- 多分支、异常、边界场景的覆盖验证
- 回归测试、重构验证，提升项目可维护性和健壮性

---

## 总结流程图（文字版）

```
[业务代码 (background.ts)]
   ↓
[Vitest + WxtVitest 自动注入 WebExtension mock]
   ↓
[测试文件用 fakeBrowser 触发事件、断言行为]
   ↓
[测试通过反馈，业务可安全演进]
```

---

如需进一步了解**mock 复杂 API、内容脚本测试、端到端与单元测试集成、CI/CD 自动化**等内容，欢迎继续提问！
