# DeepSeek 网页版 LaTeX 命令速查表

> 逆向自 chat.deepseek.com 前端（commit 29e61c85，2026-09）。双层渲染：**KaTeX 0.16.22**（HTML，1020 个命令）为主，**MicroTeX WASM**（SVG）兜底。整条公式中只要有一个命令 KaTeX 不认识，整条就落到 WASM 以 SVG 渲染；两层都不认识则显示原文。

标记说明：**[W]** = 仅 WASM 层支持（KaTeX 会报错，走 SVG 兜底）；无标记 = KaTeX 层直接支持。

## ❌ 实测禁区（静态判定能过，渲染会崩）

以下写法会让整条公式渲染失败——MicroTeX WASM 没实现完整的文本段落断行引擎，一碰就死：

- `\parbox{4cm}{...}` 段落盒
- `\begin{tabular}` 的 `p{4.5cm}` 列类型（改用 `l` / `c` / `r`，或 `*{n}{c}`）
- `\hfill` 在非数学模式下单用

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

**结构**（55）：`\accentset` `\binom` `\Bra` `\bra` `\braket` `\Braket` `\cfrac` `\dbinom` `\dfrac` `\frac` `\genfrac` `\intertext` `\joinrel` `\Ket` `\ket` `\longdiv` `\mathbin` `\mathchoice` `\mathclose` `\mathinner` `\mathop` `\mathopen` `\mathord` `\mathpunct` `\mathrel` `\overbrace` `\overparen` `\overset` `\prescript` `\Set` `\set` `\sfrac` `\shoveleft` `\shoveright` `\sideset` `\spbreve` `\spcheck` `\spddot` `\spdot` `\sphat` `\sptilde` `\sqrt` `\stackbin` `\stackrel` `\substack` `\tbinom` `\tfrac` `\underaccent` `\underbrace` `\underparen` `\underset` `\undertilde` `\xleftarrow` `\xleftrightarrow` `\xrightarrow`

**希腊字母**（66）：`\alpha` `\Alpha` `\beta` `\Beta` `\Chi` `\chi` `\Delta` `\delta` `\digamma` `\Epsilon` `\epsilon` `\Eta` `\eta` `\gamma` `\Gamma` `\Iota` `\iota` `\Kappa` `\kappa` `\Lambda` `\lambda` `\mu` `\Mu` `\Nu` `\nu` `\Omega` `\omega` `\phi` `\Phi` `\pi` `\Pi` `\psi` `\Psi` `\Rho` `\rho` `\Sigma` `\sigma` `\tau` `\Tau` `\Theta` `\theta` `\thetasym` `\upsilon` `\Upsilon` `\varDelta` `\varepsilon` `\varGamma` `\varkappa` `\varLambda` `\varOmega` `\varPhi` `\varphi` `\varPi` `\varpi` `\varPsi` `\varrho` `\varSigma` `\varsigma` `\vartheta` `\varTheta` `\varUpsilon` `\varXi` `\Xi` `\xi` `\Zeta` `\zeta`

**函数与算子**（58）：`\arccos` `\arccot` `\arccsc` `\arcctg` `\arcsec` `\arcsin` `\arctan` `\arctg` `\arg` `\argmax` `\argmin` `\bmod` `\cos` `\cosec` `\cosh` `\cot` `\cotg` `\coth` `\csc` `\csch` `\ctg` `\cth` `\DeclareMathOperator` `\DeclareMathSizes` `\deg` `\det` `\dim` `\exp` `\gcd` `\hom` `\inf` `\injlim` `\ker` `\lg` `\lim` `\liminf` `\limsup` `\ln` `\log` `\max` `\min` `\mod` `\operatorname` `\plim` `\pmod` `\Pr` `\projlim` `\sec` `\sech` `\sin` `\sinh` `\sup` `\tanh` `\tg` `\th` `\varinjlim` `\varliminf` `\varprojlim`

**大型运算符**（22）：`\bigcap` `\bigcup` `\bigodot` `\bigoplus` `\bigotimes` `\bigsqcup` `\biguplus` `\bigvee` `\bigwedge` `\coprod` `\idotsint` `\iiiint` `\iiint` `\iint` `\int` `\intop` `\oiiint` `\oiint` `\oint` `\prod` `\smallint` `\sum`

**箭头**（125）：`\cdleftarrow` `\cdrightarrow` `\circlearrowleft` `\circlearrowright` `\curvearrowleft` `\curvearrowright` `\Darr` `\darr` `\dArr` `\dashleftarrow` `\dashrightarrow` `\downarrow` `\Downarrow` `\downdownarrows` `\downharpoonleft` `\downharpoonright` `\gets` `\hArr` `\Harr` `\harr` `\hookleftarrow` `\hookrightarrow` `\iff` `\impliedby` `\implies` `\Larr` `\lArr` `\larr` `\leadsto` `\Leftarrow` `\leftarrow` `\leftarrowtail` `\leftharpoondown` `\leftharpoonup` `\leftleftarrows` `\Leftrightarrow` `\leftrightarrow` `\leftrightarrows` `\leftrightharpoons` `\leftrightsquigarrow` `\Lleftarrow` `\Longleftarrow` `\longleftarrow` `\longleftrightarrow` `\Longleftrightarrow` `\longmapsfrom` `\Longmapsfrom` `\Longmapsto` `\longmapsto` `\Longrightarrow` `\longrightarrow` `\looparrowleft` `\looparrowright` `\Lrarr` `\lrarr` `\lrArr` `\Lsh` `\Mapsfrom` `\Mapsto` `\mapsto` `\models` `\nearrow` `\nleftarrow` `\nLeftarrow` `\nLeftrightarrow` `\nleftrightarrow` `\nrightarrow` `\nRightarrow` `\nwarrow` `\overleftarrow` `\overleftharpoon` `\overleftrightarrow` `\overrightarrow` `\Overrightarrow` `\overrightharpoon` `\rarr` `\rArr` `\Rarr` `\Relbar` `\relbar` `\Rightarrow` `\rightarrow` `\rightarrowtail` `\rightharpoondown` `\rightharpoonup` `\rightleftarrows` `\rightleftharpoons` `\rightrightarrows` `\rightsquigarrow` `\Rrightarrow` `\Rsh` `\searrow` `\swarrow` `\to` `\twoheadleftarrow` `\twoheadrightarrow` `\Uarr` `\uarr` `\uArr` `\underleftarrow` `\underleftrightarrow` `\underrightarrow` `\Uparrow` `\uparrow` `\Updownarrow` `\updownarrow` `\upharpoonleft` `\upharpoonright` `\upuparrows` `\xhookleftarrow` `\xhookrightarrow` `\xLeftarrow` `\xleftharpoondown` `\xleftharpoonup` `\xLeftrightarrow` `\xleftrightharpoons` `\xmapsto` `\xRightarrow` `\xrightharpoondown` `\xrightharpoonup` `\xrightleftarrows` `\xrightleftharpoons` `\xtofrom` `\xtwoheadleftarrow` `\xtwoheadrightarrow`

**关系符**（198）：`\approx` `\approxcolon` `\approxcoloncolon` `\approxeq` `\asymp` `\backepsilon` `\backsim` `\backsimeq` `\because` `\between` `\bowtie` `\Bumpeq` `\bumpeq` `\cdlongequal` `\circeq` `\colon` `\Colonapprox` `\colonapprox` `\coloncolon` `\coloncolonapprox` `\coloncolonequals` `\coloncolonminus` `\coloncolonsim` `\coloneq` `\Coloneq` `\Coloneqq` `\coloneqq` `\colonequals` `\colonminus` `\colonsim` `\Colonsim` `\cong` `\curlyeqprec` `\curlyeqsucc` `\dashv` `\dblcolon` `\Doteq` `\doteq` `\doteqdot` `\eqcirc` `\Eqcolon` `\eqcolon` `\eqqcolon` `\Eqqcolon` `\eqsim` `\eqslantgtr` `\eqslantless` `\equalscolon` `\equalscoloncolon` `\equiv` `\fallingdotseq` `\frown` `\ge` `\geq` `\geqq` `\geqslant` `\gg` `\ggg` `\gggtr` `\gnapprox` `\gneq` `\gneqq` `\gnsim` `\gt` `\gtrapprox` `\gtrdot` `\gtreqless` `\gtreqqless` `\gtrless` `\gtrsim` `\gvertneqq` `\in` `\isin` `\le` `\leq` `\leqq` `\leqslant` `\lessapprox` `\lessdot` `\lesseqgtr` `\lesseqqgtr` `\lessgtr` `\lesssim` `\ll` `\lll` `\llless` `\lnapprox` `\lneq` `\lneqq` `\lnsim` `\lt` `\lvertneqq` `\mid` `\minuscolon` `\minuscoloncolon` `\ncong` `\ne` `\neq` `\ngeq` `\ngeqq` `\ngeqslant` `\ngtr` `\ni` `\nleq` `\nleqq` `\nleqslant` `\nless` `\nmid` `\notin` `\notni` `\nparallel` `\nprec` `\npreceq` `\nshortparallel` `\nsim` `\nsubseteq` `\nsubseteqq` `\nsucc` `\nsucceq` `\nsupseteq` `\nsupseteqq` `\ntriangleleft` `\ntrianglelefteq` `\ntriangleright` `\ntrianglerighteq` `\nVDash` `\nVdash` `\nvDash` `\nvdash` `\ordinarycolon` `\owns` `\parallel` `\perp` `\pitchfork` `\prec` `\precapprox` `\preccurlyeq` `\preceq` `\precnapprox` `\precneqq` `\precnsim` `\precsim` `\propto` `\questeq` `\risingdotseq` `\shortmid` `\shortparallel` `\sim` `\simcolon` `\simcoloncolon` `\simeq` `\smallfrown` `\smallsmile` `\smile` `\sqsubset` `\sqsubseteq` `\sqsupset` `\sqsupseteq` `\subset` `\subseteq` `\subseteqq` `\subsetneq` `\subsetneqq` `\succ` `\succapprox` `\succcurlyeq` `\succeq` `\succnapprox` `\succneqq` `\succnsim` `\succsim` `\supset` `\supseteq` `\supseteqq` `\supsetneq` `\supsetneqq` `\therefore` `\thickapprox` `\thicksim` `\trianglelefteq` `\triangleq` `\trianglerighteq` `\varpropto` `\varsubsetneq` `\varsubsetneqq` `\varsupsetneq` `\varsupsetneqq` `\vartriangle` `\vartriangleleft` `\vartriangleright` `\vcentcolon` `\Vdash` `\vDash` `\vdash` `\Vvdash` `\xleftequilibrium` `\xlongequal` `\xrightequilibrium`

**二元运算符**（55）：`\amalg` `\And` `\ast` `\barwedge` `\bigtriangledown` `\bigtriangleup` `\boxdot` `\boxminus` `\boxplus` `\boxtimes` `\bullet` `\Cap` `\cap` `\cdot` `\centerdot` `\circ` `\circledast` `\circledcirc` `\circleddash` `\Cup` `\cup` `\dagger` `\diamond` `\div` `\divideontimes` `\dotplus` `\doublecap` `\doublecup` `\intercal` `\lhd` `\ltimes` `\mp` `\odot` `\ominus` `\oplus` `\oslash` `\otimes` `\pm` `\rhd` `\rtimes` `\setminus` `\smallsetminus` `\sqcap` `\sqcup` `\star` `\times` `\triangleleft` `\triangleright` `\unlhd` `\unrhd` `\uplus` `\vee` `\veebar` `\wedge` `\wr`

**定界符**（59）：`\angl` `\angln` `\Arrowvert` `\arrowvert` `\backslash` `\bangle` `\Big` `\big` `\bigg` `\Bigg` `\biggl` `\Biggl` `\biggm` `\Biggm` `\biggr` `\Biggr` `\bigl` `\Bigl` `\Bigm` `\bigm` `\bigr` `\Bigr` `\brace` `\brack` `\lang` `\langle` `\lbrace` `\lBrace` `\lbrack` `\lceil` `\left` `\lfloor` `\lgroup` `\llbracket` `\llcorner` `\lmoustache` `\lparen` `\lrcorner` `\lvert` `\lVert` `\middle` `\rang` `\rangle` `\rbrace` `\rBrace` `\rbrack` `\rceil` `\rfloor` `\rgroup` `\right` `\rmoustache` `\rparen` `\rrbracket` `\rvert` `\rVert` `\ulcorner` `\urcorner` `\vert` `\Vert`

**重音与装饰**（53）：`\acute` `\backprime` `\bar` `\bcancel` `\breve` `\cancel` `\cdotp` `\cdots` `\check` `\clap` `\ddot` `\ddots` `\dot` `\dotminus` `\dots` `\dotsb` `\dotsc` `\dotsi` `\dotsm` `\dotso` `\grave` `\hat` `\iddots` `\ldotp` `\ldots` `\llap` `\mathclap` `\mathllap` `\mathring` `\mathrlap` `\overbrack` `\overgroup` `\overline` `\overlinesegment` `\prime` `\rlap` `\sdot` `\sout` `\tilde` `\tripledash` `\underbar` `\underbrack` `\undergroup` `\underline` `\underlinesegment` `\utilde` `\varvdots` `\vdots` `\vec` `\widecheck` `\widehat` `\widetilde` `\xcancel`

**字体与文本**（37）：`\Bbb` `\Bbbk` `\bf` `\bm` `\bold` `\boldsymbol` `\cal` `\emph` `\frak` `\it` `\mathbb` `\mathbf` `\mathcal` `\mathds` `\mathellipsis` `\mathfrak` `\mathit` `\mathnormal` `\mathrm` `\mathscr` `\mathsf` `\mathsfit` `\mathsterling` `\mathtt` `\oldstylenums` `\rm` `\sf` `\text` `\textbf` `\textit` `\textmd` `\textnormal` `\textrm` `\textsf` `\texttt` `\textup` `\tt`

**文本符号**（52）：`\aa` `\AA` `\ae` `\AE` `\c` `\H` `\i` `\j` `\lq` `\N` `\o` `\O` `\oe` `\OE` `\P` `\R` `\r` `\rq` `\S` `\ss` `\textasciicircum` `\textasciitilde` `\textbackslash` `\textbar` `\textbardbl` `\textbraceleft` `\textbraceright` `\textcircled` `\textcolor` `\textcopyright` `\textdagger` `\textdaggerdbl` `\textdegree` `\textdollar` `\textellipsis` `\textemdash` `\textendash` `\textgreater` `\textless` `\textquotedblleft` `\textquotedblright` `\textquoteleft` `\textquoteright` `\textregistered` `\textsterling` `\textstyle` `\textsubscript` `\textsuperscript` `\textunderscore` `\u` `\v` `\Z`

**颜色**（6）：`\bgcolor` `\color` `\colorbox` `\definecolor` `\fcolorbox` `\fgcolor`

**颜色名**（103）：`\apricot` `\aquamarine` `\bittersweet` `\blue` `\blueA` `\blueB` `\blueC` `\blueD` `\blueE` `\bluegreen` `\blueviolet` `\brown` `\burntorange` `\cadetblue` `\carnationpink` `\cornflowerblue` `\dandelion` `\darkorchid` `\emerald` `\forestgreen` `\fuchsia` `\goldA` `\goldB` `\goldC` `\goldD` `\goldE` `\goldenrod` `\gray` `\grayA` `\grayB` `\grayC` `\grayD` `\grayE` `\grayF` `\grayG` `\grayH` `\grayI` `\green` `\greenA` `\greenB` `\greenC` `\greenD` `\greenE` `\greenyellow` `\junglegreen` `\kaBlue` `\kaGreen` `\limegreen` `\magenta` `\maroon` `\maroonA` `\maroonB` `\maroonC` `\maroonD` `\maroonE` `\melon` `\midnightblue` `\mintA` `\mintB` `\mintC` `\olivegreen` `\orange` `\orangered` `\peach` `\periwinkle` `\pinegreen` `\pink` `\processblue` `\purple` `\purpleA` `\purpleB` `\purpleC` `\purpleD` `\purpleE` `\rawsienna` `\red` `\redA` `\redB` `\redC` `\redD` `\redE` `\redorange` `\redviolet` `\rhodamine` `\royalblue` `\royalpurple` `\rubinered` `\salmon` `\sepia` `\skyblue` `\springgreen` `\tan` `\tealA` `\tealB` `\tealC` `\tealD` `\tealE` `\thistle` `\turquoise` `\violetred` `\wildstrawberry` `\yellowgreen` `\yelloworange`

**间距与占位**（27）：`\allowbreak` `\cr` `\enskip` `\enspace` `\hphantom` `\hskip` `\hspace` `\kern` `\mathstrut` `\medspace` `\mkern` `\mskip` `\negmedspace` `\negthickspace` `\negthinspace` `\newline` `\nobreakspace` `\phantom` `\qquad` `\quad` `\smash` `\space` `\thickspace` `\thinspace` `\tmspace` `\vphantom` `\vspace`

**盒子与图形 (WASM)**（13）：`\boxbox` `\boxonbox` `\doublebox` `\fbox` `\includegraphics` `\mbox` `\ovalbox` `\raisebox` `\reflectbox` `\resizebox` `\rotatebox` `\scalebox` `\shadowbox`

**表格 (WASM)**（13）：`\arrayrulecolor` `\arraystretch` `\cellcolor` `\columncolor` `\cornersize` `\hdots` `\hdotsfor` `\hline` `\multicolumn` `\multirow` `\newcolumntype` `\rowcolor` `\tabular`

**显示样式**（14）：`\displaystyle` `\footnotesize` `\huge` `\Huge` `\Large` `\large` `\LARGE` `\normalsize` `\scriptscriptstyle` `\scriptsize` `\scriptstyle` `\sixptsize` `\small` `\tiny`

**TeX 原语与语法**（52）：`\above` `\abovewithdelims` `\atop` `\atopwithdelims` `\begin` `\begingroup` `\bgroup` `\breakEverywhere` `\char` `\choose` `\def` `\edef` `\egroup` `\end` `\endgroup` `\errmessage` `\expandafter` `\fatalIfCmdConflict` `\futurelet` `\gdef` `\global` `\hbox` `\href` `\htmlClass` `\htmlData` `\htmlId` `\htmlStyle` `\let` `\limits` `\magnification` `\makeatletter` `\makeatother` `\message` `\newcommand` `\nobreak` `\noexpand` `\nolimits` `\nonumber` `\notag` `\over` `\overwithdelims` `\providecommand` `\relax` `\renewcommand` `\renewenvironment` `\show` `\tag` `\TextOrMath` `\url` `\vcenter` `\verb` `\xdef`

**杂项符号**（114）：`\alef` `\alefsym` `\aleph` `\angle` `\beth` `\bigcirc` `\bigstar` `\blacklozenge` `\blacksquare` `\blacktriangle` `\blacktriangledown` `\blacktriangleleft` `\blacktriangleright` `\block` `\bot` `\Box` `\bull` `\checkmark` `\clubs` `\clubsuit` `\cnums` `\complement` `\Complex` `\copyright` `\curlyvee` `\curlywedge` `\dag` `\Dagger` `\daleth` `\ddag` `\ddagger` `\ddddot` `\dddot` `\diagdown` `\diagup` `\Diamond` `\diamonds` `\diamondsuit` `\DOTSB` `\DOTSI` `\dotsx` `\DOTSX` `\doublebarwedge` `\ell` `\empty` `\emptyset` `\eth` `\exist` `\exists` `\Finv` `\flat` `\forall` `\Game` `\gimel` `\greek` `\hbar` `\hearts` `\heartsuit` `\hslash` `\hybull` `\Im` `\image` `\imageof` `\imath` `\infin` `\infty` `\jmath` `\Join` `\KaTeX` `\land` `\LaTeX` `\lhblk` `\lnot` `\lor` `\maltese` `\marker` `\measuredangle` `\mho` `\micro` `\minuso` `\multimap` `\nabla` `\natnums` `\natural` `\neg` `\nexists` `\origin` `\origof` `\partial` `\phase` `\plusmn` `\pmb` `\pod` `\pounds` `\ratio` `\Re` `\real` `\Reals` `\sect` `\sharp` `\spades` `\spadesuit` `\sphericalangle` `\sqrtsign` `\surd` `\TeX` `\top` `\triangle` `\triangledown` `\uhblk` `\varnothing` `\weierp` `\wp` `\yen`

**其他**（40）：`\Android` `\AndroidTeX` `\boxed` `\ca` `\ce` `\celsius` `\ch` `\circledR` `\circledS` `\degree` `\hdashline` `\hermitmatrix` `\leftthreetimes` `\long` `\lozenge` `\not` `\nshortmid` `\Omicron` `\omicron` `\operatornamewithlimits` `\pica` `\pix` `\pixel` `\pu` `\reals` `\restriction` `\rightthreetimes` `\roman` `\Roman` `\rule` `\sh` `\square` `\sub` `\sube` `\Subset` `\supe` `\Supset` `\underscore` `\varlimsup` `\x`

## 彩蛋提示

- dvips 色名（WASM 层）：`wildstrawberry` `periwinkle` `burntorange` `cadetblue` `royalpurple` 等 30+，用 `\textcolor{名字}{...}` 触发。
- 表格玩法（WASM 层）：`\begin{tabular}` + `\multirow` `\multicolumn` `\cellcolor` `\rowcolor` `\hdotsfor`。
- `\DeclareMathOperator{\op}{Op}` 自定义算子（WASM 层）。
- KaTeX 内部命令（`\@ifstar` 等 30 余个）未列入，正常用法用不到。