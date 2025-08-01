## 项目核心功能解释：svelte-state-rune

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展中，基于 Svelte 5 的“state rune”响应式语法，用 WXT 的 storage API 实现扩展各端（内容脚本、popup、后台）和网页之间的事件驱动状态同步与持久化，打造现代响应式状态管理。**

---

## 主要功能与实现原理

### 1. Svelte 5 State Rune + WXT Storage 的响应式集成

- 使用 Svelte 5 的 `$state` 响应式语法（state rune），让类实例的属性和 UI 自动同步。
- 用 WXT 的 `storage.defineItem` 创建 session storage（`session:counter`），页面、内容脚本、popup、后台都能读写和订阅该状态。

### 2. 跨端事件驱动状态同步

- **index.html** 网页每秒触发一次 `counter:updated` 事件，事件 detail 为 `{ counter: ... }`。
- **内容脚本** 监听该事件，取出 detail 并写入 WXT session storage（`counterStore.setValue(event.detail)`）。
- **popup** 通过 `counterState` 实例，自动订阅 storage，实时反映最新 counter 值。
- **后台** 可做同样的订阅，所有端均可获知和响应状态变化。

### 3. 响应式 State 类的实现

- `lib/counter-state.svelte.ts` 定义了一个 CounterState 类，用 `$state` 修饰属性，只要 storage 变更或重新获取，state 就会自动同步到 UI。
- `counterStore.watch(this.updateCounter)` 实现了全局事件监听，只要状态改变，所有订阅实例会立即同步。

### 4. 状态持久化与初始化

- `counterStore` 提供 fallback 初始值 `{ counter: 0 }`，即使 popup 关闭后重新打开，也能还原上次状态。
- 使用 session storage，支持 session 级数据隔离，也可扩展为 local/sync 等持久化模式。

### 5. 类型安全与现代 Svelte 生态

- TypeScript 类型严格，事件类型、状态对象全自动推断。
- 组件、状态、事件都用最新 Svelte 5 语法和最佳工程实践。

---

## 主要代码结构与说明

### src/lib/counter-store.ts

```typescript
export const counterStore = storage.defineItem<{ counter: number }>(
  "session:counter",
  { fallback: { counter: 0 } }
);
```

- 定义响应式、持久化的计数器 session storage。

---

### src/lib/counter-state.svelte.ts

```typescript
class CounterState {
  state = $state(counterStore.fallback);
  constructor() {
    counterStore.getValue().then(this.updateCounter);
    counterStore.watch(this.updateCounter);
  }
  updateCounter = (newState: { counter: number } | null) => {
    this.state = newState ?? counterStore.fallback;
  };
}
export const counterState = new CounterState();
```

- 用类封装响应式状态，UI 只需用 `counterState.state.counter` 即可自动响应所有变更。

---

### src/entrypoints/content.ts

```typescript
import { counterStore } from "@/lib/counter-store";
export default defineContentScript({
  matches: ["http://localhost/*"],
  main() {
    document.addEventListener("counter:updated", (event) => {
      void counterStore.setValue(event.detail);
    });
  },
});
```

- 内容脚本监听网页事件，将数据写入扩展状态。

---

### src/entrypoints/popup/App.svelte

```svelte
<main class="app">
  <div>
    <a href="https://wxt.dev" ...><img ... /></a>
    <a href="https://svelte.dev" ...><img ... /></a>
  </div>
  <h1>WXT + Svelte</h1>
  <div class="card">
    <p>{counterState.state.counter}</p>
  </div>
  <p class="read-the-docs">...</p>
</main>
```

- UI 直接用 `counterState.state.counter`，自动响应所有更新。

---

### index.html

```html
<script>
  let counter = 0;
  setInterval(() => {
    document.dispatchEvent(
      new CustomEvent("counter:updated", { detail: { counter: counter++ } })
    );
  }, 1000);
</script>
```

- 网页端每秒触发一次事件，驱动全局计数器递增。

---

## 亮点与实际价值

- **Svelte 5 state rune 响应式体验**：不需要手写订阅/更新逻辑，状态和 UI 静默同步。
- **跨端（popup/background/content）+ 跨上下文（网页）实时同步**，场景丰富。
- **事件驱动 + 持久化**：浏览器网页事件直接驱动扩展持久状态，刷新、重开都能还原。
- **代码组织现代、类型安全、工程可维护性强**。

---

## 适用场景

- 需要扩展多端共享、持久化的复杂状态（如用户设置、计数器、表单等）
- 希望用 Svelte 5/TypeScript 最佳实践开发扩展
- 需要监听网页事件/数据，并同步到扩展各端

---

## 总结流程图（文字版）

```
[网页 index.html] --(dispatchEvent)--> [内容脚本监听，写 storage]
         ↓                                             ↓
  [popup/App.svelte] <---(storage.watch)-- [counterStore] --(storage.watch)-- [background/其它端]
         ↑                                             ↑
 [counterState.state 自动同步，无需手动订阅/更新]
```

---

如需进一步了解**多 store 管理、类型扩展、local/sync storage 切换、性能优化**等，欢迎随时提问！
