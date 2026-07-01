# 能力提升计划 - 小学升年级学习系统

> 面向小学1-2年级学生的语数英三科能力提升学习平台，支持每日自动生成练习计划、智能出题、语音朗读、学习报告等功能。

---

## 📋 目录

- [项目简介](#-项目简介)
- [技术栈与框架介绍](#-技术栈与框架介绍)
- [项目结构](#-项目结构)
- [功能特性](#-功能特性)
- [题库概览](#-题库概览)
- [本地运行](#-本地运行)
- [外网部署（脱离本工具运行）](#-外网部署脱离本工具运行)
- [页面流程](#-页面流程)
- [数据持久化](#-数据持久化)
- [开发说明](#-开发说明)

---

## 🎯 项目简介

这是一个面向**小学1-2年级升年级学生**的学习能力提升系统，覆盖 **语文、数学、英语** 三科。

**核心思路：** 按真实小学试卷的题型比例，每天自动生成「半张试卷」量的练习题，支持逐题作答、答案检查、语音朗读（英语）、进度打卡、学习报告。

### 数据快照

| 科目 | 题数 | 能力项数 |
|:----|:---:|:-------:|
| 语文 | 785 | 5 |
| 数学 | 2,346 | 6 |
| 英语 | 505 | 5 |
| **总计** | **3,636** | **16** |

---

## 🛠 技术栈与框架介绍

### 项目包含两个版本

| 版本 | 目录 | 用途 | 技术栈 |
|:----|:----|:----|:------|
| **Uni-app 源码版** | `study-app/` | 可打包为 App / 小程序的源码 | Uni-app + Vue 3 + Vuex |
| **Web 测试版** | `study-app-test/` | 浏览器直接运行测试 | 纯 HTML + Vue 3 CDN + Vuex CDN |

### 框架详解

#### Uni-app（`study-app/`）

[Uni-app](https://uniapp.dcloud.net.cn/) 是一个使用 **Vue.js** 开发**所有前端应用**的跨平台框架。一套代码可编译到：

- **iOS / Android App**（通过 DCloud 打包）
- **微信小程序 / 支付宝小程序 / 百度小程序**
- **H5 网页**
- **快应用**

本项目的 Uni-app 源码使用了：

- **Vue 3**：组件化 UI 框架
- **Vuex 4**：状态管理（存储用户配置、打卡记录、学习进度）
- **uni-ui**：Uni-app 内置 UI 组件

#### Web 测试版（`study-app-test/`）

一个**独立的单页应用 HTML 文件**，通过 CDN 引入 Vue 3 和 Vuex，所有逻辑和题库数据打包在一个 HTML 中（或通过 JSON 文件动态加载）。

**为什么有两个版本？**

1. **Uni-app 源码版**用于最终打包成小程序或 App
2. **Web 测试版**用于快速开发和测试，无需安装开发环境，浏览器直接打开即可使用

两个版本共享同一套题库 JSON 数据文件。

### 核心依赖

| 依赖 | 版本 | 用途 |
|:----|:----|:----|
| Vue | 3.x | 前端 UI 框架 |
| Vuex | 4.x | 状态管理 |
| Uni-app | 3.x | 跨平台框架（App/小程序版本） |
| SpeechSynthesis API | 浏览器内置 | 英语语音朗读 |

---

## 📁 项目结构

```
study-app/                         # Uni-app 源码（可打包 App / 小程序）
├── App.vue                        # 应用入口
├── main.js                        # 启动文件（注册 Vuex）
├── manifest.json                  # Uni-app 配置文件
├── pages.json                     # 页面路由配置
├── package.json                   # 依赖配置
├── uni.scss                       # 全局样式变量
├── store/
│   └── index.js                   # Vuex 状态管理（核心逻辑）
├── utils/
│   ├── questionBank.js            # 题库加载工具
│   └── planGenerator.js           # 计划生成辅助函数
├── static/
│   ├── data/
│   │   ├── chinese.json           # 语文题库
│   │   ├── math.json              # 数学题库
│   │   └── english.json           # 英语题库
│   └── geometry/                  # 几何 SVG 图形资源
│       ├── square.svg
│       ├── triangle.svg
│       ├── circle.svg
│       ├── clock.svg
│       ├── number_line.svg
│       ├── money.svg
│       ├── cuboid.svg
│       └── tianzige.svg
├── pages/
│   ├── index/index.vue            # 首页 - 年级选择
│   ├── subject/index.vue          # 科目选择
│   ├── abilities/index.vue        # 能力项配置
│   ├── plan/index.vue             # 本周学习计划
│   ├── daily/index.vue            # 今日学习（核心答题页）
│   ├── content/index.vue          # 学习内容概览
│   └── report/index.vue           # 学习报告

study-app-test/                    # Web 测试版（浏览器直接运行）
├── index.html                     # 完整单页应用（含 Vue/Store/组件）
├── chinese.json                   # 语文题库（与源码版共用）
├── math.json                      # 数学题库
├── english.json                   # 英语题库
├── scenes/                        # AI 生成场景图
│   ├── park.png                   # 看图写话配图
│   ├── family.png
│   ├── rain.png
│   └── ...                        # 共19张场景图
└── geometry/                      # SVG 图形资源（与源码版共用）
    ├── triangle.svg
    ├── square.svg
    ├── circle.svg
    ├── clock.svg
    └── ...
```

---

## ✨ 功能特性

### 1. 按试卷题型比例出题

根据真实小学试卷分析，每项能力按考试占比分配权重：

| 科目 | 题型比例 | 每日题量（半张试卷） |
|:----|:--------|:-----------------:|
| 数学 | 口算 9题 + 填空 2题 + 应用 3题 + 操作 2题 + 乘除 2题 | **~18题** |
| 语文 | 生字 4题 + 句子 3题 + 拼音 4题 + 看图写话 1题 + 阅读 1篇 | **~13题** |
| 英语 | 单词 4题 + 句子 3题 + 拼读 2题 + 对话 2题 + 作文 1题 | **~12题** |

### 2. 每天自动生成周计划

- 周一至周日，每天独立的学习任务
- 所有已启用的能力项每日全部出题（不轮转）
- 题目从题库随机抽取，确保每次重新生成都是新题
- 题库用完时自动提示

### 3. 交互式答题

- **选择题**：点击选项直接选择
- **数学计算**：数字输入框
- **填空题**：文本输入框
- **句子/作文**：多行文本域
- **检查答案**：输入后点击「检查答案」自动判断对错
- **提示系统**：每道题配有教学提示（不是直接给答案）

### 4. 英语语音朗读 🔊

每个英语单词和句子旁边都有 🔊 按钮，点击即可听到**纯正美式发音**（使用浏览器内置 Web Speech API，无需联网下载语音包）。

### 5. 图片辅助

- **看图写话**：19 个场景配有 AI 生成的水彩风格场景图
- **几何图形**：用 SVG 矢量图展示三角形、正方形、圆形、长方体等
- **钟表认读**：SVG 钟表图形辅助
- **汉字书写**：田字格 SVG 辅助
- **比大小**：数轴 SVG 辅助

### 6. 学习报告

- 学习天数统计
- 完成题目总数
- 连续打卡天数
- 本周每日完成进度
- 成就墙（6 个成就徽章）

### 7. 数据持久化

全部用户数据存储在浏览器 localStorage 中，**关闭页面不会丢失**。

---

## 📊 题库概览

### 语文（785题）

| 能力项 | 题数 | 说明 |
|:------|:---:|:-----|
| 写生字 (c1) | 205 | 一年级真实汉字，带拼音、笔画、组词、例句 |
| 练句子 (c2) | 55 | 仿写、扩句、把字句、被字句、造句、修改病句 |
| 看图写话 (c3) | 30 | 20个场景配AI图片，10个文字场景 |
| 阅读理解 (c4) | 15 | 短文阅读 + 3个问题/篇 |
| 拼音巩固 (c5) | 480 | 拼读、标调、辨音、综合 |

### 数学（2,346题）

| 能力项 | 题数 | 说明 |
|:------|:---:|:-----|
| 20以内加减 (m1) | 474 | 1-20 加减法 |
| 100以内加减 (m2) | 424 | 20-100 进退位加减 |
| 比一比 (m3) | 298 | 大小/多少/轻重/长短比较 |
| 表内乘除法 (m4) | 350 | 乘法口诀、除法入门 |
| 应用题 (m5) | 500 | 一步/两步应用题 |
| 图形与时间 (m6) | 300 | 图形识别、钟表认读、人民币 |

### 英语（505题）

| 能力项 | 题数 | 说明 |
|:------|:---:|:-----|
| 认单词 (e1) | 240 | 分类词汇认读，配例句+语音 |
| 说句子 (e2) | 125 | 日常简单句子，配翻译+语音 |
| 自然拼读 (e3) | 62 | 字母发音、CVC拼读、发音辨音 |
| 简单对话 (e4) | 35 | 日常情景对话练习 |
| 小作文 (e5) | 43 | 看图说话、自我介绍、描述 |

---

## 🚀 本地运行

### 方式一：Web 测试版（最简单，推荐）

不需安装任何环境，**直接打开浏览器**：

```bash
# 方法1：直接用浏览器打开（但无法加载JSON题库数据）
study-app-test/index.html
```

**建议使用 HTTP 服务器运行（否则 JSON 文件无法通过 AJAX 加载）：**

#### Windows

```bash
# 使用 Node.js 的 http-server
npx http-server study-app-test -p 3000

# 或使用 Python
cd study-app-test
python -m http.server 3000
```

#### macOS / Linux

```bash
cd study-app-test
# Python 3
python3 -m http.server 3000
# 或 Node.js
npx http-server . -p 3000
```

然后在浏览器打开 `http://localhost:3000`。

### 方式二：Uni-app 源码版（需安装开发环境）

#### 前提条件

- [Node.js](https://nodejs.org/) >= 16.x
- [HBuilderX](https://www.dcloud.io/hbuilderx.html)（推荐）或 npm

#### 使用 HBuilderX（简单方式）

1. 打开 HBuilderX
2. 菜单 `文件 → 导入 → 导入本地项目`
3. 选择 `study-app/` 目录
4. 顶部工具栏选择运行目标（浏览器/小程序模拟器/手机）
5. 点击运行

#### 使用命令行

```bash
cd study-app

# 安装依赖
npm install

# 以 H5 方式运行（浏览器预览）
npm run dev:h5

# 构建微信小程序
npm run dev:mp-weixin

# 构建 App
npm run build:app
```

---

## 🌐 外网部署（脱离本工具运行）

### 方式一：使用 CloudStudio（最简单）

在 WorkBuddy 中执行：

```bash
# 部署 study-app-test 目录到云环境
# workbuddy_cloudstudio_deploy 工具会返回一个公网链接
```

部署成功后，会获得类似 `https://xxxx.app.codebuddy.work` 的外网地址。

### 方式二：部署到任意静态托管服务

将 `study-app-test/` 目录下的所有文件上传到：

| 服务 | 说明 | 费用 |
|:----|:----|:----|
| **Vercel** | 自动部署，绑定 git 仓库 | 免费 |
| **Netlify** | 拖拽上传或 Git 部署 | 免费 |
| **GitHub Pages** | 上传到 gh-pages 分支 | 免费 |
| **腾讯云静态网站** | 对象存储 + CDN | 按量付费 |
| **阿里云 OSS** | 对象存储 | 按量付费 |

**部署步骤：**

1. 将 `study-app-test/` 目录上传到静态托管服务
2. 确保 `index.html` 设为入口文件
3. 确保 `chinese.json`、`math.json`、`english.json` 可在同路径下访问
4. 访问分配的 URL 即可

### 方式三：Node.js 简易服务器

```bash
# 1. 安装 Node.js（如果没装）
# 2. 启动静态文件服务器
cd study-app-test
npx http-server . -p 3000 --cors

# 3. 用内网穿透工具（如 frp、ngrok）暴露到外网
# ngrok 示例：
npx ngrok http 3000
```

---

## 📱 页面流程

```
首页（选择年级）
    │
    ▼
选择科目（语文/数学/英语，可多选）
    │
    ▼
配置能力项（勾选想练习的能力）
    │
    ▼
生成本周计划（周一至周日）
    │
    ├──→ 每日学习（打卡答题）
    │        │
    │        ├── 查看题目
    │        ├── 输入答案
    │        ├── 检查答案（自动判断对错）
    │        ├── 查看提示（解题思路）
    │        ├── 语音朗读（英语）
    │        └── 打卡完成
    │
    └──→ 学习报告
             │
             ├── 学习天数统计
             ├── 完成总题数
             ├── 连续打卡天数
             ├── 本周进度条
             └── 成就徽章
```

---

## 💾 数据持久化

所有用户数据存储在浏览器的 `localStorage` 中，key 为 `study_app_data`：

| 数据 | 说明 |
|:----|:-----|
| `grade` | 选择的年级 |
| `subjects` | 已选科目列表 |
| `abilities` | 已启用的能力项 |
| `weeklyPlan` | 当前周计划（含每日题目 ID） |
| `checkIns` | 打卡记录（日期 → 科目 → 题ID） |
| `usedQuestions` | 已用过的题目 ID（保证不重复出题） |
| `stats` | 统计数据（天数、题数、连续打卡） |

**清除数据：** 点击导航栏的「重置」按钮可清除所有数据。

---

## 🔧 开发说明

### 题库数据格式

#### 数学题示例

```json
{
  "id": "m1_001",
  "difficulty": 1,          // 1=基础, 2=进阶, 3=挑战
  "type": "计算",
  "question": "3 + 5 = ?",
  "answer": "8"
}
```

#### 语文汉字示例

```json
{
  "id": "c1_001",
  "difficulty": 1,
  "type": "生字",
  "character": "天",
  "pinyin": "tiān",
  "strokes": 4,
  "words": ["天空", "今天", "天气"],
  "sentence": "今天的天气真好。"
}
```

#### 英语单词示例

```json
{
  "id": "e1_001",
  "difficulty": 1,
  "type": "单词",
  "category": "动物",
  "word": "cat",
  "meaning": "猫",
  "sentence": "I have a cat."
}
```

#### 看图写话示例（带图片）

```json
{
  "id": "c3_01",
  "difficulty": 1,
  "type": "看图写话",
  "scene": "公园",
  "image": "scenes/park.png",
  "prompt": "图上画的是什么地方？有什么？请用一两句话写出来。",
  "keywords": "公园 花 草 树 蝴蝶",
  "example": "公园里有五颜六色的花朵..."
}
```

### 权重配置（每日出题数）

权重由 `abilityWeights` 控制，位于 `store/index.js` 的 `generatePlan` 方法中：

```javascript
const abilityWeights = {
  chinese: { c1: 4, c2: 3, c3: 1, c4: 1, c5: 4 },
  math: { m1: 5, m2: 4, m3: 2, m4: 2, m5: 3, m6: 2 },
  english: { e1: 4, e2: 3, e3: 2, e4: 2, e5: 1 }
};
```

### 添加新题目

1. 编辑对应科目的 JSON 文件
2. 按格式添加题目，确保 `id` 唯一
3. 同步更新两个版本（`study-app/static/data/` 和 `study-app-test/`）
4. 刷新页面即可生效

### 语音功能

英语语音朗读使用浏览器内置的 **Web Speech API**（`SpeechSynthesisUtterance`）：

```javascript
var utter = new SpeechSynthesisUtterance("Hello");
utter.lang = 'en-US';
utter.rate = 0.85;  // 语速略慢，适合儿童
window.speechSynthesis.speak(utter);
```

支持 Chrome、Edge、Safari 等现代浏览器。

---

## 📄 许可

本项目为个人学习辅助工具，题库内容仅供教育用途。

---

*最后更新: 2026-06-30*
