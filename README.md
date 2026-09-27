# DeepSeek 网页版 LaTeX 命令速查表

逆向自 chat.deepseek.com 前端 bundle 的 LaTeX 支持清单：**1186 条命令 + 35 个环境**，每条标注走哪一层渲染。

**在线速查表（GitHub Pages）**：https://louisyang841.github.io/deepseek-latex-cheatsheet/

## DeepSeek 的 LaTeX 渲染架构

| | 主渲染器 | 兜底渲染器 |
|---|---|---|
| 引擎 | KaTeX 0.16.22 | MicroTeX WASM（C++ LaTeX 引擎 Emscripten 编译） |
| 加载 | 首个公式出现时懒加载 `katex.*.js` chunk | KaTeX 抛错后才懒加载 |
| 输出 | HTML + CSS（DOM 特征 `span.katex`） | SVG（DOM 特征 `span.ds-markdown-math-svg`） |

渲染管线：自研流式 markdown 解析器识别公式（支持打字机式输出下未闭合的 `$…$` / `\(…\)`）→ KaTeX `renderToString(value, { throwOnError: true, strict: false })` → 抛错则整条公式交给 MicroTeX WASM 以 SVG 重画 → 再失败显示原文并埋点上报。

**WASM 的本质是几何盒子排版引擎**：图层堆叠、坐标变换、画框都极擅长（七层套娃无压力），但它不是文字处理软件——没有文字流断行引擎。这解释了下面为什么「自动换行」类命令全是死穴。

**关键规则：一条公式只要混进一个 KaTeX 不认识的命令，整条都会落到 SVG 层**。玩花样时要么全用 KaTeX 层命令，要么整条全用 WASM 专属命令，不要混。

## ❌ 致命雷区（实测：崩溃原因高度集中，只有三类）

**① 文本自动换行（最致命）**——`\parbox`、`\shortstack`、`\begin{tabular}` 的 `p{4.5cm}` 列：WASM 没有断行引擎，凡是要它「自动折行」必崩。唯一解法是 `\begin{array}` + `\\` 人工拆行。

**② 跨层混用**——KaTeX 专属命令（`\sout` `\cancel`）与 WASM 专属命令（`\overparen` `\ovalbox`）别写进同一个 `$$`：降级后 WASM 不认识 KaTeX 的命令，直接报错。解法是隔离原则：两层严格拆到独立公式块。

**③ 零散残缺**——`\hfill` 非数学模式单用（用 array 的 `l`/`r` 列替代）；`\hdashline`（用 `\hline` 或 `\cline` 替代）；`\definecolor` 的 HTML 模式（用 rgb 浮点数或内置 dvips 色名替代）；同条公式内 `\definecolor` 后立即 `\bgcolor`（分块定义再调用）。

## ✅ 黄金开发规范

- **WASM 层管排版 UI**：`\ovalbox`（胶囊）`\shadowbox`（阴影）`\doublebox`（双线框）`\rotatebox`（旋转）`\scalebox`（缩放）`\reflectbox`（镜像）
- **KaTeX 层管文本公式**：`\sout` `\ce{}` `\cancel`、基础数学符号
- **安全底座**：`array`（分屏/对齐/多行）、`tabular`（状态面板，只用 `\hline`）
- **颜色系统**：内置 dvips 色名（`wildstrawberry`、`periwinkle`）优先；自定义色用 `rgb{1.0,0.71,0.75}` 独立块定义
- **排版哲学**：长文本必须人工拆解——所有让引擎自动换行的手段，全是死路

> 实测：七层盒子套娃（`\reflectbox`→`\doublebox`→`\scalebox`→`\rotatebox`→`\shadowbox`→`\ovalbox`→`\colorbox`）完美渲染；tabular 单元格塞四层套娃也安全（只要不用 p{} 列）。**嵌套深度不是问题，文本自动换行才是唯一红线。**

## 清单规模

| 层 | 数量 | 说明 |
|---|---|---|
| KaTeX 层 | 1020 | 主力，HTML 渲染，观感与网页正文一致 |
| WASM 专属 | 166 | KaTeX 会直接报错的彩蛋命令 |
| 环境 `\begin{...}` | 35 | 两层对照（`multline` / `flalign` 仅 WASM 有） |

## WASM 专属亮点

- **fancybox 盒子**：`\ovalbox` `\doublebox` `\shadowbox` `\boxbox` `\mbox`
- **graphicx 变换**：`\rotatebox` `\scalebox` `\resizebox` `\reflectbox`
- **表格**：`\begin{tabular}` + `\multirow` `\multicolumn` `\cellcolor` `\rowcolor` `\hdotsfor`
- **100 个 dvips 色名**：`\textcolor{wildstrawberry}{...}`、`maroonA`–`tealE` 系列色号
- **化学**：`\bond{}`（`\ce{}` / `\pu{}` 反而走 KaTeX——它内置了 mhchem）
- **其他**：`\longdiv`、`\DeclareMathOperator`、`\prescript`、`\sideset`、mathtools 括号装饰

## 文件

| 文件 | 内容 |
|---|---|
| `index.html` | 可搜索 / 按层筛选 / 点击复制的在线速查表 |
| `deepseek-latex-cheatsheet.md` | Markdown 版，可直接投喂给 DeepSeek |
| `latex_support_analysis.md` | 两层渲染路径分析报告（证据链） |
| `data/katex_cmds.txt` | KaTeX 层全量 1059 个命令字面量 |
| `data/wasm_only.txt` | WASM 层原始候选（含 C++ 噪音，仅深挖用） |

## 方法

浏览器 view-source 保存首页 → 剥离 view-source 包装还原源码 → 提取 bundle 清单并下载 → `main.js` 定位渲染逻辑与 chunk 映射 → katex chunk（0.16.22）提取命令字面量 → microtex-wasm 二进制提取字符串 → 与 [MicroTeX](https://github.com/NanoMichael/MicroTeX) 官方源码词汇交叉验证，把 3798 个噪音候选洗出可信命令集。

## 注意

静态分析快照，DeepSeek 更新前端后可能漂移；个别条目实测无效属构建版本差异。判定基于字符串交叉验证，存在固有误差（`wasm_only.txt` 未洗的原始数据仅供深挖）。
