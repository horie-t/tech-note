---
title: "線形代数学、微分積分学で使われる記号の読み方とTeXでの表記"
emoji: "🐥"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["tex", "線形代数学", "微分積分学"]
published: false
---

線形代数学や微分積分学で使われる記号の読み方とTeXでの表記をまとめました。

# 線形代数学で使われる記号一覧

## ベクトル・行列の表記

| 記号 | 読み方 | TeX |
|---|---|---|
| **v**, $\vec{v}$ | ベクトル・ヴイ、太字またはアロー | `\mathbf{v}`, `\vec{v}`, `\boldsymbol{v}` |
| $\hat{v}$ | ヴイ・ハット、単位ベクトル | `\hat{v}` |
| **0** | ゼロベクトル | `\mathbf{0}`, `\vec{0}` |
| **e**ᵢ | 標準基底ベクトル | `\mathbf{e}_i` |
| A, M | 行列(大文字) | `A`, `M` |
| (aᵢⱼ) | エー・アイ・ジェイ、行列の成分表示 | `(a_{ij})` |
| Aᵀ, Aᵗ | 転置(transpose) | `A^T`, `A^\top`, `A^t` |
| A* | エー・スター、随伴・共役転置 | `A^*`, `A^\ast` |
| A^H, A^† | エルミート共役、ダガー | `A^H`, `A^\dagger` |
| A⁻¹ | 逆行列、エー・インバース | `A^{-1}` |
| $\bar{A}$ | 共役行列、エー・バー | `\bar{A}`, `\overline{A}` |
| A^+ | ムーア・ペンローズ擬似逆行列 | `A^+`, `A^\dagger` |

## 行列演算・特殊な値

| 記号 | 読み方 | TeX |
|---|---|---|
| det(A), \|A\| | 行列式、デターミナント | `\det(A)`, `|A|`, `\lvert A \rvert` |
| tr(A) | トレース、跡 | `\mathrm{tr}(A)`, `\operatorname{tr}` |
| rank(A) | 階数、ランク | `\mathrm{rank}(A)`, `\operatorname{rank}` |
| nullity(A) | 退化次数、ナリティ | `\mathrm{nullity}(A)` |
| dim(V) | 次元、ディメンション | `\dim(V)` |
| I, Iₙ | 単位行列、アイデンティティ | `I`, `I_n`, `\mathbf{I}` |
| O, 0 | 零行列 | `O`, `\mathbf{0}` |
| diag(...) | 対角行列 | `\mathrm{diag}(\ldots)` |
| adj(A) | 余因子行列、アジョイント | `\mathrm{adj}(A)` |
| cof(A) | 余因子 | `\mathrm{cof}(A)` |

## 内積・ノルム

| 記号 | 読み方 | TeX |
|---|---|---|
| ⟨u, v⟩ | 内積、インナープロダクト | `\langle u, v \rangle` |
| (u, v) | 内積 | `(u, v)` |
| u · v | ドット積、内積 | `u \cdot v` |
| u × v | クロス積、外積 | `u \times v` |
| u ⊗ v | テンソル積 | `u \otimes v` |
| u ∧ v | 楔積、ウェッジ積 | `u \wedge v` |
| ‖v‖ | ノルム、ヴイのノルム | `\|v\|`, `\lVert v \rVert` |
| ‖v‖₂ | ユークリッドノルム、L²ノルム | `\|v\|_2` |
| ‖v‖∞ | 無限大ノルム、最大値ノルム | `\|v\|_\infty` |
| ‖A‖_F | フロベニウスノルム | `\|A\|_F` |
| ⊥ | 直交、垂直 | `\perp` |

## 写像・空間

| 記号 | 読み方 | TeX |
|---|---|---|
| ℝⁿ | アール・エヌ、n次元実空間 | `\mathbb{R}^n` |
| ℂⁿ | シー・エヌ、n次元複素空間 | `\mathbb{C}^n` |
| V, W | ベクトル空間 | `V`, `W` |
| V* | ヴイ・スター、双対空間 | `V^*`, `V^\ast` |
| V** | 二重双対空間 | `V^{**}` |
| V ⊕ W | 直和 | `V \oplus W` |
| V ⊗ W | テンソル積空間 | `V \otimes W` |
| Ker(T) | カーネル、核 | `\mathrm{Ker}(T)`, `\ker(T)` |
| Im(T) | イメージ、像 | `\mathrm{Im}(T)`, `\operatorname{Im}` |
| Hom(V,W) | 線形写像全体 | `\mathrm{Hom}(V,W)` |
| End(V) | 自己準同型 | `\mathrm{End}(V)` |
| GL(n) | 一般線形群 | `\mathrm{GL}(n)`, `\mathrm{GL}_n` |
| SL(n) | 特殊線形群 | `\mathrm{SL}(n)` |
| O(n) | 直交群 | `\mathrm{O}(n)` |
| SO(n) | 特殊直交群 | `\mathrm{SO}(n)` |
| U(n) | ユニタリ群 | `\mathrm{U}(n)` |
| span | スパン、張る | `\mathrm{span}`, `\operatorname{span}` |

## 固有値・分解

| 記号 | 読み方 | TeX |
|---|---|---|
| λ | ラムダ、固有値 | `\lambda` |
| σ(A) | スペクトル、固有値全体 | `\sigma(A)` |
| ρ(A) | スペクトル半径 | `\rho(A)` |
| Eλ | 固有空間 | `E_\lambda` |
| ⊕ | 直和分解 | `\oplus`, `\bigoplus` |

---

# 微積分学で使われる記号一覧

## 極限・連続

| 記号 | 読み方 | TeX |
|---|---|---|
| lim | リミット、極限 | `\lim` |
| lim sup | リミットスペリオール、上極限 | `\limsup`, `\varlimsup` |
| lim inf | リミットインフィリオール、下極限 | `\liminf`, `\varliminf` |
| → | 〜に近づく | `\to`, `\rightarrow` |
| → | (xが)〜に向かう | `x \to a` |
| → | プラス側から(右極限) | `x \to a^+` |
| → | マイナス側から(左極限) | `x \to a^-` |
| ∞ | 無限大、インフィニティ | `\infty` |
| ε, δ | イプシロン・デルタ | `\varepsilon`, `\delta` |
| sup | スプ、上限 | `\sup` |
| inf | インフ、下限 | `\inf` |
| max, min | マックス、ミン | `\max`, `\min` |
| arg max | アーグマックス | `\arg\max`, `\operatorname*{arg\,max}` |
| arg min | アーグミン | `\arg\min` |

## 微分

| 記号 | 読み方 | TeX |
|---|---|---|
| f'(x) | エフ・プライム、導関数 | `f'(x)` |
| f''(x) | エフ・ダブルプライム | `f''(x)` |
| f^(n) | エフ・カッコ・エヌ、n階導関数 | `f^{(n)}(x)` |
| df/dx | ディー・エフ・ディー・エックス | `\frac{df}{dx}`, `\dfrac{df}{dx}` |
| d²f/dx² | 二階微分 | `\frac{d^2 f}{dx^2}` |
| ∂f/∂x | デル・エフ・デル・エックス、偏微分 | `\frac{\partial f}{\partial x}` |
| ∂ | デル、パーシャル、ラウンドディー | `\partial` |
| ∇ | ナブラ、デル | `\nabla` |
| ∇f | グラディエント、勾配 | `\nabla f`, `\mathrm{grad}\, f` |
| ∇·F | ダイバージェンス、発散 | `\nabla \cdot F`, `\mathrm{div}\, F` |
| ∇×F | ローテーション、回転、カール | `\nabla \times F`, `\mathrm{curl}\, F`, `\mathrm{rot}\, F` |
| Δ, ∇² | ラプラシアン | `\Delta`, `\nabla^2` |
| □ | ダランベルシアン | `\Box`, `\square` |
| Df, Jf | ヤコビ行列 | `Df`, `J_f`, `\mathbf{J}` |
| Hf | ヘッシアン | `Hf`, `\mathbf{H}_f` |
| df | 全微分、外微分 | `df`, `\mathrm{d}f` |
| ḟ, ẍ | エフ・ドット、エックス・ダブルドット(時間微分) | `\dot{f}`, `\ddot{x}` |

## 積分

| 記号 | 読み方 | TeX |
|---|---|---|
| ∫ | インテグラル | `\int` |
| ∫∫ | 二重積分 | `\iint` |
| ∫∫∫ | 三重積分 | `\iiint` |
| ∮ | 周回積分、線積分 | `\oint` |
| ∯ | 閉曲面積分 | `\oiint` |
| ∰ | 閉体積分 | `\oiiint` |
| dx | ディー・エックス、微小量 | `dx`, `\mathrm{d}x`, `\,dx` |
| dμ | ディー・ミュー、測度 | `d\mu` |
| Σ | シグマ、総和 | `\sum` |
| ∏ | プロダクト、総積 | `\prod` |

## 関数・記号

| 記号 | 読み方 | TeX |
|---|---|---|
| exp | エクスポネンシャル、指数 | `\exp` |
| log, ln | ログ、自然対数 | `\log`, `\ln` |
| sin, cos, tan | サイン、コサイン、タンジェント | `\sin`, `\cos`, `\tan` |
| sinh, cosh | ハイパボリックサイン、コサイン | `\sinh`, `\cosh` |
| arcsin, arctan | アークサイン、アークタンジェント | `\arcsin`, `\arctan` |
| ⌊x⌋ | フロア、床関数 | `\lfloor x \rfloor` |
| ⌈x⌉ | シーリング、天井関数 | `\lceil x \rceil` |
| sgn | サイン関数(符号) | `\mathrm{sgn}`, `\operatorname{sgn}` |
| \|x\| | アブソリュート、絶対値 | `|x|`, `\lvert x \rvert` |
| O(·) | ビッグオー、ランダウ記号 | `O(\cdot)`, `\mathcal{O}(\cdot)` |
| o(·) | スモールオー | `o(\cdot)` |
| Θ(·) | シータ | `\Theta(\cdot)` |
| ~ | 漸近的に等しい | `\sim` |
| ≈ | 近似的に等しい | `\approx` |
