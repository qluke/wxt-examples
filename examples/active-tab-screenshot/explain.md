## 项目核心功能解释：active-tab-screenshot

---

### 一句话简介

**这是一个可以“一键截图当前活动标签页并自动下载图片”的浏览器扩展。**

---

### 工作原理与核心流程

#### 1. 权限声明

- 要求 `activeTab`、`downloads` 权限。
  - `activeTab`：允许扩展在用户点击扩展图标后，获得当前标签页的更多操作权限（比如截图）。
  - `downloads`：允许扩展自动发起图片下载。

#### 2. 主要功能流程

1. **用户点击扩展图标**

   - 扩展监听 action（或 browserAction）图标的点击事件。

2. **截取当前标签页可见区域的截图**

   - 调用 `browser.tabs.captureVisibleTab()` 获取当前活动标签页的截图数据（base64 DataURL 格式的 PNG 图片）。

3. **自动下载截图图片**

   - 生成文件名（如 `Screenshot-2025-08-01T16-16-02.123Z.png`）。
   - 对于 Manifest V3（MV3），直接用 DataURL 作为下载链接。
   - 对于 Manifest V2（MV2），先把 DataURL 转为 Blob，再用 `URL.createObjectURL` 生成下载链接。
   - 调用 `browser.downloads.download()` 触发浏览器下载。

4. **异常处理**
   - 如果在不支持截图的页面（如 chrome:// 页面）或因其它原因失败，会打印错误到控制台。

#### 3. 重要代码片段（伪代码总结）

```typescript
onExtensionIconClicked(tab):
    try:
        dataUrl = captureCurrentTabScreenshot()
        downloadImage(dataUrl)
    catch error:
        logError(error)
```

---

### 主要文件解释

- **`entrypoints/background.ts`**  
  主要逻辑都在这里，包括监听点击、截图、下载、兼容 MV2/MV3、异常处理。

- **`utils/data-urls.ts`**  
  提供 `dataUrltoBlob` 工具函数，把 base64 图片 DataURL 转为二进制 Blob，方便 MV2 下下载。

- **`wxt.config.ts`**  
  配置扩展权限和启动页面（方便开发测试）。

---

### 使用演示流程

1. 安装扩展并进入开发环境页面（如 `https://wxt.dev`）。
2. 点击浏览器右上角扩展图标。
3. 页面不会有任何 UI 变化，但会自动下载一张**当前页面可视区域的截图**到本地。
4. 下载的图片文件名自动带有时间戳，方便区分。

---

### 特性与亮点

- **无需内容脚本**，只用后台脚本即可操作截图和下载。
- **自动适配 MV2/MV3**，兼容新版/旧版扩展机制。
- **零侵入**，不会影响页面内容或样式。
- **极简用户体验**，点击即截图下载，无需弹窗或其它交互。
- **安全受控**，只有用户主动点击扩展图标时才会触发截图，且只对当前活动标签页有效。

---

### 典型用途

- 用户临时保存网页截图，不依赖任何网页内 JS。
- 开发者测试页面渲染效果、记录 bug。
- 任何需要“快速一键截图当前标签页”的场景。

---

### 总结流程图（文字版）

1. 用户点击扩展图标
   ↓
2. 扩展请求当前活动标签页截图
   ↓
3. 得到截图 DataURL
   ↓
4. 自动生成文件名并下载图片

---

如需代码演示、流程图或进一步原理剖析，请随时告知！
