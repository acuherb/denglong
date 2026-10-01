预览页面已经可以正常访问，页面内容显示“春节快乐！灯笼测试页 新年快乐”，说明灯笼效果已经正常渲染。下面是加入预览地址后的完整 README：

```markdown
# 春节灯笼 · Spring Lantern

一段纯前端的春节自动挂灯笼脚本，内置 2025-2050 年春节日期表，到点自动挂，过了自动撤，无需手动开关。

**在线预览**：[https://acuherb.github.io/denglong/](https://acuherb.github.io/denglong/)

**效果**：页面左右两侧各挂一只大红灯笼，缓缓摆动，进入春节区间自动出现。

---

## ✨ 特性

- **自动判断日期**：内置 2025-2050 年的春节公历日期，到达春节区间自动渲染
- **零依赖**：纯原生 JavaScript + CSS，不引入任何外部库
- **不挡点击**：灯笼容器 `pointer-events: none`，不会影响页面正常交互
- **响应式**：720px、420px 两档断点，手机端灯笼自动变小、贴边
- **无闪烁**：无鼠标监听逻辑，灯笼默认常显，摆动动画由 CSS 驱动
- **可关闭**：不在日期区间内时脚本直接 return，不注入任何 DOM

---

## 🖼️ 效果示意

- 左灯笼：`新 年`
- 右灯笼：`快 乐`
- 摆幅：±10°，周期 5 秒
- 灯笼底部流苏：3 秒一次的独立摆动

预览地址：[https://acuherb.github.io/denglong/](https://acuherb.github.io/denglong/)

---

## 🚀 使用方式

### 方式一：只挂首页

把整段代码追加到：

```
layouts/_partials/custom/profile.html
```

### 方式二：全站每页都挂

1. 创建 `layouts/_partials/custom/footer.html`，把 `<script>` 那段放进去。
2. 在 `config/_default/params.toml` 里注册：

```toml
[custom_partials]
footer = [
  "custom/footer.html",
]
```

---

## ⚙️ 配置项

打开脚本，改这几个变量即可：

| 变量 | 默认值 | 说明 |
|---|---|---|
| `TEXT_LEFT` | `'新年'` | 左灯笼显示的文字 |
| `TEXT_RIGHT` | `'快乐'` | 右灯笼显示的文字 |
| `DAYS_BEFORE` | `7` | 春节前多少天开始挂 |
| `DAYS_AFTER` | `15` | 春节后多少天撤掉 |

**示例**：想从腊月廿三挂到二月初二，可改为：

```js
var DAYS_BEFORE = 15;
var DAYS_AFTER  = 30;
```

---

## 📐 原理

### 日期判断

`SPRING_FESTIVALS` 是一张 `年 -> 春节公历日期` 的映射表。脚本运行时会：

1. 取当前年份 `year` 和 `year + 1` 两年的春节日期（跨年时正月初一可能在明年 1 月，所以两个都要看）
2. 分别计算今天与春节的天数差 `diffDays`
3. 只要任一年落在 `[-DAYS_BEFORE, DAYS_AFTER]` 区间内，就渲染灯笼
4. 否则直接 `return`

### 样式注入

灯笼的 CSS 和 HTML 都在脚本里以字符串形式存在，运行到日期判断通过后：

- 把 `<style>` 追加到 `<head>`
- 把两个 `.lantern-*` 容器追加到 `<body>`

避免在页面上硬编码一堆 HTML，也方便独立分发。

---

## ⚠️ 注意事项

### 1. 日期表只到 2050 年

`SPRING_FESTIVALS` 覆盖 2025 到 2050 年。**2051 年之后需要手动补充**，否则判断会失效（脚本返回 false，不挂灯笼）。

补充方式：每年农历十二月，往表里加一行即可：

```js
'2051': '2051-02-xx',
'2052': '2052-02-xx',
```

春节日期可以从中国科学院紫金山天文台发布的历书或任意权威农历查询工具获取。

### 2. 首页 vs 全站

`profile.html` 只在首页生效。如果要全站每页都挂，按上面的「方式二」操作。

### 3. 与主题 z-index 的关系

灯笼用 `z-index: 9999`，比常规内容高，但因为 `pointer-events: none`，不会遮挡任何交互。如果你有其他也设了 `z-index: 9999+` 的组件（比如某些弹窗），可能会被灯笼盖住。视情况调整脚本里的 z-index。

### 4. 不适用于 iframe 内嵌

脚本没有做 `window.top !== window` 的判断。如果你的页面会被嵌入 iframe（比如小组件预览），灯笼会在 iframe 内也渲染。如果不需要，可以在脚本开头加：

```js
if (window.top !== window) return;
```

### 5. 版权与来源

灯笼样式参考自开源社区流传的「春节挂灯笼」CSS 方案，本脚本对其做了日期判断和结构重写。原样式未标注明确许可证，如有顾虑可自行替换为自己的 CSS 实现。

---

## 🎨 自定义

### 换颜色

CSS 里颜色集中在几处：

- 灯笼主体：`.lantern-center { background-color: red }`
- 金色边框与流苏：`#ecaa2f`
- 阴影：`0 0 80px -10px #f00`

### 换文字方向或字号

`.lantern-text` 里：

- `writing-mode: vertical-lr;` 控制竖排（默认）
- 改 `horizontal-tb` 可改横排
- `font-size: 28px;` 控制字号

### 换摆幅或周期

`@keyframes lantern` 里：

- `rotate(-10deg)` / `rotate(10deg)` 控制摆幅
- `.lantern-container` 的 `animation: lantern 5s ...` 控制周期

---

## 📄 许可

MIT License。可自由用于个人或商业站点，保留出处即可。

---

> 观以明理，颐以养正。
