## 项目核心功能解释：web-worker-setup

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展（基于 Vite）中，内容脚本内通过 Vite 原生语法创建和使用 Web Worker，实现耗时/异步任务的多线程处理，并解决与扩展环境的兼容问题。**

---

## 主要功能与实现原理

### 1. Vite 原生 Web Worker 支持

- 通过 Vite 的特殊 import 语法（`import MyWorker from "./worker?worker&inline"`），在内容脚本中一行代码即可创建 Web Worker，无需手动打包或管理 worker 文件。
- Worker 代码支持 TypeScript、自动类型检查，和主项目共用依赖、配置。

### 2. 内容脚本中创建 Worker 实例

- 在内容脚本主文件（`entrypoints/content/index.ts`）中：
  ```typescript
  import MyWorker from "./worker?worker&inline";
  export default defineContentScript({
    matches: ["*://*/*"],
    main() {
      console.log("Creating web worker");
      const worker = new MyWorker();
      console.log("Created!");
    },
  });
  ```
- 这样就能在内容脚本运行时自动创建 worker，worker 代码在独立线程运行。

### 3. Worker 代码结构与兼容性

- Worker 文件（`worker.ts`）支持标准 Web Worker API、TypeScript。
- 兼容 WXT 的内容脚本沙箱：加了一行 `globalThis._content = undefined;` 作为 hack，解决 [WXT issue #942](https://github.com/wxt-dev/wxt/issues/942) 的兼容问题。
- Worker 代码可包含复杂计算、异步任务、消息通信等。

### 4. 支持多端环境和现代语法

- `tsconfig.json` 配置了 `lib: ["DOM", "ESNext", "WebWorker"]`，保证主线程和 worker 都有类型支持。
- Worker 文件可用 `postMessage`/`onmessage` 与主线程通信，未来可扩展为内容脚本与 worker 间的复杂数据交互。

---

## 主要代码结构与说明

### entrypoints/content/index.ts

```typescript
// @ts-expect-error: Query params not typed
import MyWorker from "./worker?worker&inline";

export default defineContentScript({
  matches: ["*://*/*"],
  main() {
    console.log("Creating web worker");
    const worker = new MyWorker();
    console.log("Created!");
  },
});
```

- 重点：Vite 原生 worker 语法，一行代码创建 worker 实例。

---

### entrypoints/content/worker.ts

```typescript
// Work around for: https://github.com/wxt-dev/wxt/issues/942
// @ts-ignore
globalThis._content = undefined;

// Worker code below:
console.log("Hello from worker");
```

- 兼容 hack，防止内容脚本沙箱变量污染 worker。
- 实际 worker 内可实现任何密集型、异步任务。

---

## 亮点与实际价值

- **极简 worker 用法**：无需 webpack loader、文件复制、复杂配置，Vite 直接一行 import 创建 worker，完全 TypeScript 支持。
- **扩展内容脚本专用**：解决了内容脚本环境 worker 创建常见的路径、作用域等兼容问题。
- **多线程能力**：可在扩展内容脚本内安全、高效地执行密集运算、后台任务、网络请求等。
- **易于维护和扩展**：worker 代码和主线程代码结构、类型完全一致，易于团队协作和升级。

---

## 典型适用场景

- 扩展内容脚本中需要执行大计算、异步处理，但不希望阻塞页面主线程
- 复杂数据处理、加密解密、大数据解析、AI、OCR 等
- 实现内容脚本和 worker 之间复杂的消息交互、任务分发
- 保持 TypeScript 类型安全、现代开发体验

---

## 总结流程图（文字版）

```
[内容脚本 index.ts]
    ↓
[import Worker from './worker?worker&inline']
    ↓
[const worker = new Worker()]
    ↓
[worker.ts 独立线程运行, 可 postMessage/onmessage 通信]
```

---

如需扩展**worker 与主线程通信、worker 资源管理（终止/重启）、高级多 worker 协作、与扩展 API 结合**等，欢迎继续提问！
