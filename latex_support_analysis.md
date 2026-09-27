# DeepSeek 网页 LaTeX 命令支持分析

- KaTeX 层：**KaTeX 0.16.22**（懒加载 chunk `static/katex.ef343f2527.js` + `static/katex.effce08cb3.css`），命令字面量 **1059** 个（函数+宏+符号引用）。完整清单见 `katex_cmds.txt`。
- 兜底层：**MicroTeX WASM**（`static/18372.262cb82004.js` + `static/microtex-wasm.c7b36d1fce.wasm`，约 3MB），仅当 KaTeX `renderToString` 抛错时懒加载，输出 SVG。
- 判定方法：KaTeX 层 = katex chunk 内 `"\\name"` 字符串；WASM 层 = wasm 二进制中命令表/宏表字符串。字符串存在 ≠ 一定可用（有少量 C++ 运行时噪音），但主流命令的判定可靠。

| 命令 | KaTeX 层 | WASM 层 | 实际渲染路径 |
|---|---|---|---|
| `\fbox` | ✓ | ✓ | KaTeX (HTML) |
| `\framebox` | — | — | 不支持 |
| `\boxed` | ✓ | ✓ | KaTeX (HTML) |
| `\ovalbox` | — | ✓ | WASM SVG 兜底 |
| `\doublebox` | — | ✓ | WASM SVG 兜底 |
| `\shadowbox` | — | ✓ | WASM SVG 兜底 |
| `\boxbox` | — | ✓ | WASM SVG 兜底 |
| `\boxonbox` | — | ✓ | WASM SVG 兜底 |
| `\mbox` | — | ✓ | WASM SVG 兜底 |
| `\fcolorbox` | ✓ | ✓ | KaTeX (HTML) |
| `\colorbox` | ✓ | — | KaTeX (HTML) |
| `\dashbox` | — | — | 不支持 |
| `\rotatebox` | — | ✓ | WASM SVG 兜底 |
| `\scalebox` | — | ✓ | WASM SVG 兜底 |
| `\resizebox` | — | ✓ | WASM SVG 兜底 |
| `\reflectbox` | — | ✓ | WASM SVG 兜底 |
| `\raisebox` | ✓ | ✓ | KaTeX (HTML) |
| `\ce` | ✓ | ✓ | KaTeX (HTML) |
| `\pu` | ✓ | ✓ | KaTeX (HTML) |
| `\bond` | — | ✓ | WASM SVG 兜底 |
| `\longdiv` | — | ✓ | WASM SVG 兜底 |
| `\matrix` | ✓ | ✓ | KaTeX (HTML) |
| `\pmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\bmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\Bmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\vmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\Vmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\smallmatrix` | ✓ | ✓ | KaTeX (HTML) |
| `\cases` | ✓ | — | KaTeX (HTML) |
| `\aligned` | ✓ | ✓ | KaTeX (HTML) |
| `\align` | ✓ | ✓ | KaTeX (HTML) |
| `\alignat` | ✓ | ✓ | KaTeX (HTML) |
| `\gather` | ✓ | ✓ | KaTeX (HTML) |
| `\multline` | — | ✓ | WASM SVG 兜底 |
| `\flalign` | — | ✓ | WASM SVG 兜底 |
| `\array` | ✓ | ✓ | KaTeX (HTML) |
| `\subarray` | ✓ | — | KaTeX (HTML) |
| `\equation` | ✓ | — | KaTeX (HTML) |
| `\split` | ✓ | ✓ | KaTeX (HTML) |
| `\text` | ✓ | ✓ | KaTeX (HTML) |
| `\textbf` | ✓ | ✓ | KaTeX (HTML) |
| `\textit` | ✓ | ✓ | KaTeX (HTML) |
| `\texttt` | ✓ | ✓ | KaTeX (HTML) |
| `\textsc` | — | — | 不支持 |
| `\textnormal` | ✓ | — | KaTeX (HTML) |
| `\textsuperscript` | — | ✓ | WASM SVG 兜底 |
| `\textsubscript` | — | ✓ | WASM SVG 兜底 |
| `\prescript` | — | ✓ | WASM SVG 兜底 |
| `\intertext` | — | ✓ | WASM SVG 兜底 |
| `\lceil` | ✓ | ✓ | KaTeX (HTML) |
| `\rceil` | ✓ | ✓ | KaTeX (HTML) |
| `\lfloor` | ✓ | ✓ | KaTeX (HTML) |
| `\rfloor` | ✓ | ✓ | KaTeX (HTML) |
| `\abs` | — | — | 不支持 |
| `\norm` | — | — | 不支持 |
| `\ceil` | — | — | 不支持 |
| `\floor` | — | — | 不支持 |
| `\cancel` | ✓ | — | KaTeX (HTML) |
| `\bcancel` | ✓ | ✓ | KaTeX (HTML) |
| `\xcancel` | ✓ | ✓ | KaTeX (HTML) |
| `\sout` | ✓ | — | KaTeX (HTML) |
| `\phantom` | ✓ | — | KaTeX (HTML) |
| `\hphantom` | ✓ | ✓ | KaTeX (HTML) |
| `\vphantom` | ✓ | ✓ | KaTeX (HTML) |
| `\smash` | ✓ | ✓ | KaTeX (HTML) |
| `\mathclap` | ✓ | ✓ | KaTeX (HTML) |
| `\clap` | ✓ | — | KaTeX (HTML) |
| `\llap` | ✓ | ✓ | KaTeX (HTML) |
| `\rlap` | ✓ | ✓ | KaTeX (HTML) |
| `\sideset` | — | ✓ | WASM SVG 兜底 |
| `\overbrace` | ✓ | ✓ | KaTeX (HTML) |
| `\underbrace` | ✓ | ✓ | KaTeX (HTML) |
| `\overbracket` | — | ✓ | WASM SVG 兜底 |
| `\underbracket` | — | ✓ | WASM SVG 兜底 |
| `\widehat` | ✓ | ✓ | KaTeX (HTML) |
| `\widetilde` | ✓ | ✓ | KaTeX (HTML) |
| `\accentset` | — | ✓ | WASM SVG 兜底 |
| `\underaccent` | — | ✓ | WASM SVG 兜底 |
| `\textcolor` | ✓ | ✓ | KaTeX (HTML) |
| `\colorbox` | ✓ | — | KaTeX (HTML) |
| `\fcolorbox` | ✓ | ✓ | KaTeX (HTML) |
| `\definecolor` | — | ✓ | WASM SVG 兜底 |
| `\color` | ✓ | ✓ | KaTeX (HTML) |
| `\KaTeX` | ✓ | — | KaTeX (HTML) |
| `\TeX` | ✓ | ✓ | KaTeX (HTML) |
| `\LaTeX` | ✓ | ✓ | KaTeX (HTML) |

## 典型结论

- `\ovalbox`：KaTeX 0.16.22 不支持 → 走 MicroTeX WASM → 以 SVG 渲染。这就是它「被支持」的原因。
- fancybox 家族 `\ovalbox` / `\doublebox` / `\shadowbox` / `\boxbox` / `\boxonbox` / `\mbox` 只有 WASM 层支持（`\fbox` / `\fcolorbox` 两层都有，走 KaTeX）。
- graphicx 的 `\rotatebox` / `\scalebox` / `\resizebox` / `\reflectbox` 只有 WASM 层支持。
- **KaTeX 层内置了 mhchem 扩展**：`\ce{}` / `\pu{}` 走 KaTeX HTML 渲染；`\bond{}` 走 WASM。
- 环境语法 `\begin{...}`：amsmath 主流环境（matrix 全家、cases、aligned、align、gather、equation、split、subarray、darray、dcases、rcases）KaTeX 都支持；**`multline` 和 `flalign` KaTeX 不支持，会落到 WASM 渲染**。`\pmatrix{...}` 这类 Plain TeX 简写（不带 begin）只有 WASM 支持。
- mathtools 风格的 `\overbracket` / `\underbracket` / `\accentset` / `\underaccent` / `\prescript` / `\sideset`、以及 `\longdiv`，只有 WASM 层支持。
- 同名命令冲突时 **KaTeX 优先**（HTML 路径）；两个渲染器都失败 → 显示 LaTeX 原文并埋点上报。

## 怎么肉眼区分一条公式走了哪层
- KaTeX 渲染：DOM 里是 `<span class="katex">…</span>`（HTML+CSS 排版）。
- WASM 渲染：DOM 里是 `<span class="ds-markdown-math-svg">` + 一个 `<svg>`（这正是 `\ovalbox` 公式的样子）。

## 原始数据
- `katex_cmds.txt`：KaTeX 层 1059 个命令名
- `wasm_only.txt`：wasm 字符串中 KaTeX 层没有的候选（含 C++ 噪音，仅供深挖）