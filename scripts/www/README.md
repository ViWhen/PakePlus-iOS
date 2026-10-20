# 习惯打卡 · PWA

一个纯前端的习惯打卡应用。单页面、零依赖、零构建，数据只存在浏览器本地，
装到主屏幕后断网也能正常打卡。

---

## 一、文件

```
habit-pwa/
├── index.html      全部界面与逻辑（HTML + CSS + JS 单文件，无框架无构建）
├── manifest.json   PWA 清单：名称、图标、主题色、全屏模式
├── sw.js           Service Worker：离线缓存
├── icons/
│   ├── icon-192.png
│   ├── icon-512.png
│   ├── apple-touch-icon.png   iOS 添加到主屏用的 180×180
│   └── favicon-32.png
└── README.md
```

一共 5 个文件、约 90KB，没有任何 npm 依赖。

---

## 二、跑起来

### 本地

```bash
cd habit-pwa
python3 -m http.server 8080
# 打开 http://localhost:8080
```

或者 `npx serve .`、`php -S localhost:8080`，任意静态服务器都行。

> 直接双击 `index.html`（`file://`）也能用，但 Service Worker 不会注册，
> 因此没有离线缓存。要体验完整 PWA 请走 `http://localhost` 或 https。

### 部署

把整个目录丢给任意静态托管即可，不需要构建步骤：

- **Netlify / Vercel / Cloudflare Pages**：拖拽目录，或连 Git 仓库
- **GitHub Pages**：推到仓库，Settings → Pages 选分支
- 自己的服务器：放到站点根目录（SW 的作用域是它所在目录）

> Service Worker 只在 **https** 或 **localhost** 下生效，这是浏览器的硬性要求。

---

## 三、装到主屏幕

### iPhone / iPad（Safari）

1. Safari 打开站点（**必须 Safari**，微信/Chrome 内置浏览器不行）
2. 点底部「分享」按钮
3. 下滑选「**添加到主屏幕**」→ 添加
4. 桌面出现绿色对勾图标，点开就是全屏应用，没有地址栏

已做的适配：

- `apple-touch-icon`（180×180）作为桌面图标
- `apple-mobile-web-app-capable` → 全屏运行
- `viewport-fit=cover` + `env(safe-area-inset-*)` → 刘海屏和底部横条不遮挡内容
- 状态栏样式 `default`（白底黑字，和浅色界面衔接）
- 离线可用：iOS 11.3+ 支持 Service Worker，加到主屏后断网照样打开

### Android（Chrome）

打开站点 → 右上角菜单 → **安装应用 / 添加到主屏幕**。

### 桌面（Chrome / Edge）

地址栏右侧的「安装」图标，或菜单 → 安装习惯打卡。装完是一个独立窗口。

---

## 四、离线是怎么做的

`sw.js` 里两种策略：

| 请求类型 | 策略 | 原因 |
| --- | --- | --- |
| 页面导航 | 网络优先，失败回退缓存 | 在线时总能拿到最新版本，断网时用缓存兜底 |
| 静态资源 | 缓存优先 | 装过一次后秒开 |

安装时预缓存 `index.html`、`manifest.json`、4 个图标；激活时清掉旧版本缓存
（`CACHE` 常量里的版本号，改代码后记得加一）。

---

## 五、数据

| 项目 | 说明 |
| --- | --- |
| 存放 | `localStorage`，键名 `habit.pwa.v1` |
| 内容 | 习惯列表、每日打卡记录、音效开关 |
| 上传 | **不上传**，没有任何网络请求 |
| 会丢的场景 | 清理浏览器数据、无痕模式关窗口、换设备、换浏览器 |

### 备份与恢复

「数据」页：

- **导出备份** → 下载 `习惯打卡-备份-20261020-1744.json`，全量数据
- **导入恢复** → 选文件还原，导入前会告诉你备份里有多少习惯、多少天记录（会覆盖）
- **导出 CSV** → 长表，Excel 直接打开不乱码

备份 JSON 结构：

```json
{
  "app": "habit-checkin",
  "version": 1,
  "exportedAt": "2026-10-20T11:33:00.000Z",
  "habits": [{ "id": "h1", "name": "早起", "icon": "🌅", "period": "wake",
               "color": "#FF9500", "order": 0 }],
  "checkins": { "2026-10-20": ["h1", "h2"] },
  "settings": { "sound": true }
}
```

CSV：

```
日期,星期,时间段,习惯名称,图标
2026-10-20,星期二,起床,早起,🌅
```

---

## 六、功能

| 功能 | 说明 |
| --- | --- |
| 图标式打卡 | 每行 4 个，点图标打卡；填充设定颜色 + 白色对勾角标 + 弹跳动画 |
| 时段分组 | 起床 → 晨间 → 中午 → 午间 → 晚间 → 睡前 → 任意时间 → 全天，空组不显示 |
| 自定义图标 | 编辑页可用系统输入法的 emoji 面板敲任意 emoji（组合 emoji 不会被截断），另附 48 个常用图标 |
| 连续天数 | 顶部总连续（今天没打卡不立即断签）+ 图标下方的单项连续 |
| 日历 | 圆点区分全部/部分完成，点历史日期可补卡，当天列表已完成的自动置顶 |
| 音效 | WebAudio 实时合成，无需音频文件；可在数据页关闭 |
| 响应式 | 手机全屏；桌面自动收成 480px 宽的居中窗口 |

---

## 七、常见问题

**iPhone 上找不到「添加到主屏幕」**
→ 必须用 Safari 打开。微信、QQ、Chrome iOS 版的内置浏览器都没有这个入口。

**加到主屏后是白屏**
→ 首次要在 Safari 里完整打开一次，让 Service Worker 装好缓存再添加。

**改了代码但打开还是旧版**
→ 把 `sw.js` 里 `CACHE = 'habit-pwa-v3'` 的版本号加一，再刷新两次（第一次装新 SW，第二次生效）。
也可以开发者工具 → Application → Service Workers → Unregister。

**iOS 上没声音**
→ iOS 要求用户先触摸页面才能播放音频，第一次点击图标后就会正常。
静音键打开时 Safari 不播放 WebAudio，关掉即可。

**无痕模式数据丢了**
→ 无痕窗口的存储只在当前会话有效，属正常。请用普通窗口。

**想改配色 / 图标表 / 时段**
→ 均在 `index.html` 内：CSS 变量 `--brand`、`const PALETTE`、`const ICONS`、`const PERIODS`。
