# WPIM Web 即时通讯系统 - 网页桌面版手册

版本: 4.1.16  
生成日期: 2025年  
适用对象: 最终用户、前端开发人员、系统集成商

---

## 1. 产品概述
信贸通网页桌面版本是一个基于 WebSocket 与 WebRTC 技术的现代 Web 即时通讯解决方案。采用 SDK + UI Adapter 的分离架构，能够根据用户设备自动加载适配界面（DesktopAdapter），为桌面浏览器提供多窗口（Window-based）交互，提供一致且流畅的通讯体验。

### 核心亮点
- 多端自适应（同一套后端 / SDK，桌面使用 DesktopAdapter）
- 实时消息：基于 WebSocket 的低延迟传输
- 音视频通话：集成 WebRTC，支持 1v1 语音/视频、屏幕共享（部分）
- 丰富消息类型：文本、表情、图片（压缩/预览）、文件、自定义消息
- 国际化 (I18n) 与深色模式支持

---

## 2. 功能特性详解

### 2.1 基础通讯功能
- 私聊 / 群聊：支持一对一和群组聊天
- 消息漫游：登录后自动拉取历史消息
- 多媒体发送：
  - 图片：支持拖拽、粘贴、点击选择；内置压缩与查看器（旋转/缩放/下载）
  - 文件：支持大文件上传与进度显示
  - 表情：内置 Emoji 面板，支持最近使用记录
  - 截图（桌面端）：集成截图工具，支持隐藏窗口截图与涂鸦编辑
- 消息交互：撤回、删除（本地）、转发、引用/回复（代码中预留）、右键菜单（复制/转发/删除）

### 2.2 音视频通话 (VoIP)
- 完整呼叫流程：呼叫、接听、拒绝、忙线、取消、挂断
- 桌面通话界面：悬浮独立窗口、最小化/全屏、画面切换、拖拽小窗、静音、关闭摄像头、声波纹动画
- 技术：基于 WebRTC，支持 STUN/TURN 与 ICE 候选穿透

### 2.3 联系人管理
- 好友分组、搜索、添加（带验证）、好友请求处理
- 群组列表与群资料查看
- 黑名单/隐私设置（UI 有入口，具体逻辑依赖服务端）

### 2.4 个性化与设置
- 深/浅色主题、五级字体大小、语言（简体中文/English）
- 通知（桌面通知/声音/标题闪烁/应用内气泡）
- 桌面壁纸（纯色或图片）
- 个人资料：头像上传裁剪、昵称/签名修改

---

## 3. 桌面端用户指南 (Desktop UI)

桌面端采用多窗口 (Window-based) 交互，模拟原生桌面应用体验。

### 3.1 主界面
- 侧边栏：头像（可点击修改）、会话列表、通讯录、设置入口
- 列表区：
  - 会话列表：显示最近联系人/群，支持置顶、右键删除
  - 通讯录：好友/群分组展示，含“新的朋友”入口处理请求
- 搜索栏：顶部支持搜索本地联系人或聊天记录
- 聊天区：主要显示区，未选中会话时显示空状态或帮助信息

### 3.2 聊天操作
- 发送：输入框 Enter 发送（可选 Ctrl+Enter）
- 工具栏：表情、图片/文件、截图、通话按钮（仅私聊显示）
- 截图功能：点击或快捷键触发，支持编辑（画笔/箭头/马赛克）
- 独立窗口（Pop-out）：双击会话可弹出独立聊天窗口，便于多任务处理

### 3.3 系统功能
- 添加好友：点击 + 号输入账号并发送验证请求
- 修改头像：点击左上角头像弹出裁剪窗口
- 设置：齿轮图标配置声音、通知、隐私、语言、字体等

---

## 4. 技术架构与集成指南 (For Developers)

### 4.1 目录与命名空间
- 全局命名空间：`window.ImCore`（核心）与 `window.ImUI`（UI）
- 主要模块：
  - ImApplication：应用入口
  - ImCore：SDK 控制、Core（WebSocket）、Protocol（编解码）、VoipManager（WebRTC 封装）
  - ImUI：DesktopAdapter（桌面控制器）、*Window/*Screen 视图组件

### 4.2 初始化示例
在 HTML 中引入 im.all.js 及依赖（jQuery、font-awesome、fabric.js 等）后：

```javascript
const config = {
    socketUrl: 'ws://your-server:port/ws',
    apiServer: 'http://your-server:port',
    fileUploadUrl: 'http://your-server/upload',
    avatarBaseUrl: 'http://your-server/avatars/',
    language: 'zh',
    transformImageUrl: (url) => url.replace('http:', 'https:')
};

const app = new ImApplication();
app.start(config);
```

> 注意：以上为示例配置，请勿直接在生产环境使用示例凭证或真实地址。

### 4.3 事件监听（EventEmitter）
常用事件与说明：
- `login_success` — 登录成功 — User Object
- `message` — 新消息 — Message Model（content, from, to, mediaType）
- `sessions_update` — 会话更新 — Array of Sessions
- `contacts_update` — 联系人更新 — Array of Groups
- `incoming_call` — 收到来电 — `{ targetId, callType }`
- `kicked_out` — 被踢下线 — `{ Message }`

### 4.4 协议格式
使用自定义文本协议：`Command SubCommand {JSON_Data}`  
示例：`.Login { "Uin": 123, "Password": "..." }`  
解析入口：`ImCore.Protocol.decode(string)`

### 4.5 文件 / 图片上传
- 内置方法：`uploadFile`、`uploadImage`
- 推荐通过 HTTP POST 上传；兼容旧版 WebSocket Base64 发送
- UploadKeyGenerator（MD5: Uin + FileNameLen + Salt）用于校验 Key（不要在公开文档中放真实 Salt）

### 4.6 WebRTC 流程简述
- VoipManager 负责信令；WebRTCClient 负责流媒体
- 呼叫流程：Request -> incoming_call -> Accept -> 初始化 PeerConnection -> 交换 SDP/Candidate -> 建立 P2P -> `stream_added` -> 渲染 `<video>`

---

## 5. 配置与常见问题

### 5.1 本地存储键
- `im_language`, `im_dark_mode`, `im_font_size`, `im_cfg_sound`, `im_cfg_popup`


### 5.2 常见问题
- 截图不可用：检查 html-to-image/html2canvas，跨域图像会导致问题
- 视频黑屏：确保在 HTTPS 环境下运行并检查 STUN/TURN 配置
- 头像不显示：确认 `config.avatarBaseUrl` 与后端 CORS 设置

## 6. 问题沟通
- 请访问：https://www.semot.com

  
(手册结束)
