## 项目核心功能解释：basic-messaging

---

### 一句话总结

**这是一个演示浏览器扩展“一次性消息（sendMessage）”和“长连接消息（connect/port）”通信机制的示例项目。**

---

## 主要功能说明

### 1. 一次性消息（One-time Message）

- **popup 页面** 有两个按钮：
  - “Send 'Hello' Message”：发送 `{type: "hello", name: "Aaron"}` 给后台，并显示收到的响应。
  - “Send Unknown Message”：发送 `{type: "unknown"}`，后台未处理，显示无响应或 undefined。
- **后台脚本** 监听所有一次性消息（`onMessage.addListener`），如果 `type` 是 `"hello"`，就返回问候语，否则不响应。

**效果：**

- 用户点按钮后，能看到后台返回的问候语，或看到无响应。

---

### 2. 长连接消息（Long-lived Port）

- **popup 页面** 在加载时调用 `browser.runtime.connect()`，与后台建立一个长连接（port）。
- **后台脚本** 记录所有连入的 port，每隔 1 秒用 `postMessage` 广播一条消息（包括当前时间戳和随机数）。
- **popup 页面** 监听 port 的 `onMessage`，每来一条消息就在页面上新增一行展示。

**效果：**

- 打开弹窗时，页面下方会不断收到后台发来的消息（每秒一次），并动态展示出来。

---

## 主要文件与代码片段说明

### popup/main.ts（前端弹窗逻辑）

```typescript
// 一次性消息发送
sendHelloMessageBtn.onclick = async () => {
  const response = await browser.runtime.sendMessage({
    type: "hello",
    name: "Aaron",
  });
  helloResponsePre.textContent = JSON.stringify(response) || "(No response)";
};

sendUnknownMessageBtn.onclick = async () => {
  const response = await browser.runtime.sendMessage({ type: "unknown" });
  unknownResponsePre.textContent = JSON.stringify(response) || "(No response)";
};

// 长连接监听
const port = browser.runtime.connect();
port.onMessage.addListener((message) => {
  // 每秒收到后台来的消息
  const li = document.createElement("li");
  li.textContent = JSON.stringify(message);
  longLivedMessageList.append(li);
});
```

---

### background.ts（后台逻辑）

```typescript
// 1. 一次性消息监听
browser.runtime.onMessage.addListener((message, _, sendResponse) => {
  if (message.type === "hello") {
    sendResponse(`Hello ${message.name}, this is the background!`);
    return true;
  }
});

// 2. 长连接port管理与定时广播
let ports: Browser.runtime.Port[] = [];
setInterval(() => {
  const message = { date: Date.now(), value: Math.random() };
  ports.forEach((port) => port.postMessage(message));
}, 1000);

browser.runtime.onConnect.addListener((port) => {
  ports.push(port);
  port.onDisconnect.addListener(() => {
    ports.splice(ports.indexOf(port), 1);
  });
});
```

---

## 交互流程总结

### 一次性消息（sendMessage）

1. popup 用户点按钮 -> 调用 `sendMessage`
2. background 的 `onMessage` 收到并处理，返回响应
3. popup 展示响应内容

### 长连接消息（connect/Port）

1. popup 一打开就 `connect` 到后台
2. background 记录 port，每隔一秒 `postMessage` 一条消息
3. popup 监听 port，收到消息就展示

---

## 适用场景

- **一次性消息**：用于请求一次性数据、指令（如 “请后台获取xxx”）
- **长连接**：用于后台主动推送数据、批量同步、实时通知（如后台定时任务、socket桥接等）

---

## 亮点与学习价值

- 覆盖了**所有现代浏览器扩展的消息通信模式**
- 一次性消息和长连接消息**并存、分工明确**
- 演示了前端如何用最简单代码消费后台消息

---

## 总结流程图（文字版）

```
[popup] --(sendMessage)--> [background]
    ^                           |
    |----(响应/数据)-----------|

[popup] <===(port.postMessage 每秒)== [background]
```

---

如需补充**完整代码注释版、流程图或交互演示原理**，请随时提出！
