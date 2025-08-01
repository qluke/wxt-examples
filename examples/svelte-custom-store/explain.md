## 项目核心功能解释：svelte-custom-store

---

### 一句话总结

**本项目演示了如何在 WXT 浏览器扩展中，结合 Svelte 的自定义 store 和 WXT 的存储 API，实现内容脚本、popup、background 等多端间的响应式状态同步与持久化，支持实时订阅与持久存储。**

---

## 主要功能与实现原理

### 1. Svelte store + WXT storage 双向结合

- 实现了一个自定义的 Svelte store（见 `src/lib/store.ts`），其核心是：
  - 用 Svelte 的 `writable` 创建响应式 store。
  - 用 WXT 的 `storage.defineItem` 管理本地（local）持久化，并监听变更。
  - store 的 set 操作会同步更新 WXT storage，storage 的 watch 会同步更新 Svelte store。
- 这样 popup、content script、background 等任意端都可以用 Svelte 方式订阅和操作同一个全局状态，且自动持久化和跨端同步。

### 2. 全端响应式状态订阅和修改

- **background.ts**、**content.ts**、**popup/App.svelte** 等都可以直接 `import { someProperty } from "@/lib/store"` 进行订阅。
- 只要任一端修改 store，所有端都会实时收到变更（通过 WXT 存储 watch 推送），并自动同步 UI 和本地存储。

### 3. 持久化与初始化

- 初始值（如 `true`）通过 `fallback` 配置写入。
- 刷新 popup、重开 service worker、重开网页都能保留最新状态，因为数据实际存在 WXT 的 local storage。

### 4. Svelte 体验与 TypeScript 类型安全

- Svelte 组件内可直接用 `$someProperty` 绑定 UI（如 checkbox 双向绑定）。
- 支持强类型，IDE 自动推断 store 类型和存储 key。

---

## 主要代码结构与说明

### src/lib/store.ts（自定义 Svelte store）

```typescript
function createStore<T>(value: T, storageKey: StorageItemKey) {
  const { subscribe, set } = writable(value);

  const storageItem = storage.defineItem<T>(storageKey, { fallback: value });
  storageItem.getValue().then(set);
  const unwatch = storageItem.watch(set);

  return {
    subscribe,
    set: (value: T) => storageItem.setValue(value),
  };
}
export const someProperty = createStore(true, "local:someProperty");
```

- 将 Svelte store 与 WXT 存储数据双向绑定，跨端变更自动同步。

---

### popup/App.svelte

```svelte
<script lang="ts">
  import { someProperty } from "@/lib/store";
</script>

<label>
  <span>Toggle someProperty</span>
  <input type="checkbox" bind:checked={$someProperty} />
</label>
```

- Svelte 风格的响应式 UI，checkbox 状态与全局 store 实时同步。

---

### entrypoints/background.ts、content.ts

```typescript
import { someProperty } from "@/lib/store";
import { get } from "svelte/store";

export default defineBackground(() => {
  console.log("someProperty", get(someProperty));
  someProperty.subscribe((value) => {
    console.log("someProperty changed", value);
  });
});
```

- 后台/内容脚本同样可以订阅或读取状态，任何端的变更都会被所有端监听到。

---

### wxt.config.ts

```typescript
export default defineConfig({
  srcDir: "src",
  modules: ["@wxt-dev/module-svelte"],
  manifest: {
    permissions: ["storage"],
  },
});
```

- 开启 Svelte 支持，声明 `storage` 权限。

---

## 亮点与实际价值

- **全端响应式状态同步**：popup、content、background 只需一份代码，状态实时同步无副作用。
- **自动本地持久化**：无需手写 localStorage/Chrome Storage 兼容代码，WXT 自动管理。
- **Svelte 语法友好**：在 Svelte 组件中用 `$store` 即可响应 UI，无需额外 glue 代码。
- **支持 TypeScript 强类型推断**，开发体验极佳。
- **可拓展为多 store、多 key、多类型持久化管理**。

---

## 适用场景

- 需要扩展各端间实时同步、持久化的设置、开关、会话状态
- 复杂扩展架构（如 popup/内容脚本/后台均需访问和响应同一份数据）
- 希望用 Svelte store 状态管理风格开发扩展
- 希望极简地实现“storage + 跨端响应式同步”能力

---

## 总结流程图（文字版）

```
[popup/内容脚本/后台]
    ↓
[import { someProperty } from "@/lib/store"]
    ↓
[set/subscribe 触发 WXT storage 更新/订阅]
    ↓
[WXT storage 变更自动广播到所有端，Svelte store 自动更新]
    ↓
[UI/逻辑实时同步，状态持久化]
```

---

如需进一步了解**store 多 key 管理、存储空间切换（local/sync）、大数据对象支持、性能优化**等，欢迎继续提问！
