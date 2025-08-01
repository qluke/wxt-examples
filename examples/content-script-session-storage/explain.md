## 项目核心功能解释：content-script-session-storage

---

### 一句话总结

**本项目演示了如何在浏览器扩展中，利用 session storage 在 background 和 content script 之间安全共享会话级（session）数据。**

---

## 功能场景与目标

- **在后台脚本中设置一个 session storage 的值（扩展会话启动时间）**
- **内容脚本能安全获取该值，并在控制台打印出来**
- 场景典型用途：比如扩展的会话唯一标识、会话级配置参数、跨同一扩展 session 的页面通信等

---

## 关键交互流程

1. **后台脚本初始化 session storage**

   - 启动时将当前时间（`Date.now()`）写入 session storage，key 为 `"session:start-time"`。
   - 配置 session storage 的访问级别，确保内容脚本也能访问该 session 区域（Chrome MV3 必需）。

2. **内容脚本获取并使用 session storage**

   - 内容脚本在所有网页注入（`matches: ["*://*/*"]`）。
   - 执行时获取 `"session:start-time"`，如果值存在则在控制台打印 session 启动时间，否则提示未设置。

3. **数据定义与类型安全**
   - 用 `wxt/utils/storage` 封装 session storage 的 get/set，类型安全。

---

## 主要文件与代码解读

### 1. utils/storage.ts

```typescript
import { storage } from "wxt/utils/storage";

// 定义一个 session storage 的数据项（类型为 number，代表时间戳）
export const sessionStartTime =
  storage.defineItem<number>("session:start-time");
```

---

### 2. entrypoints/background.ts

```typescript
export default defineBackground(() => {
  // 设置 session storage 的值为当前时间戳
  const startTime = Date.now();
  void sessionStartTime.setValue(startTime);
  console.log("Setting session start time:", new Date(startTime).toISOString());

  // 配置 session storage 访问级别，确保内容脚本可以访问
  // (Chrome 特性, 其它浏览器可能不需要)
  // @ts-expect-error: setAccessLevel not typed
  void browser.storage.session.setAccessLevel?.({
    accessLevel: "TRUSTED_AND_UNTRUSTED_CONTEXTS",
  });
});
```

---

### 3. entrypoints/content.ts

```typescript
export default defineContentScript({
  matches: ["*://*/*"],
  async main() {
    const startTime = await sessionStartTime.getValue();
    if (startTime == null) {
      console.log("No start time, reload tab");
    } else {
      console.log("Session start time:", new Date(startTime).toISOString());
    }
  },
});
```

---

### 4. 权限要求

- manifest 需声明 `"storage"` 权限

---

## 典型使用流程

1. 扩展启动时，后台会记录当前 session 的开始时间到 session storage。
2. 每个网页加载内容脚本时，自动从 session storage 取出这个时间戳并打印。
3. 只要扩展会话不关闭，所有 tab 的内容脚本拿到的 session 值都一致。
4. 关闭扩展或浏览器，session storage 会自动清空。

---

## 亮点与意义

- **安全隔离**：session storage 只在本扩展 session 有效，不会持久存储、不跨 session 泄漏。
- **内容脚本可安全读写**：通过配置 accessLevel，内容脚本能访问 session storage，适合在页面和后台之间安全同步临时数据。
- **类型安全**：用 wxt storage 工具，避免 key/value 类型错乱。
- **演示了 Chrome MV3 下内容脚本 session storage 权限用法**。

---

## 适用场景

- 统计扩展 session 生命周期
- 在内容脚本与后台间同步“一次性临时数据”
- 需要 session 级唯一标志、临时 token、会话配置信息等

---

## 总结流程图（文字版）

```
[background] (扩展启动)
    │
    └──> 设置 session storage: session:start-time = now
             │
[content script] (每个页面注入)
    │
    └──> 读取 session:start-time 并打印
```

---

如需详细代码注释、原理剖析或更多实际应用案例，欢迎补充提问！
