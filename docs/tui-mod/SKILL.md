# OpenCode TUI 魔改 Skill

当用户要求对 opencode TUI 进行 UI 美化、主题定制、侧边栏改造、编译 exe 时使用此 skill。

## 快速恢复所有修改

```powershell
.\restore.ps1 -SourceDir "I:\opencode-1.17.9"
```

这会从 `patches/` 目录将所有修改过的文件覆盖回源码目录。

## 文件清单

```
SKILL.md              ← 本文件（工作流文档）
restore.ps1           ← 恢复脚本（一键还原所有修改）
patches/
├── app.tsx           ← Spinner 注册
├── session-index.tsx ← ☰按钮 + 遮罩关闭 + position:relative + Spinner
├── sidebar.tsx       ← 侧边栏 History/Model/删除/高亮
├── footer.tsx        ← 底部状态栏彩色圆点
├── home.tsx          ← 首页去除 Logo
├── dialog.tsx        ← 弹窗顶部光条
├── startup-loading.tsx ← 加载页美化
├── theme-index.ts    ← 主题注册（5个livecode主题）
├── logo.tsx          ← 取消粗体
├── logo.ts           ← 原始Logo（已恢复OPENCODE）
├── livecode.json     ← 主题：豆沙绿护眼
├── livecode2.json    ← 主题：深色磨砂霓虹
├── livecode3.json    ← 主题：暖沙浅色
├── livecode4.json    ← 主题：晴空蓝浅色
└── livecode5.json    ← 主题：淡紫雾浅色
```

## 目录结构

源码根目录：`I:\opencode-1.17.9`
用户数据目录：`I:\opencode-1.17.9\mydata\`（壁纸等）
Bun：`I:\bun-windows-x64-baseline\bun.exe`

## 硬约束

1. **不要硬编码绝对路径** — 所有工具脚本中的路径必须基于 `%~dp0` 或相对路径
2. **新增文件夹必须用户同意** — 根目录只能有 `canwork/` `memory/` `tempdata/` `mydata/`
3. **数据只存上述四个目录**
4. **每次新对话先 `recall` 加载记忆**

---

## 一、侧边栏魔改

### 涉及文件
| 文件 | 修改 |
|------|------|
| `packages/tui/src/app.tsx` | 添加 `registerSpinner` |
| `packages/tui/src/routes/session/index.tsx` | 添加 `registerSpinner` + `position:relative` + ☰按钮 + 遮罩关闭 |
| `packages/tui/src/routes/session/sidebar.tsx` | 替换为增强版侧边栏 |

### sidebar.tsx 功能清单
- History 列表：同目录最近 20 条会话，按更新时间排序
- 子对话过滤：`!s.parentID` 的不显示
- 当前会话高亮：▶ + `theme.backgroundElement` 背景
- 删除按钮：padding 扩大热区，点击一次确认（3秒超时），再点击执行删除
- Model 选择：点击打开 `DialogModel`
- +New 按钮：导航回首页

### index.tsx 改动
```tsx
// ☰ 浮动按钮（右下角 absolute）
<Show when={!sidebarVisible()}>
  <box position="absolute" bottom={1} right={2}
    backgroundColor={theme.border} paddingX={1}
    onMouseUp={() => { setSidebar(() => ...); setSidebarOpen(!sidebarVisible()) }}>
    <text fg={theme.background}>☰</text>
  </box>
</Show>

// 窄屏遮罩关闭
<Match when={!wide()}>
  <box position="absolute" top={0} left={0} right={0} bottom={0}
    alignItems="flex-end"
    onMouseUp={() => { setSidebar(() => "hide"); setSidebarOpen(false) }}>
    <box position="absolute" top={0} right={0} width={42} height="100%"
      onMouseUp={(e) => e?.stopPropagation?.()}>
      <Sidebar sessionID={route.sessionID} />
    </box>
  </box>
</Match>
```

---

## 二、品牌文字修改

### Logo 组件
- 文件：`packages/tui/src/logo.ts`
- 左侧 ASCII art 拼出 "OPEN"，右侧拼出 "CODE"
- 修改左侧即可改变品牌名

### 首页 Logo 替换
- 文件：`packages/tui/src/routes/home.tsx`
- 使用 `<pluginRuntime.Slot name="home_logo" mode="replace">` 可被插件替换
- 直接渲染纯文字：`<text fg={theme.textMuted}>Live</text><text fg={theme.text}>Code</text>`
- 或用 `<box border={["top","bottom"]} borderColor={theme.primary}>` 加边框美化

### 注意事项
- 需要 `import { useTheme } from "../context/theme"` 和 `const { theme } = useTheme()`
- logo.tsx 中 `bold` 参数控制粗体，右半部分默认 `true`
- 取消粗体：`true` → `!!props.ink`

---

## 三、主题系统

### 工作原理
- 主题 JSON 位于 `packages/tui/src/theme/assets/*.json`
- 在 `packages/tui/src/theme/index.ts` 中 import 并注册到 `DEFAULT_THEMES`
- 支持 8 位 hex `#RRGGBBAA`（alpha 通道）
- 特殊值 `"transparent"` 表示全透明

### 主题 JSON 结构
```json
{
  "$schema": "https://opencode.ai/theme.json",
  "defs": { "name": "#hexcolor" },
  "theme": {
    "primary": "defs中的引用名",
    "background": "transparent",
    "text": "#ffffff",
    "textMuted": "#888888",
    // ... 所有 theme 颜色
  }
}
```

### 自定义主题清单
| 主题 | 背景 | 主色 | 风格 |
|------|------|------|------|
| `livecode` | `#c7edcc` 实心 | `#5b8c5b` | 豆沙绿护眼（浅色） |
| `livecode2` | `transparent` | `#00d4ff` | 深色霓虹磨砂（配合终端 acrylic） |
| `livecode3` | `#f5efe8` 实心 | `#e8a87c` | 暖沙浅色 |
| `livecode4` | `#e8f4f8` 实心 | `#55bbee` | 晴空蓝浅色 |
| `livecode5` | `#f2ecf8` 实心 | `#a888dd` | 淡紫雾浅色 |

### 注册新主题
1. 创建 JSON 文件到 `packages/tui/src/theme/assets/`
2. 在 `theme/index.ts` 中添加：
   ```ts
   import mytheme from "./assets/mytheme.json" with { type: "json" }
   ```
3. 在 `DEFAULT_THEMES` 中添加：`mytheme,`

### 切换主题
`ctrl+p` → `/themes` → 选择主题名

### 壁纸作为背景
- 图片放到 `mydata/wallpaper.png`（跟随项目搬迁）
- 用 Python 提取主色调：`PIL.Image.quantize(16)` + `Counter`
- Windows Terminal 配置：
  ```json
  "backgroundImage": "I:\\opencode-1.17.9\\mydata\\wallpaper.png",
  "backgroundImageOpacity": 0.6,
  "useAcrylic": true,
  "opacity": 65
  ```

---

## 四、UI 美化

### 1. 消息气泡
文件：`packages/tui/src/routes/session/index.tsx`

**UserMessage** — 添加 **You** 标签：
```tsx
<box marginBottom={1}>
  <text fg={color()}><b>You</b></text>
</box>
```

**AssistantMessage** — 添加模型名 + 耗时：
```tsx
<box marginTop={1} marginBottom={0} paddingLeft={3}>
  <text fg={theme.primary}><b>{model()}</b></text>
  <Show when={duration() > 0}>
    <text fg={theme.textMuted}> · {Math.round(duration() / 1000)}s</text>
  </Show>
</box>
```

### 2. 对话框
文件：`packages/tui/src/ui/dialog.tsx`

顶部加主色光条：
```tsx
<box height={1} backgroundColor={theme.primary} width="100%" />
```

### 3. 底部状态栏
文件：`packages/tui/src/routes/session/footer.tsx`

- 添加 `border={["top"]} borderColor={theme.border}` 分隔线
- LSP 使用 `theme.success` 绿色圆点 ●
- MCP 使用 `theme.primary` 青色圆点 ●
- Permission 使用 `theme.warning` 警告色 ▲
- 未连接时使用 `theme.primary` ● 提示 /connect

### 4. 首页 Prompt 边框
文件：`packages/tui/src/routes/home.tsx`

```tsx
<box border={["top", "bottom"]} borderColor={theme.primary} paddingX={1}>
```

### 5. 启动加载页
文件：`packages/tui/src/component/startup-loading.tsx`

- `Spinner` 颜色改为 `theme.primary`
- 添加 `border={["left", "right"]} borderColor={theme.primary}`

---

## 五、编译 exe

### 命令
```powershell
$env:OPENCODE_CHANNEL="release"
$env:OPENCODE_VERSION="1.17.9"
& "I:\bun-windows-x64-baseline\bun.exe" run script/build.ts --single --baseline --skip-install --skip-embed-web-ui
```

### 参数说明
| 参数 | 作用 |
|------|------|
| `--single` | 仅编译当前平台 |
| `--baseline` | 使用 baseline（非 AVX2）bun |
| `--skip-install` | 跳过 native 依赖安装 |
| `--skip-embed-web-ui` | 跳过 Web UI 内嵌（加快编译） |

### 输出路径
```
packages\opencode\dist\opencode-windows-x64\bin\opencode.exe
packages\opencode\dist\opencode-windows-x64-baseline\bin\opencode.exe
```

### 首次编译前
```powershell
& "I:\bun-windows-x64-baseline\bun.exe" install --ignore-scripts
```

---

## 六、已知问题 & 解决方案

| 问题 | 原因 | 解决 |
|------|------|------|
| `zod/v4` 找不到 | bun `.bun` 缓存不完整 | `bun install --force` |
| `theme is not defined` | 组件内未调用 `useTheme()` | 添加 `const { theme } = useTheme()` |
| 浅色主题文字看不清 | 透明背景 + 深色文字 | 改实心浅色背景 + 深色文字，或保持浅色文字 |
| 浅色主题文字也看不清 | 面板透明度太高 | 提高 panel/element 的 alpha 值 |
| 编译 native 依赖失败 | 网络问题/跨平台包 | 加 `--skip-install` 跳过 |

---

## 七、记忆保存

每次重要修改后保存到 agentmemory：
```
agentmemory_remember(category="workflow", text="...", metadata={...})
```

新对话开始先加载：
```
agentmemory_recall(category="workflow", query="opencode tui mod")
```
