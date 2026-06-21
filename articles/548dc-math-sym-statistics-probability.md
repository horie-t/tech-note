---
title: "統計学・確率論で使われる記号の読み方とTeXでの表記"
emoji: "📝"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["tex", "統計学", "確率論"]
published: true
---

統計学や確率論で使われる記号の読み方とTeXでの表記をまとめました。

# 統計学で使われる記号一覧

## 基本統計量

| 記号 | 読み方 | TeX |
|---|---|---|
| $\bar{x}$ | エックス・バー、標本平均 | `\bar{x}`, `\overline{x}` |
| μ | ミュー、母平均 | `\mu` |
| s, s² | エス、標本標準偏差・分散 | `s`, `s^2` |
| σ, σ² | シグマ、母標準偏差・分散 | `\sigma`, `\sigma^2` |
| $\hat{\theta}$ | シータ・ハット、推定量 | `\hat{\theta}` |
| $\tilde{\theta}$ | シータ・チルダ | `\tilde{\theta}` |
| θ* | 真のパラメータ | `\theta^*` |
| n | エヌ、標本サイズ | `n` |
| N | エヌ、母集団サイズ | `N` |
| df, ν | ディーエフ、自由度 | `\mathrm{df}`, `\nu` |
| Md | 中央値、メディアン | `\mathrm{Md}`, `\mathrm{median}` |
| Mo | 最頻値、モード | `\mathrm{Mo}`, `\mathrm{mode}` |
| Q1, Q3 | 第1四分位、第3四分位 | `Q_1`, `Q_3` |
| IQR | 四分位範囲 | `\mathrm{IQR}` |

## 統計量・誤差

| 記号 | 読み方 | TeX |
|---|---|---|
| SE | スタンダードエラー、標準誤差 | `\mathrm{SE}`, `\mathrm{se}` |
| SD | スタンダードデビエーション、標準偏差 | `\mathrm{SD}`, `\mathrm{sd}` |
| CV | 変動係数 | `\mathrm{CV}` |
| MSE | 平均二乗誤差 | `\mathrm{MSE}` |
| RMSE | 平方根平均二乗誤差 | `\mathrm{RMSE}` |
| MAE | 平均絶対誤差 | `\mathrm{MAE}` |
| RSS | 残差平方和 | `\mathrm{RSS}` |
| TSS | 全変動平方和 | `\mathrm{TSS}` |
| R² | アールスクエア、決定係数 | `R^2` |

## 仮説検定・推定

| 記号 | 読み方 | TeX |
|---|---|---|
| H₀ | エイチゼロ、帰無仮説 | `H_0` |
| H₁, Hₐ | 対立仮説 | `H_1`, `H_a` |
| α | アルファ、有意水準、第一種過誤 | `\alpha` |
| β | ベータ、第二種過誤 | `\beta` |
| 1−β | 検出力、パワー | `1-\beta` |
| p, p値 | ピー値 | `p`, `p\text{-value}` |
| z, t, F, χ² | 検定統計量 | `z`, `t`, `F`, `\chi^2` |
| CI | 信頼区間 | `\mathrm{CI}` |
| L(θ) | 尤度、ライクリフッド | `L(\theta)`, `\mathcal{L}(\theta)` |
| ℓ(θ) | 対数尤度 | `\ell(\theta)`, `\log L(\theta)` |
| AIC, BIC | 情報量規準 | `\mathrm{AIC}`, `\mathrm{BIC}` |

## 相関・回帰

| 記号 | 読み方 | TeX |
|---|---|---|
| r | アール、標本相関係数 | `r` |
| ρ | ロー、母相関係数 | `\rho` |
| Cov | コバリアンス、共分散 | `\mathrm{Cov}`, `\operatorname{Cov}` |
| Var | バリアンス、分散 | `\mathrm{Var}`, `\operatorname{Var}` |
| Corr | 相関 | `\mathrm{Corr}`, `\operatorname{Corr}` |
| β, β̂ | 回帰係数 | `\beta`, `\hat{\beta}` |
| ε | イプシロン、誤差項 | `\varepsilon`, `\epsilon` |
| $\hat{y}$ | ワイ・ハット、予測値 | `\hat{y}` |

---

# 確率論で使われる記号一覧

## 確率・事象

| 記号 | 読み方 | TeX |
|---|---|---|
| P, ℙ | ピー、確率 | `P`, `\mathbb{P}`, `\Pr` |
| P(A) | エーの確率 | `P(A)`, `\Pr(A)` |
| P(A\|B) | ビーが与えられたときのエーの条件付き確率 | `P(A \mid B)` |
| Ω | オメガ、標本空間 | `\Omega` |
| ω | スモールオメガ、根元事象 | `\omega` |
| ℱ | エフ、σ-代数 | `\mathcal{F}`, `\mathscr{F}` |
| (Ω, ℱ, P) | 確率空間 | `(\Omega, \mathcal{F}, P)` |
| Aᶜ, A̅ | エー・コンプリメント、余事象 | `A^c`, `\overline{A}`, `A^\complement` |
| ∅ | 空事象 | `\emptyset` |
| ⊥, ⫫ | 独立、インディペンデント | `\perp`, `\perp\!\!\!\perp`, `\independent` |

## 確率変数・分布

| 記号 | 読み方 | TeX |
|---|---|---|
| X, Y, Z | 確率変数(大文字) | `X`, `Y`, `Z` |
| x, y, z | 実現値(小文字) | `x`, `y`, `z` |
| F(x) | エフ・エックス、累積分布関数 | `F(x)`, `F_X(x)` |
| f(x) | エフ・エックス、確率密度関数 | `f(x)`, `f_X(x)` |
| p(x) | ピー・エックス、確率質量関数 | `p(x)`, `p_X(x)` |
| ϕ(x) | ファイ、標準正規密度 | `\phi(x)`, `\varphi(x)` |
| Φ(x) | ファイ、標準正規分布関数 | `\Phi(x)` |
| M_X(t) | モーメント母関数 | `M_X(t)` |
| φ_X(t) | 特性関数 | `\varphi_X(t)` |
| ~ | チルダ、〜に従う | `\sim` |
| X ~ N(μ,σ²) | 正規分布に従う | `X \sim N(\mu, \sigma^2)` |

## 期待値・モーメント

| 記号 | 読み方 | TeX |
|---|---|---|
| E[X], 𝔼[X] | エクスペクテーション、期待値 | `E[X]`, `\mathbb{E}[X]`, `\mathrm{E}[X]` |
| E[X\|Y] | 条件付き期待値 | `E[X \mid Y]` |
| Var(X) | バリアンス、分散 | `\mathrm{Var}(X)`, `V(X)` |
| SD(X) | 標準偏差 | `\mathrm{SD}(X)` |
| Cov(X,Y) | コバリアンス、共分散 | `\mathrm{Cov}(X,Y)` |
| Corr(X,Y) | 相関係数 | `\mathrm{Corr}(X,Y)` |
| μₙ | エヌ次モーメント | `\mu_n` |

## 主な分布

| 記号 | 読み方 | TeX |
|---|---|---|
| N(μ,σ²) | 正規分布(ノーマル) | `N(\mu, \sigma^2)`, `\mathcal{N}(\mu, \sigma^2)` |
| Bin(n,p) | 二項分布 | `\mathrm{Bin}(n,p)`, `B(n,p)` |
| Po(λ), Pois(λ) | ポアソン分布 | `\mathrm{Po}(\lambda)`, `\mathrm{Pois}(\lambda)` |
| Exp(λ) | 指数分布 | `\mathrm{Exp}(\lambda)` |
| U(a,b) | 一様分布、ユニフォーム | `U(a,b)`, `\mathrm{Unif}(a,b)` |
| Geom(p) | 幾何分布 | `\mathrm{Geom}(p)` |
| Γ(α,β) | ガンマ分布 | `\Gamma(\alpha,\beta)` |
| Beta(α,β) | ベータ分布 | `\mathrm{Beta}(\alpha,\beta)` |
| χ²(k) | カイ二乗分布 | `\chi^2(k)`, `\chi^2_k` |
| t(ν) | スチューデントのt分布 | `t(\nu)`, `t_\nu` |
| F(d₁,d₂) | エフ分布 | `F(d_1, d_2)` |

## 収束・極限

| 記号 | 読み方 | TeX |
|---|---|---|
| →ᵃ·ˢ· | ほとんど確実収束 | `\xrightarrow{a.s.}`, `\overset{a.s.}{\to}` |
| →ᴾ | 確率収束 | `\xrightarrow{P}`, `\overset{P}{\to}` |
| →ᵈ | 分布収束、法則収束 | `\xrightarrow{d}`, `\overset{d}{\to}` |
| →ᴸᵖ | Lp収束 | `\xrightarrow{L^p}` |
| =ᵈ | 同分布 | `\overset{d}{=}`, `\stackrel{d}{=}` |
| i.i.d. | アイ・アイ・ディー、独立同分布 | `\mathrm{i.i.d.}`, `\overset{\mathrm{iid}}{\sim}` |
| a.s. | ほとんど確実に | `\mathrm{a.s.}` |
| a.e. | ほとんど至るところで | `\mathrm{a.e.}` |

## 確率過程

| 記号 | 読み方 | TeX |
|---|---|---|
| {Xₜ}, (Xₜ)ₜ≥0 | 確率過程 | `\{X_t\}`, `(X_t)_{t \geq 0}` |
| Bₜ, Wₜ | ブラウン運動、ウィーナー過程 | `B_t`, `W_t` |
| dW, dBₜ | 確率微分 | `dW`, `dB_t` |
| ℱₜ | フィルトレーション | `\mathcal{F}_t` |
| τ | タウ、停止時刻 | `\tau` |
| 𝟙_A, I_A | 指示関数、インディケーター | `\mathbf{1}_A`, `\mathbb{1}_A`, `I_A` |
