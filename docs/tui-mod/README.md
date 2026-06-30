# OpenCode TUI 魔改增强包

对 [opencode](https://github.com/cymylive/opencode) TUI 界面的美化与功能增强。直接在源代码上 patch，编译后即可生效。

## 功能一览

### 1. 侧边栏增强（sidebar.tsx）

- **History 列表** — 显示同目录最近 20 条会话，按更新时间排序，过滤子对话
- **当前会话高亮** — ▶ 标记 + 背景色高亮
- **删除按钮** — 扩大热区，点击一次确认（3 秒超时），再点执行删除
- **Model 选择** — 点击打开 Model 切换面板
- **+New 按钮** — 快速跳转首页新建会话

### 2. ☰ 侧边栏触发按钮（session-index.tsx）

- 右下角浮动 `☰` 按钮，点击展开侧边栏
- 窄屏模式：侧边栏覆盖显示，点击遮罩层关闭

### 3. 消息气泡美化（session-index.tsx）

- **User 消息** — 顶部显示 `You` 标签
- **Assistant 消息** — 显示模型名 + 响应耗时（秒）

### 4. 底部状态栏（footer.tsx）

- 顶部分隔线
- LSP 状态 — `●` 绿色圆点
- MCP 状态 — `●` 青色圆点
- Permission 状态 — `▲` 警告色
- 未连接时显示 `/connect` 提示

### 5. 对话框美化（dialog.tsx）

- 顶部主色光条装饰

### 6. 首页美化（home.tsx）

- 去除大 Logo，改用简洁文字
- Prompt 输入框添加主色上下边框

### 7. 启动加载页（startup-loading.tsx）

- Spinner 颜色跟随主题主色
- 左右边框装饰

### 8. 品牌文字（logo.ts / logo.tsx）

- 左侧 `OPEN` + 右侧 `CODE` 的 ASCII art
- 可取消粗体

### 9. 主题系统 — 5 款自定义主题

| 主题 | 背景 | 主色 | 风格 |
|------|------|------|------|
| `livecode` | `#c7edcc` | `#5b8c5b` | 豆沙绿护眼（浅色） |
| `livecode2` | transparent | `#00d4ff` | 深色霓虹磨砂（配合 Acrylic） |
| `livecode3` | `#f5efe8` | `#e8a87c` | 暖沙浅色 |
| `livecode4` | `#e8f4f8` | `#55bbee` | 晴空蓝浅色 |
| `livecode5` | `#f2ecf8` | `#a888dd` | 淡紫雾浅色 |

切换方式：`ctrl+p` → `/themes` → 选择主题

## Patch 文件清单

```
patches/
├── app.tsx               # Spinner 注册
├── session-index.tsx     # ☰按钮 + 遮罩关闭 + position:relative + Spinner
├── sidebar.tsx           # 增强侧边栏（History/Model/删除/高亮）
├── footer.tsx            # 底部状态栏彩色圆点
├── home.tsx              # 首页去除 Logo + 输入框边框
├── dialog.tsx            # 弹窗顶部光条
├── startup-loading.tsx   # 加载页美化
├── theme-index.ts        # 主题注册（5 个 livecode 主题）
├── logo.tsx              # 取消粗体
├── logo.ts               # 原始 Logo（已恢复 OPENCODE）
├── livecode.json         # 主题：豆沙绿护眼
├── livecode2.json        # 主题：深色磨砂霓虹
├── livecode3.json        # 主题：暖沙浅色
├── livecode4.json        # 主题：晴空蓝浅色
└── livecode5.json        # 主题：淡紫雾浅色
```

## 使用方法

### 恢复修改

```powershell
.\restore.ps1 -SourceDir "你的opencode源码目录"
```

### 编译 exe

```powershell
$env:OPENCODE_CHANNEL="release"
$env:OPENCODE_VERSION="1.17.9"
bun run script/build.ts --single --baseline --skip-install --skip-embed-web-ui
```

输出位置：`packages\opencode\dist\opencode-windows-x64-baseline\bin\opencode.exe`

## 兼容性

基于 opencode **v1.17.9** 开发，更高版本可能需要调整 patch 文件。
