## 项目核心功能解释：vue-storage-composable

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展中，结合 Vue 3、VueUse 和 WXT 的 storage API，封装出“响应式、自动同步、持久化”的 storage composable，实现扩展各端（popup、background、内容脚本）之间的数据共享与实时 UI 响应。**

---

## 主要功能与实现原理

### 1. 响应式 Storage Composable 封装

- **核心：`useStoredValue` 组合式函数（composable）**
  - 使用 VueUse 的 `useAsyncState` 实现异步初始化和 isLoading 状态。
  - 挂载时自动从 storage 读取初始值并监听变更（`storage.watch`），storage 变化时自动同步到 Vue 的响应式 `state`。
  - 提供一个 writable 的 computed ref，支持 UI 直接读写，写入时自动同步 storage。
  - 卸载时自动取消监听，避免内存泄漏。

### 2. 实时多端数据同步

- **background.ts** 定时每秒自增 `session:count`，写入扩展 storage。
- **popup/App.vue** 通过 `useStoredValue("session:count", 0)` 响应式获取并显示最新计数，同时可以点击按钮自增。
- **所有端**（popup、background、内容脚本等）都能实时订阅同一 storage key，任何端的变更都会同步到所有 UI，无需手写事件通信。

### 3. 现代 Vue 3 生态与最佳实践

- 支持 SFC（单文件组件）、TypeScript 强类型、自动类型推断。
- 使用 VueUse 生态（`useAsyncState`）提升开发体验和代码简洁性。
- 组合式 API，易于复用和扩展到更多 storage key、业务场景。

### 4. 持久化与状态管理

- 可切换 `local`、`sync`、`session` 等 storage 区域，支持扩展不同持久化需求。
- 在 popup 关闭/重开时，storage 保留最新状态，UI 自动还原。

---

## 主要代码结构与说明

### composables/useStoredValue.ts

```typescript
import { UseAsyncStateOptions, useAsyncState } from "@vueuse/core";
import { computed, onMounted, onUnmounted } from "vue";
import { storage, StorageItemKey } from "wxt/utils/storage";

export default function <T>(
  key: StorageItemKey,
  initialValue?: T,
  opts?: UseAsyncStateOptions<true, T | null>
) {
  const { state, ...asyncState } = useAsyncState<T | null, [], true>(
    () => storage.getItem(key),
    initialValue ?? null,
    opts
  );

  let unwatch: (() => void) | undefined;
  onMounted(() => {
    unwatch = storage.watch<T>(key, async (newValue) => {
      state.value = newValue ?? initialValue ?? null;
    });
  });
  onUnmounted(() => {
    unwatch?.();
  });

  return {
    state: computed({
      get() {
        return state.value;
      },
      set(newValue) {
        void storage.setItem(key, newValue);
        state.value = newValue;
      },
    }),
    ...asyncState,
  };
}
```

- 重点：**storage 变化自动驱动响应式 Vue 状态，UI 与存储自然同步**。

---

### popup/App.vue

```vue
<script lang="ts" setup>
import useStoredValue from "@/composables/useStoredValue";
const { state: count, isLoading } = useStoredValue<number>("session:count", 0);
</script>

<template>
  <div>
    <p>Count: {{ isLoading ? "Loading..." : count }}</p>
    <button @click="count++">+1</button>
  </div>
</template>
```

- 重点：UI 直接用响应式变量 `count`，可读可写，和 storage 保持同步，自动响应 background 变更。

---

### entrypoints/background.ts

```typescript
export default defineBackground(() => {
  setInterval(async () => {
    const oldValue = await storage.getItem<number>("session:count");
    const newValue = (oldValue ?? 0) + 1;
    await storage.setItem("session:count", newValue);
  }, 1000);
});
```

- 重点：后台每秒自增 `session:count`，所有端 UI 会自动同步最新值。

---

## 亮点与实际价值

- **全端实时响应式状态同步**：只需一行 Vue 代码，自动订阅 storage，UI/业务逻辑全同步。
- **极简但强大**：无需手写事件桥、消息通信、localStorage 兼容代码。
- **支持所有扩展端**：popup、background、内容脚本、options page 均可用同一 storage composable。
- **类型安全、易于扩展**：支持任意 key、泛型、异步初始值。

---

## 适用场景

- 需要扩展多端共享、持久化的复杂状态（如用户设置、计数器、列表等）
- 希望用 Vue 组合式 API 实现响应式 storage 管理
- 希望极简地实现“storage + 跨端响应式同步”能力
- 多端业务逻辑、UI 需要同步响应 storage 变化

---

## 总结流程图（文字版）

```
[background/popup/content script/任何端]
    ↓
[useStoredValue("session:count")]
    ↓
[storage.setItem/watch 自动同步到所有端]
    ↓
[所有 UI/逻辑自动响应，数据持久化]
```

---

如需进一步了解**多 storage key 管理、复杂对象结构、性能优化、与 Vuex/Pinia 集成**等，欢迎继续提问！
