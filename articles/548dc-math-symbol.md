---
title: "圏論、集合論で使われる記号の読み方とTeXでの表記"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tex", "圏論", "集合論"]
published: true
---

圏論や集合論で使われる記号の読み方とTeXでの表記をまとめました。

## 圏論で使われる記号一覧

### 射・矢印

| 記号 | 読み方 | TeX |
|---|---|---|
| → | 矢印、射(morphism) | `\to`, `\rightarrow` |
| ⟶ | 長い矢印 | `\longrightarrow` |
| ← | 逆向き矢印 | `\leftarrow`, `\gets` |
| ↦ | マップスツー、〜を〜に写す | `\mapsto` |
| ⟼ | 長いマップスツー | `\longmapsto` |
| ⇒ | 二重矢印、自然変換 | `\Rightarrow` |
| ⟹ | 長い二重矢印 | `\Longrightarrow` |
| ↪ | フック矢印、モノ射(単射) | `\hookrightarrow` |
| ↣ | 尾付き矢印、モノ射 | `\rightarrowtail` |
| ↠ | 二重頭矢印、エピ射(全射) | `\twoheadrightarrow` |
| ⇄ | 随伴ペアの矢印 | `\rightleftarrows` |
| ⇢ | 破線矢印(普遍性で使用) | `\dashrightarrow` |

### 等号・同型関係

| 記号 | 読み方 | TeX |
|---|---|---|
| = | イコール、等しい | `=` |
| ≅ | 同型(isomorphic) | `\cong` |
| ≃ | 同値、ホモトピー同値 | `\simeq` |
| ∼ | チルダ、ニョロ、相似 | `\sim` |
| ≈ | ニアリーイコール | `\approx` |
| ≡ | 合同、定義 | `\equiv` |
| ≔ | コロンイコール、定義 | `\coloneqq`(amsmath) |

### 演算

| 記号 | 読み方 | TeX |
|---|---|---|
| ∘ | 合成(composition)、丸 | `\circ` |
| ⊗ | テンソル積 | `\otimes` |
| ⊕ | 直和 | `\oplus` |
| × | 積、デカルト積 | `\times` |
| ∐ | 余積(coproduct)、アマルガム | `\coprod`, `\amalg` |
| ∏ | 直積、プロダクト | `\prod` |
| ⨿ | 双対直和、コプロダクト | `\amalg` |
| ⋆ | スター積 | `\star` |
| ⋅ | ドット、点 | `\cdot` |

### 随伴・関係

| 記号 | 読み方 | TeX |
|---|---|---|
| ⊣ | 随伴(adjoint)、左随伴 | `\dashv` |
| ⊢ | ターンスタイル、右随伴 | `\vdash` |
| ⊥ | パープ、ボトム、随伴記号 | `\perp`, `\bot` |
| ⊤ | トップ | `\top` |

### 極限と余極限

| 記号 | 読み方 | TeX |
|---|---|---|
| lim | リミット、極限 | `\lim` |
| colim | コリミット、余極限 | `\mathrm{colim}`, `\operatorname{colim}` |
| ⟵lim | 射影極限(逆極限) | `\varprojlim` |
| ⟶lim | 順極限(直極限) | `\varinjlim` |
| ⌟ | プルバック角 | `\lrcorner` |
| ⌜ | プッシュアウト角 | `\ulcorner` |

#### 特殊な対象・圏

| 記号 | 読み方 | TeX |
|---|---|---|
| 𝟏 | 終対象(terminal object) | `\mathbf{1}` |
| 𝟎 | 始対象(initial object) | `\mathbf{0}` |
| ∅ | 空集合、空 | `\emptyset`, `\varnothing` |
| 𝒞, 𝒟 | カリグラフィー体の圏 | `\mathcal{C}`, `\mathcal{D}` |
| 𝐂 | 太字の圏 | `\mathbf{C}` |
| Set | 集合の圏 | `\mathbf{Set}`, `\mathrm{Set}` |
| Top | 位相空間の圏 | `\mathbf{Top}` |
| Grp | 群の圏 | `\mathbf{Grp}` |
| Ab | アーベル群の圏 | `\mathbf{Ab}` |
| Ring | 環の圏 | `\mathbf{Ring}` |
| Mod | 加群の圏 | `\mathbf{Mod}`, `R\text{-}\mathbf{Mod}` |
| Cat | 圏の圏 | `\mathbf{Cat}` |
| Vect | ベクトル空間の圏 | `\mathbf{Vect}` |
| 𝒞ᵒᵖ | 反対圏(opposite category) | `\mathcal{C}^{\mathrm{op}}` |

### Hom集合・対象

| 記号 | 読み方 | TeX |
|---|---|---|
| Hom(A,B) | ホム集合 | `\mathrm{Hom}(A,B)`, `\operatorname{Hom}` |
| Mor(A,B) | 射の集合 | `\mathrm{Mor}(A,B)` |
| Ob(𝒞) | 対象クラス | `\mathrm{Ob}(\mathcal{C})` |
| End(A) | 自己準同型環 | `\mathrm{End}(A)` |
| Aut(A) | 自己同型群 | `\mathrm{Aut}(A)` |
| id, 1 | 恒等射(identity) | `\mathrm{id}`, `1_X` |

### 関手・自然変換でよく使うギリシャ文字

| 記号 | 読み方 | TeX |
|---|---|---|
| η | イータ(単位 unit) | `\eta` |
| ε | イプシロン(余単位 counit) | `\varepsilon`, `\epsilon` |
| μ | ミュー(乗法 multiplication) | `\mu` |
| α | アルファ(結合子 associator) | `\alpha` |
| λ | ラムダ(左単位子) | `\lambda` |
| ρ | ロー(右単位子) | `\rho` |
| Δ | デルタ(対角関手) | `\Delta` |
| Σ | シグマ(総和、添字付き余積) | `\Sigma` |

### その他よく使う記号

| 記号 | 読み方 | TeX |
|---|---|---|
| ∈ | 元、属する | `\in` |
| ∋ | を含む | `\ni` |
| ⊂, ⊆ | 部分集合 | `\subset`, `\subseteq` |
| ∀ | 任意の(forall) | `\forall` |
| ∃ | 存在する(exists) | `\exists` |
| ∃! | 一意に存在 | `\exists!` |
| □ | 証明終わり、四角 | `\square`, `\Box` |
| ↯ | 矛盾 | `\lightning`(stmaryrd) |

## 集合論で使われる記号一覧

### 基本記号

| 記号 | 読み方 | TeX |
|---|---|---|
| ∈ | 元、属する、イン | `\in` |
| ∉ | 属さない、ノットイン | `\notin`, `\not\in` |
| ∋ | を含む | `\ni` |
| ∌ | を含まない | `\not\ni`, `\notni` |
| { } | 波括弧、集合 | `\{ \}`, `\{x \mid P(x)\}` |
| ∅ | 空集合、エンプティ | `\emptyset`, `\varnothing` |
| \| | 縦棒、〜なる、such that | `\mid`, `\vert` |
| : | コロン、〜なる | `:` |

### 包含関係

| 記号 | 読み方 | TeX |
|---|---|---|
| ⊂ | 部分集合(真部分集合の意で使うことも) | `\subset` |
| ⊆ | 部分集合(または等しい) | `\subseteq` |
| ⊊ | 真部分集合 | `\subsetneq` |
| ⊃ | 上位集合 | `\supset` |
| ⊇ | 上位集合(または等しい) | `\supseteq` |
| ⊋ | 真上位集合 | `\supsetneq` |
| ⊄ | 部分集合でない | `\not\subset`, `\nsubseteq` |

### 集合演算

| 記号 | 読み方 | TeX |
|---|---|---|
| ∪ | 和集合、ユニオン、カップ | `\cup` |
| ∩ | 共通部分、インターセクション、キャップ | `\cap` |
| ∖ | 差集合、セットマイナス | `\setminus` |
| ∁, ᶜ | 補集合(complement) | `\complement`, `A^c`, `\overline{A}` |
| × | 直積、デカルト積、クロス | `\times` |
| ⊕ | 対称差(または直和) | `\oplus`, `\triangle` |
| △ | 対称差 | `\triangle`, `\bigtriangleup` |
| ⨆ | 非交和、ディスジョイントユニオン | `\sqcup`, `\bigsqcup` |
| ⋃ | 大ユニオン、合併 | `\bigcup` |
| ⋂ | 大インターセクション、共通部分 | `\bigcap` |
| ∏ | 直積 | `\prod` |
| ∐ | 余積、コプロダクト | `\coprod` |

### 順序対・写像

| 記号 | 読み方 | TeX |
|---|---|---|
| (a,b) | 順序対、ペア | `(a,b)` |
| ⟨a,b⟩ | 山括弧の順序対 | `\langle a,b \rangle` |
| → | 矢印、写像 | `\to`, `\rightarrow` |
| ↦ | マップスツー、〜を〜に写す | `\mapsto` |
| ↪ | 単射(injection)、フック矢印 | `\hookrightarrow` |
| ↠ | 全射(surjection)、二重頭矢印 | `\twoheadrightarrow` |
| ≡>, ⤖ | 全単射 | `\rightarrowtail` 等の組合せ |
| ∘ | 合成、丸 | `\circ` |
| f⁻¹ | エフ・インバース、逆写像 | `f^{-1}` |
| f\|_A | エフ・レストリクト・トゥ・A、制限 | `f|_A`, `f\restriction A` |
| f[A] | 像(image) | `f[A]`, `f(A)` |
| Im, Ran | 像、値域 | `\mathrm{Im}`, `\mathrm{Ran}` |
| Dom | 定義域、ドメイン | `\mathrm{Dom}`, `\mathrm{dom}` |

### 数の集合

| 記号 | 読み方 | TeX |
|---|---|---|
| ℕ | 自然数全体、エヌ | `\mathbb{N}` |
| ℤ | 整数全体、ゼット | `\mathbb{Z}` |
| ℚ | 有理数全体、キュー | `\mathbb{Q}` |
| ℝ | 実数全体、アール | `\mathbb{R}` |
| ℂ | 複素数全体、シー | `\mathbb{C}` |
| ω | オメガ、最小無限順序数 | `\omega` |

### 濃度・順序数

| 記号 | 読み方 | TeX |
|---|---|---|
| \|A\| | 濃度、絶対値 | `|A|`, `\lvert A \rvert` |
| #A | 濃度、シャープ | `\#A` |
| card(A) | カーディナリティ、濃度 | `\mathrm{card}(A)` |
| ℵ | アレフ | `\aleph` |
| ℵ₀ | アレフゼロ、アレフヌル(可算濃度) | `\aleph_0` |
| ℵ₁ | アレフワン | `\aleph_1` |
| ℶ | ベート | `\beth` |
| ℶ₀ | ベートゼロ | `\beth_0` |
| 𝔠 | 連続体濃度、シー | `\mathfrak{c}` |
| 2^ℵ₀ | ツー・アレフゼロ、連続体濃度 | `2^{\aleph_0}` |
| ω₁ | オメガワン、最小非可算順序数 | `\omega_1` |
| ord(A) | 順序数、オーディナル | `\mathrm{ord}(A)` |
| Ord | 順序数のクラス | `\mathrm{Ord}` |
| Card | 濃度のクラス | `\mathrm{Card}` |

### 同値・順序関係

| 記号 | 読み方 | TeX |
|---|---|---|
| ∼ | チルダ、ニョロ、同値 | `\sim` |
| ≈ | ニアリーイコール、近似 | `\approx` |
| ≃ | 同値、ホモトピー同値 | `\simeq` |
| ≅ | 同型、合同 | `\cong` |
| ≡ | 合同、定義 | `\equiv` |
| ≤ | 以下、ロー・オア・イコール | `\leq`, `\le` |
| ≥ | 以上、グレーター・オア・イコール | `\geq`, `\ge` |
| ≺ | 先行、プリシード | `\prec` |
| ≼ | 先行または等しい | `\preceq` |
| ⊑ | スクエア・サブセット・イコール | `\sqsubseteq` |
| < | 小なり、レス・ザン | `<` |
| > | 大なり、グレーター・ザン | `>` |

### 論理記号(集合論で頻出)

| 記号 | 読み方 | TeX |
|---|---|---|
| ∧ | かつ、ウェッジ、アンド | `\wedge`, `\land` |
| ∨ | または、ヴィー、オア | `\vee`, `\lor` |
| ¬ | 否定、ノット | `\neg`, `\lnot` |
| ⇒ | ならば、インプライ | `\Rightarrow`, `\implies` |
| ⇔ | 必要十分、同値 | `\Leftrightarrow`, `\iff` |
| ∀ | 任意の、フォーオール | `\forall` |
| ∃ | 存在する、エグジスト | `\exists` |
| ∃! | 一意に存在 | `\exists!` |
| ⊢ | ターンスタイル、証明可能 | `\vdash` |
| ⊨ | ダブルターンスタイル、モデル、充足 | `\models`, `\vDash` |
| ≔, := | 定義する、コロンイコール | `\coloneqq`, `:=` |

### 冪集合・関連集合

| 記号 | 読み方 | TeX |
|---|---|---|
| 𝒫(A), P(A) | 冪集合、パワーセット | `\mathcal{P}(A)`, `\mathfrak{P}(A)` |
| 2^A | ツー・エー、冪集合 | `2^A` |
| A^B | エー・ビー乗、関数集合 | `A^B` |
| [A]^n | エー・ノ・エヌ・ぶぶん、n元部分集合全体 | `[A]^n` |
| [A]^{<ω} | 有限部分集合全体 | `[A]^{<\omega}` |
| A/∼ | 商集合、エー・モッド・チルダ | `A/{\sim}` |

### 公理的集合論

| 記号 | 読み方 | TeX |
|---|---|---|
| ZF | ゼット・エフ、ツェルメロ–フレンケル | `\mathrm{ZF}` |
| ZFC | ゼット・エフ・シー(選択公理付) | `\mathrm{ZFC}` |
| AC | 選択公理(Axiom of Choice) | `\mathrm{AC}` |
| CH | 連続体仮説 | `\mathrm{CH}` |
| GCH | 一般連続体仮説 | `\mathrm{GCH}` |
| V | 累積階層全体、フォン・ノイマン宇宙 | `V`, `\mathbf{V}` |
| L | 構成可能宇宙、エル | `L`, `\mathbf{L}` |
| V_α | ブイ・アルファ、累積階層 | `V_\alpha` |
| L_α | エル・アルファ | `L_\alpha` |
| HF | 遺伝的有限集合 | `\mathrm{HF}` |
| Vᴹ | 内部モデル | `V^M` |
| Con(T) | 無矛盾性 | `\mathrm{Con}(T)` |

### 強制法・モデル理論

| 記号 | 読み方 | TeX |
|---|---|---|
| ⊩ | フォース、強制 | `\Vdash` |
| ⊮ | 強制しない | `\nVdash` |
| ≼, ⪯ | 初等的部分構造 | `\preceq`, `\preccurlyeq` |
| M[G] | エム・ブラケット・ジー、ジェネリック拡大 | `M[G]` |
| ⌜φ⌝ | 引用、ゲーデル数 | `\ulcorner \varphi \urcorner` |

### その他の記号

| 記号 | 読み方 | TeX |
|---|---|---|
| ∞ | 無限大、インフィニティ | `\infty` |
| □ | 証明終わり、四角 | `\square`, `\Box` |
| ⨯, ⊓ | 並 | `\sqcap` |
| ↾ | 制限、レストリクション | `\restriction`, `\upharpoonright` |
| ⊆ᶠⁱⁿ | 有限部分集合 | `\subseteq_{\mathrm{fin}}` |

