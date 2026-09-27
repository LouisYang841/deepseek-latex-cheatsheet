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

标记说明：**[W]** = 仅 WASM 层支持（KaTeX 会报错，走 SVG 兜底）；无标记 = KaTeX 层直接支持。

---
## 环境语法 `\begin{...}`

| 环境 | KaTeX 层 | WASM 层 |
|---|---|---|
| `\begin{Bmatrix}` | ✓ | ✓ |
| `\begin{Bmatrix*}` | ✓ | — |
| `\begin{Vmatrix}` | ✓ | ✓ |
| `\begin{Vmatrix*}` | ✓ | — |
| `\begin{align}` | ✓ | — |
| `\begin{align*}` | ✓ | — |
| `\begin{alignat}` | ✓ | ✓ |
| `\begin{aligned}` | ✓ | ✓ |
| `\begin{alignedat}` | ✓ | ✓ |
| `\begin{array}` | ✓ | ✓ |
| `\begin{bmatrix}` | ✓ | ✓ |
| `\begin{bmatrix*}` | ✓ | — |
| `\begin{cases}` | ✓ | — |
| `\begin{darray}` | ✓ | — |
| `\begin{dcases}` | ✓ | — |
| `\begin{displaymath} **[W]**` | — | ✓ |
| `\begin{drcases}` | ✓ | — |
| `\begin{equation}` | ✓ | — |
| `\begin{equation*}` | ✓ | — |
| `\begin{flalign} **[W]**` | — | ✓ |
| `\begin{gather}` | ✓ | ✓ |
| `\begin{gather*}` | ✓ | — |
| `\begin{gathered}` | ✓ | ✓ |
| `\begin{matrix}` | ✓ | — |
| `\begin{matrix*}` | ✓ | — |
| `\begin{multline} **[W]**` | — | ✓ |
| `\begin{pmatrix}` | ✓ | ✓ |
| `\begin{pmatrix*}` | ✓ | — |
| `\begin{rcases}` | ✓ | — |
| `\begin{smallmatrix}` | ✓ | ✓ |
| `\begin{smallmatrix*}` | ✓ | — |
| `\begin{split}` | ✓ | ✓ |
| `\begin{subarray}` | ✓ | — |
| `\begin{vmatrix}` | ✓ | ✓ |
| `\begin{vmatrix*}` | ✓ | — |

## WASM 专属命令（KaTeX 不认识，混用会整条 SVG 化）

**结构**：`\accentset` `\intertext` `\joinrel` `\longdiv` `\overparen` `\prescript` `\sfrac` `\shoveleft` `\shoveright` `\sideset` `\spbreve` `\spcheck` `\spddot` `\spdot` `\sphat` `\sptilde` `\stackbin` `\underaccent` `\underparen` `\undertilde`

**函数与算子**：`\arccot` `\arccsc` `\arcsec` `\csch` `\DeclareMathOperator` `\DeclareMathSizes` `\sech`

**箭头**：`\array` `\Longmapsfrom` `\longmapsfrom` `\Longmapsto` `\Mapsfrom` `\Mapsto`

**关系符**：`\questeq`

**定界符**：`\Arrowvert` `\arrowvert` `\bangle`

**重音与装饰**：`\dotminus` `\iddots` `\overbrack` `\underbrack`

**字体与文本**：`\mathds` `\oldstylenums`

**文本符号**：`\textsubscript` `\textsuperscript`

**颜色**：`\bgcolor` `\definecolor` `\fgcolor`

**颜色名**：`\apricot` `\aquamarine` `\bittersweet` `\bluegreen` `\blueviolet` `\brown` `\burntorange` `\cadetblue` `\carnationpink` `\cornflowerblue` `\dandelion` `\darkorchid` `\emerald` `\forestgreen` `\fuchsia` `\goldenrod` `\greenyellow` `\junglegreen` `\limegreen` `\magenta` `\maroon` `\melon` `\midnightblue` `\olivegreen` `\orangered` `\peach` `\periwinkle` `\pinegreen` `\processblue` `\rawsienna` `\redorange` `\redviolet` `\rhodamine` `\royalblue` `\royalpurple` `\rubinered` `\salmon` `\sepia` `\skyblue` `\springgreen` `\thistle` `\turquoise` `\violetred` `\wildstrawberry` `\yellowgreen` `\yelloworange`

**间距与占位**：`\vspace`

**盒子与图形 (WASM)**：`\boxbox` `\boxonbox` `\doublebox` `\mbox` `\ovalbox` `\reflectbox` `\resizebox` `\rotatebox` `\scalebox` `\shadowbox`

**表格 (WASM)**：`\arrayrulecolor` `\cellcolor` `\columncolor` `\cornersize` `\hdots` `\hdotsfor` `\multicolumn` `\multirow` `\newcolumntype` `\rowcolor` `\tabular`

**TeX 原语与语法**：`\abovewithdelims` `\atopwithdelims` `\breakEverywhere` `\fatalIfCmdConflict` `\magnification` `\makeatletter` `\makeatother` `\overwithdelims` `\renewenvironment`

**杂项符号**：`\block` `\hybull` `\lhblk` `\marker` `\micro` `\origin` `\sqrtsign` `\uhblk`

**其他**：`\alignat` `\aligned` `\alignedat` `\Android` `\AndroidTeX` `\Bmatrix` `\bmatrix` `\celsius` `\displaymath` `\flalign` `\gather` `\hermitmatrix` `\pica` `\pix` `\pixel` `\pmatrix` `\Roman` `\roman` `\smallmatrix` `\split` `\underscore` `\Vmatrix` `\vmatrix`

## KaTeX 层全量命令（1020 个，去内部命令）

**结构**（55）：`\accentset` `\binom` `\Bra` `\bra` `\braket` `\Braket` `\cfrac` `\dbinom` `\dfrac` `\frac` `\genfrac` `\intertext` `\joinrel` `\Ket` `\ket` `\longdiv` `\mathbin` `\mathchoice` `\mathclose` `\mathinner` `\mathop` `\mathopen` `\mathord` `\mathpunct` `\mathrel` `\overbrace` `\overparen` `\overset` `\prescript` `\set` `\Set` `\sfrac` `\shoveleft` `\shoveright` `\sideset` `\spbreve` `\spcheck` `\spddot` `\spdot` `\sphat` `\sptilde` `\sqrt` `\stackbin` `\stackrel` `\substack` `\tbinom` `\tfrac` `\underaccent` `\underbrace` `\underparen` `\underset` `\undertilde` `\xleftarrow` `\xleftrightarrow` `\xrightarrow`

**希腊字母**（66）：`\alpha` `\Alpha` `\beta` `\Beta` `\Chi` `\chi` `\delta` `\Delta` `\digamma` `\epsilon` `\Epsilon` `\eta` `\Eta` `\Gamma` `\gamma` `\Iota` `\iota` `\Kappa` `\kappa` `\Lambda` `\lambda` `\Mu` `\mu` `\nu` `\Nu` `\Omega` `\omega` `\Phi` `\phi` `\pi` `\Pi` `\Psi` `\psi` `\Rho` `\rho` `\sigma` `\Sigma` `\Tau` `\tau` `\theta` `\Theta` `\thetasym` `\upsilon` `\Upsilon` `\varDelta` `\varepsilon` `\varGamma` `\varkappa` `\varLambda` `\varOmega` `\varphi` `\varPhi` `\varpi` `\varPi` `\varPsi` `\varrho` `\varSigma` `\varsigma` `\varTheta` `\vartheta` `\varUpsilon` `\varXi` `\Xi` `\xi` `\zeta` `\Zeta`

**函数与算子**（58）：`\arccos` `\arccot` `\arccsc` `\arcctg` `\arcsec` `\arcsin` `\arctan` `\arctg` `\arg` `\argmax` `\argmin` `\bmod` `\cos` `\cosec` `\cosh` `\cot` `\cotg` `\coth` `\csc` `\csch` `\ctg` `\cth` `\DeclareMathOperator` `\DeclareMathSizes` `\deg` `\det` `\dim` `\exp` `\gcd` `\hom` `\inf` `\injlim` `\ker` `\lg` `\lim` `\liminf` `\limsup` `\ln` `\log` `\max` `\min` `\mod` `\operatorname` `\plim` `\pmod` `\Pr` `\projlim` `\sec` `\sech` `\sin` `\sinh` `\sup` `\tanh` `\tg` `\th` `\varinjlim` `\varliminf` `\varprojlim`

**大型运算符**（22）：`\bigcap` `\bigcup` `\bigodot` `\bigoplus` `\bigotimes` `\bigsqcup` `\biguplus` `\bigvee` `\bigwedge` `\coprod` `\idotsint` `\iiiint` `\iiint` `\iint` `\int` `\intop` `\oiiint` `\oiint` `\oint` `\prod` `\smallint` `\sum`

**箭头**（125）：`\cdleftarrow` `\cdrightarrow` `\circlearrowleft` `\circlearrowright` `\curvearrowleft` `\curvearrowright` `\darr` `\Darr` `\dArr` `\dashleftarrow` `\dashrightarrow` `\downarrow` `\Downarrow` `\downdownarrows` `\downharpoonleft` `\downharpoonright` `\gets` `\harr` `\Harr` `\hArr` `\hookleftarrow` `\hookrightarrow` `\iff` `\impliedby` `\implies` `\Larr` `\lArr` `\larr` `\leadsto` `\Leftarrow` `\leftarrow` `\leftarrowtail` `\leftharpoondown` `\leftharpoonup` `\leftleftarrows` `\Leftrightarrow` `\leftrightarrow` `\leftrightarrows` `\leftrightharpoons` `\leftrightsquigarrow` `\Lleftarrow` `\Longleftarrow` `\longleftarrow` `\longleftrightarrow` `\Longleftrightarrow` `\Longmapsfrom` `\longmapsfrom` `\longmapsto` `\Longmapsto` `\longrightarrow` `\Longrightarrow` `\looparrowleft` `\looparrowright` `\Lrarr` `\lrarr` `\lrArr` `\Lsh` `\Mapsfrom` `\mapsto` `\Mapsto` `\models` `\nearrow` `\nleftarrow` `\nLeftarrow` `\nleftrightarrow` `\nLeftrightarrow` `\nrightarrow` `\nRightarrow` `\nwarrow` `\overleftarrow` `\overleftharpoon` `\overleftrightarrow` `\overrightarrow` `\Overrightarrow` `\overrightharpoon` `\Rarr` `\rArr` `\rarr` `\relbar` `\Relbar` `\rightarrow` `\Rightarrow` `\rightarrowtail` `\rightharpoondown` `\rightharpoonup` `\rightleftarrows` `\rightleftharpoons` `\rightrightarrows` `\rightsquigarrow` `\Rrightarrow` `\Rsh` `\searrow` `\swarrow` `\to` `\twoheadleftarrow` `\twoheadrightarrow` `\uarr` `\uArr` `\Uarr` `\underleftarrow` `\underleftrightarrow` `\underrightarrow` `\uparrow` `\Uparrow` `\updownarrow` `\Updownarrow` `\upharpoonleft` `\upharpoonright` `\upuparrows` `\xhookleftarrow` `\xhookrightarrow` `\xLeftarrow` `\xleftharpoondown` `\xleftharpoonup` `\xLeftrightarrow` `\xleftrightharpoons` `\xmapsto` `\xRightarrow` `\xrightharpoondown` `\xrightharpoonup` `\xrightleftarrows` `\xrightleftharpoons` `\xtofrom` `\xtwoheadleftarrow` `\xtwoheadrightarrow`

**关系符**（198）：`\approx` `\approxcolon` `\approxcoloncolon` `\approxeq` `\asymp` `\backepsilon` `\backsim` `\backsimeq` `\because` `\between` `\bowtie` `\Bumpeq` `\bumpeq` `\cdlongequal` `\circeq` `\colon` `\Colonapprox` `\colonapprox` `\coloncolon` `\coloncolonapprox` `\coloncolonequals` `\coloncolonminus` `\coloncolonsim` `\coloneq` `\Coloneq` `\Coloneqq` `\coloneqq` `\colonequals` `\colonminus` `\colonsim` `\Colonsim` `\cong` `\curlyeqprec` `\curlyeqsucc` `\dashv` `\dblcolon` `\doteq` `\Doteq` `\doteqdot` `\eqcirc` `\eqcolon` `\Eqcolon` `\eqqcolon` `\Eqqcolon` `\eqsim` `\eqslantgtr` `\eqslantless` `\equalscolon` `\equalscoloncolon` `\equiv` `\fallingdotseq` `\frown` `\ge` `\geq` `\geqq` `\geqslant` `\gg` `\ggg` `\gggtr` `\gnapprox` `\gneq` `\gneqq` `\gnsim` `\gt` `\gtrapprox` `\gtrdot` `\gtreqless` `\gtreqqless` `\gtrless` `\gtrsim` `\gvertneqq` `\in` `\isin` `\le` `\leq` `\leqq` `\leqslant` `\lessapprox` `\lessdot` `\lesseqgtr` `\lesseqqgtr` `\lessgtr` `\lesssim` `\ll` `\lll` `\llless` `\lnapprox` `\lneq` `\lneqq` `\lnsim` `\lt` `\lvertneqq` `\mid` `\minuscolon` `\minuscoloncolon` `\ncong` `\ne` `\neq` `\ngeq` `\ngeqq` `\ngeqslant` `\ngtr` `\ni` `\nleq` `\nleqq` `\nleqslant` `\nless` `\nmid` `\notin` `\notni` `\nparallel` `\nprec` `\npreceq` `\nshortparallel` `\nsim` `\nsubseteq` `\nsubseteqq` `\nsucc` `\nsucceq` `\nsupseteq` `\nsupseteqq` `\ntriangleleft` `\ntrianglelefteq` `\ntriangleright` `\ntrianglerighteq` `\nvdash` `\nVdash` `\nvDash` `\nVDash` `\ordinarycolon` `\owns` `\parallel` `\perp` `\pitchfork` `\prec` `\precapprox` `\preccurlyeq` `\preceq` `\precnapprox` `\precneqq` `\precnsim` `\precsim` `\propto` `\questeq` `\risingdotseq` `\shortmid` `\shortparallel` `\sim` `\simcolon` `\simcoloncolon` `\simeq` `\smallfrown` `\smallsmile` `\smile` `\sqsubset` `\sqsubseteq` `\sqsupset` `\sqsupseteq` `\subset` `\subseteq` `\subseteqq` `\subsetneq` `\subsetneqq` `\succ` `\succapprox` `\succcurlyeq` `\succeq` `\succnapprox` `\succneqq` `\succnsim` `\succsim` `\supset` `\supseteq` `\supseteqq` `\supsetneq` `\supsetneqq` `\therefore` `\thickapprox` `\thicksim` `\trianglelefteq` `\triangleq` `\trianglerighteq` `\varpropto` `\varsubsetneq` `\varsubsetneqq` `\varsupsetneq` `\varsupsetneqq` `\vartriangle` `\vartriangleleft` `\vartriangleright` `\vcentcolon` `\vDash` `\vdash` `\Vdash` `\Vvdash` `\xleftequilibrium` `\xlongequal` `\xrightequilibrium`

**二元运算符**（55）：`\amalg` `\And` `\ast` `\barwedge` `\bigtriangledown` `\bigtriangleup` `\boxdot` `\boxminus` `\boxplus` `\boxtimes` `\bullet` `\cap` `\Cap` `\cdot` `\centerdot` `\circ` `\circledast` `\circledcirc` `\circleddash` `\cup` `\Cup` `\dagger` `\diamond` `\div` `\divideontimes` `\dotplus` `\doublecap` `\doublecup` `\intercal` `\lhd` `\ltimes` `\mp` `\odot` `\ominus` `\oplus` `\oslash` `\otimes` `\pm` `\rhd` `\rtimes` `\setminus` `\smallsetminus` `\sqcap` `\sqcup` `\star` `\times` `\triangleleft` `\triangleright` `\unlhd` `\unrhd` `\uplus` `\vee` `\veebar` `\wedge` `\wr`

**定界符**（59）：`\angl` `\angln` `\Arrowvert` `\arrowvert` `\backslash` `\bangle` `\big` `\Big` `\Bigg` `\bigg` `\Biggl` `\biggl` `\biggm` `\Biggm` `\Biggr` `\biggr` `\bigl` `\Bigl` `\Bigm` `\bigm` `\Bigr` `\bigr` `\brace` `\brack` `\lang` `\langle` `\lBrace` `\lbrace` `\lbrack` `\lceil` `\left` `\lfloor` `\lgroup` `\llbracket` `\llcorner` `\lmoustache` `\lparen` `\lrcorner` `\lVert` `\lvert` `\middle` `\rang` `\rangle` `\rBrace` `\rbrace` `\rbrack` `\rceil` `\rfloor` `\rgroup` `\right` `\rmoustache` `\rparen` `\rrbracket` `\rVert` `\rvert` `\ulcorner` `\urcorner` `\vert` `\Vert`

**重音与装饰**（53）：`\acute` `\backprime` `\bar` `\bcancel` `\breve` `\cancel` `\cdotp` `\cdots` `\check` `\clap` `\ddot` `\ddots` `\dot` `\dotminus` `\dots` `\dotsb` `\dotsc` `\dotsi` `\dotsm` `\dotso` `\grave` `\hat` `\iddots` `\ldotp` `\ldots` `\llap` `\mathclap` `\mathllap` `\mathring` `\mathrlap` `\overbrack` `\overgroup` `\overline` `\overlinesegment` `\prime` `\rlap` `\sdot` `\sout` `\tilde` `\tripledash` `\underbar` `\underbrack` `\undergroup` `\underline` `\underlinesegment` `\utilde` `\varvdots` `\vdots` `\vec` `\widecheck` `\widehat` `\widetilde` `\xcancel`

**字体与文本**（37）：`\Bbb` `\Bbbk` `\bf` `\bm` `\bold` `\boldsymbol` `\cal` `\emph` `\frak` `\it` `\mathbb` `\mathbf` `\mathcal` `\mathds` `\mathellipsis` `\mathfrak` `\mathit` `\mathnormal` `\mathrm` `\mathscr` `\mathsf` `\mathsfit` `\mathsterling` `\mathtt` `\oldstylenums` `\rm` `\sf` `\text` `\textbf` `\textit` `\textmd` `\textnormal` `\textrm` `\textsf` `\texttt` `\textup` `\tt`

**文本符号**（52）：`\aa` `\AA` `\AE` `\ae` `\c` `\H` `\i` `\j` `\lq` `\N` `\o` `\O` `\OE` `\oe` `\P` `\R` `\r` `\rq` `\S` `\ss` `\textasciicircum` `\textasciitilde` `\textbackslash` `\textbar` `\textbardbl` `\textbraceleft` `\textbraceright` `\textcircled` `\textcolor` `\textcopyright` `\textdagger` `\textdaggerdbl` `\textdegree` `\textdollar` `\textellipsis` `\textemdash` `\textendash` `\textgreater` `\textless` `\textquotedblleft` `\textquotedblright` `\textquoteleft` `\textquoteright` `\textregistered` `\textsterling` `\textstyle` `\textsubscript` `\textsuperscript` `\textunderscore` `\u` `\v` `\Z`

**颜色**（6）：`\bgcolor` `\color` `\colorbox` `\definecolor` `\fcolorbox` `\fgcolor`

**颜色名**（103）：`\apricot` `\aquamarine` `\bittersweet` `\blue` `\blueA` `\blueB` `\blueC` `\blueD` `\blueE` `\bluegreen` `\blueviolet` `\brown` `\burntorange` `\cadetblue` `\carnationpink` `\cornflowerblue` `\dandelion` `\darkorchid` `\emerald` `\forestgreen` `\fuchsia` `\goldA` `\goldB` `\goldC` `\goldD` `\goldE` `\goldenrod` `\gray` `\grayA` `\grayB` `\grayC` `\grayD` `\grayE` `\grayF` `\grayG` `\grayH` `\grayI` `\green` `\greenA` `\greenB` `\greenC` `\greenD` `\greenE` `\greenyellow` `\junglegreen` `\kaBlue` `\kaGreen` `\limegreen` `\magenta` `\maroon` `\maroonA` `\maroonB` `\maroonC` `\maroonD` `\maroonE` `\melon` `\midnightblue` `\mintA` `\mintB` `\mintC` `\olivegreen` `\orange` `\orangered` `\peach` `\periwinkle` `\pinegreen` `\pink` `\processblue` `\purple` `\purpleA` `\purpleB` `\purpleC` `\purpleD` `\purpleE` `\rawsienna` `\red` `\redA` `\redB` `\redC` `\redD` `\redE` `\redorange` `\redviolet` `\rhodamine` `\royalblue` `\royalpurple` `\rubinered` `\salmon` `\sepia` `\skyblue` `\springgreen` `\tan` `\tealA` `\tealB` `\tealC` `\tealD` `\tealE` `\thistle` `\turquoise` `\violetred` `\wildstrawberry` `\yellowgreen` `\yelloworange`

**间距与占位**（27）：`\allowbreak` `\cr` `\enskip` `\enspace` `\hphantom` `\hskip` `\hspace` `\kern` `\mathstrut` `\medspace` `\mkern` `\mskip` `\negmedspace` `\negthickspace` `\negthinspace` `\newline` `\nobreakspace` `\phantom` `\qquad` `\quad` `\smash` `\space` `\thickspace` `\thinspace` `\tmspace` `\vphantom` `\vspace`

**盒子与图形 (WASM)**（13）：`\boxbox` `\boxonbox` `\doublebox` `\fbox` `\includegraphics` `\mbox` `\ovalbox` `\raisebox` `\reflectbox` `\resizebox` `\rotatebox` `\scalebox` `\shadowbox`

**表格 (WASM)**（13）：`\arrayrulecolor` `\arraystretch` `\cellcolor` `\columncolor` `\cornersize` `\hdots` `\hdotsfor` `\hline` `\multicolumn` `\multirow` `\newcolumntype` `\rowcolor` `\tabular`

**显示样式**（14）：`\displaystyle` `\footnotesize` `\Huge` `\huge` `\LARGE` `\large` `\Large` `\normalsize` `\scriptscriptstyle` `\scriptsize` `\scriptstyle` `\sixptsize` `\small` `\tiny`

**TeX 原语与语法**（52）：`\above` `\abovewithdelims` `\atop` `\atopwithdelims` `\begin` `\begingroup` `\bgroup` `\breakEverywhere` `\char` `\choose` `\def` `\edef` `\egroup` `\end` `\endgroup` `\errmessage` `\expandafter` `\fatalIfCmdConflict` `\futurelet` `\gdef` `\global` `\hbox` `\href` `\htmlClass` `\htmlData` `\htmlId` `\htmlStyle` `\let` `\limits` `\magnification` `\makeatletter` `\makeatother` `\message` `\newcommand` `\nobreak` `\noexpand` `\nolimits` `\nonumber` `\notag` `\over` `\overwithdelims` `\providecommand` `\relax` `\renewcommand` `\renewenvironment` `\show` `\tag` `\TextOrMath` `\url` `\vcenter` `\verb` `\xdef`

**杂项符号**（114）：`\alef` `\alefsym` `\aleph` `\angle` `\beth` `\bigcirc` `\bigstar` `\blacklozenge` `\blacksquare` `\blacktriangle` `\blacktriangledown` `\blacktriangleleft` `\blacktriangleright` `\block` `\bot` `\Box` `\bull` `\checkmark` `\clubs` `\clubsuit` `\cnums` `\complement` `\Complex` `\copyright` `\curlyvee` `\curlywedge` `\dag` `\Dagger` `\daleth` `\ddag` `\ddagger` `\ddddot` `\dddot` `\diagdown` `\diagup` `\Diamond` `\diamonds` `\diamondsuit` `\DOTSB` `\DOTSI` `\DOTSX` `\dotsx` `\doublebarwedge` `\ell` `\empty` `\emptyset` `\eth` `\exist` `\exists` `\Finv` `\flat` `\forall` `\Game` `\gimel` `\greek` `\hbar` `\hearts` `\heartsuit` `\hslash` `\hybull` `\Im` `\image` `\imageof` `\imath` `\infin` `\infty` `\jmath` `\Join` `\KaTeX` `\land` `\LaTeX` `\lhblk` `\lnot` `\lor` `\maltese` `\marker` `\measuredangle` `\mho` `\micro` `\minuso` `\multimap` `\nabla` `\natnums` `\natural` `\neg` `\nexists` `\origin` `\origof` `\partial` `\phase` `\plusmn` `\pmb` `\pod` `\pounds` `\ratio` `\Re` `\real` `\Reals` `\sect` `\sharp` `\spades` `\spadesuit` `\sphericalangle` `\sqrtsign` `\surd` `\TeX` `\top` `\triangle` `\triangledown` `\uhblk` `\varnothing` `\weierp` `\wp` `\yen`

**其他**（40）：`\Android` `\AndroidTeX` `\boxed` `\ca` `\ce` `\celsius` `\ch` `\circledR` `\circledS` `\degree` `\hdashline` `\hermitmatrix` `\leftthreetimes` `\long` `\lozenge` `\not` `\nshortmid` `\Omicron` `\omicron` `\operatornamewithlimits` `\pica` `\pix` `\pixel` `\pu` `\reals` `\restriction` `\rightthreetimes` `\Roman` `\roman` `\rule` `\sh` `\square` `\sub` `\sube` `\Subset` `\supe` `\Supset` `\underscore` `\varlimsup` `\x`

## 彩蛋提示

- dvips 色名（WASM 层）：`wildstrawberry` `periwinkle` `burntorange` `cadetblue` `royalpurple` 等 30+，用 `\textcolor{名字}{...}` 触发。
- 表格玩法（WASM 层）：`\begin{tabular}` + `\multirow` `\multicolumn` `\cellcolor` `\rowcolor` `\hdotsfor`。
- `\DeclareMathOperator{\op}{Op}` 自定义算子（WASM 层）。
- KaTeX 内部命令（`\@ifstar` 等 30 余个）未列入，正常用法用不到。

## 文件

| 文件 | 内容 |
|---|---|
| `index.html` | 可搜索 / 按层筛选 / 点击复制的在线速查表 |
| `deepseek-latex-cheatsheet.md` | 纯清单版，适合直接投喂给 DeepSeek |
| `latex_support_analysis.md` | 两层渲染路径分析报告（证据链） |
| `data/katex_cmds.txt` | KaTeX 层全量 1059 个命令字面量 |
| `data/wasm_only.txt` | WASM 层原始候选（含 C++ 噪音，仅深挖用） |

## 方法

浏览器 view-source 保存首页 → 剥离 view-source 包装还原源码 → 提取 bundle 清单并下载 → `main.js` 定位渲染逻辑与 chunk 映射 → katex chunk（0.16.22）提取命令字面量 → microtex-wasm 二进制提取字符串 → 与 [MicroTeX](https://github.com/NanoMichael/MicroTeX) 官方源码词汇交叉验证，把 3798 个噪音候选洗出可信命令集。

## 注意

静态分析快照，DeepSeek 更新前端后可能漂移；个别条目实测无效属构建版本差异。判定基于字符串交叉验证，存在固有误差（`data/wasm_only.txt` 未洗的原始数据仅供深挖）。

## 法律与合规声明

- 本仓库为**非官方社区项目**，与 DeepSeek（杭州深度求索）无任何隶属、合作或认可关系。
- 全部分析基于**浏览器公开可获取的前端资源**（任何访客加载页面时都会自动下载的静态文件），未绕过任何访问控制或技术保护措施，未收集任何用户数据。
- 命令支持清单是对 **KaTeX（MIT License）** 与 **MicroTeX（MIT License）** 两个开源引擎行为的事实性记录；命令名为 LaTeX 通用语法接口，属功能性事实，不受著作权保护。
- 仓库内不含 DeepSeek 的任何源代码或二进制文件，所有文件均为本项目自产的分析产物。
- 仅供学习研究用途；如权利人认为不妥请提 issue 联系，将配合处理（改名/下架）。
