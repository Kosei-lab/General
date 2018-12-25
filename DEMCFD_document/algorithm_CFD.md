# CFDの概観

## 離散化

CFDには有限要素法、有限体積法、有限差分法、粒子法などいくつかの手法がありますが、本ソルバーでは多くの流体ソルバーと同様に有限体積法を採用しています。
実際の離散化にはスタガード格子を用いています。スタガード格子では各セルから出ていく量や入ってくる量をそのまま変数として置いているので、スタガード格子を用いているということは有限体積法である、と考えてよいです。

## おおまかな流れ

CFDシミュレーションのアルゴリズムは以下のようになっています。

```flow
st=>start: 処理開始

o1=>operation: drag_force
o2=>operation: advection_and_diffusion
o3=>operation: explicit_drag
o4=>operation: explicit_jump
o5=>operation: invert_fluid_diag
o6=>inputoutput: immersed_boundary_condition_velosity(pre)
o7=>subroutine: bound_explicit
o8=>operation: MacSolvePressure
o9=>subroutine: bound_pressure
o10=>operation: update_fluid
o11=>inputoutput: immersed_boundary_condition_velosity(post)
o12=>subroutine: bound_velosity
o13=>inputoutput: immersed_boundary_extend_scalar
o14=>operation: clear_fluid


e=>end: 処理終了

st->o1->o2->o3->o4->o6->o7->o8->o9->o10->o11->o12->o13->o14->e

```

```drag_force```はDEMCFD_dragforce、```advection_and_diffusion```についてはCFD advection and diffusion、平行四辺形型の部分はImmersed Boundaryについての部分なのでCFD_IBM、```MacSolvePressure```はCFD pressure solverにおいて説明します。

その他の部分、(``` explicit_jump```、```update_fluid```)については簡単な部分ですので、この項で述べます。

## フラクショナルステップ法
CFDの支配方程式はナビエ・ストークス方程式と連続の式です。

\[ \frac{\partial \varepsilon}{\partial t}+\nabla\cdot\left(\varepsilon \bm{u}_f\right)=0\]

\[ \frac{\partial \left( \varepsilon \rho_f \bm{u_f}\right)}{\partial t}+\nabla\cdot\left(\varepsilon \rho_f\bm{u}_f\bm{u}_f\right)=-\varepsilon\nabla p-\bm{f_d}+\nabla\cdot\left(\varepsilon \bm{\tau}\right)+\varepsilon\rho_f \bm{g}\]

これ以降の解き方はいくつも存在します(SIMPLE法、MAC法など)が、本ソルバーではフラクショナルステップ法を用いて計算を行います。時間ステップをオイラー陽解法的に離散化すると、

\[ \frac{ \varepsilon^{n+1}-\varepsilon^{n}}{\Delta t}+\nabla\cdot\left(\varepsilon^{n+1} \bm{u}_f^{n+1}\right)=0\]

\[ \rho_f\frac{ \varepsilon^{n+1}\bm{u}_f^{n+1}-\varepsilon^{n}\bm{u}_f^{n}}{\Delta t}+\nabla\cdot\left(\varepsilon^n \rho_f\bm{u}_f^n\bm{u}_f^n\right)=-\varepsilon^{n+1}\nabla p^{n+1}-\bm{f_d}^n+\nabla\cdot\left(\varepsilon^n \bm{\tau}^n\right)+\varepsilon^n\rho_f \bm{g}\]

ナビエ・ストークス方程式を陽的な項と陰的な項に分離します。

\[ \rho_f\frac{ \varepsilon^{n+1}\bm{u}_f^{\ast}-\varepsilon^{n}\bm{u}_f^{n}}{\Delta t}=-\nabla\cdot\left(\varepsilon^n \rho_f\bm{u}_f^n\bm{u}_f^n\right)-\bm{f_d}^n+\nabla\cdot\left(\varepsilon^n \bm{\tau}^n\right)+\varepsilon^n\rho_f \bm{g}\]

\[ \rho_f\frac{ \varepsilon^{n+1}\bm{u}_f^{n+1}-\varepsilon^{n+1}\bm{u}_f^{\ast}}{\Delta t}=-\varepsilon^{n+1}\nabla p^{n+1}\]

上の式から、

\[\bm{u}_f^{\ast}=\frac{\varepsilon^{n}}{\varepsilon^{n+1}}\bm{u}_f^n+\frac{\Delta t}{\rho_f \varepsilon^{n+1}}\left(-\nabla\cdot\left(\varepsilon^n \rho_f\bm{u}_f^n\bm{u}_f^n\right)-\bm{f_d}^n+\nabla\cdot\left(\varepsilon^n \bm{\tau}^n\right)+\varepsilon^n\rho_f \bm{g}\right) -(1)\]

を得ることができます。(TODO 第1項の計算で\( \frac{\varepsilon^{n}}{\varepsilon^{n+1}}\)  はどこで計算されている？)また

下の式と連続の式からポアソン方程式

\[ \frac{\Delta t}{\rho}\nabla\cdot\left( \varepsilon^{n+1}\nabla p^{n+1}\right)=\frac{ \varepsilon^{n+1}-\varepsilon^{n}}{\Delta t}+\nabla\cdot\left(\varepsilon^{n+1} \bm{u}_f^{\ast}\right) -(2)\]

を得ます。上述の2式はその順番に陽に解けますので、最後に

\[ \bm{u}_f^{n+1}=\bm{u}_f^{\ast}-\frac{\Delta t}{\rho_f}\nabla p^{n+1} -(3)\]

から次のステップの速度を得ることができます。

以上がフラクショナルステップ法の導出ですが、計算上は下3式が使用されます。(1)式は```drag_force```と```explicit_drag```(第3項)、```advection_and_diffusion```(第2項、第4項)、``` explicit_jump```(第5項)、(2)式は```MacSolvePressure```、(3)式は```update_fluid```に相当します。

## 空間離散化における符号
スタガード格子ではセル中心の値P(i,j,k)を囲むのがu(i,j,k)とu(i+1,j,k)なのかu(i-1,j,k)とu(i,j,k)なのかという二種類の書き方がありますが、このソルバーではu(i-1,j,k)とu(i,j,k)で囲むことにしています。