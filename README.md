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

**关键规则：一条公式只要混进一个 KaTeX 不认识的命令，整条都会落到 SVG 层**。玩花样时要么全用 KaTeX 层命令，要么整条全用 WASM 专属命令，不要混。

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
