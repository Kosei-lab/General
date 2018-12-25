# 拡散項の導出

## ナビエ・ストークス方程式へ戻る
まず、基礎式レベルからこの項を見直してみましょう。というのも、通常ナビエ・ストークス方程式に「せん断応力テンソル」というものは存在せず、粘性項は\(\nu \nabla^2\bm{u}\)として表現されているからです。この２つは同じものなのでしょうか？
結論から述べると、この２つは同じ項を示しますが、後者が「粘性率一定」という条件を課していること(と、局所体積平均法が用いられていないこと)によって簡略化されているのです。特に粘性率一定という条件はVOF法を用いる際に崩れる前提ですので、本ソルバーでは後者の式を用いず、テンソルを用いた計算を行っています。

ナビエ・ストークス方程式は運動方程式から導けます。まず、検査体積におけるx方向の運動方程式を考えると

\[ \rho \frac{Du}{Dt}=X+\left( \frac{\partial \sigma_{xx}}{\partial x}+\frac{\partial \tau_{xy}}{\partial y}+\frac{\partial \tau_{xz}}{\partial z}\right)\]

と書けます。右辺は全微分(時間項と移流項の合計と考えてもらって大丈夫です。この部分については移流項の項で詳しく説明します。)、左辺は第一項が外力項、第二～四項が各面が受ける応力を示しています。

面に垂直な方向\(\sigma_{xx}\)は以下のように求められます。

\[ \sigma_{xx}=-p+\mu\left( 2\frac{\partial u}{\partial x}\right)+\lambda \rm{div{\bm{u}}}\]

ここで、第1項は圧力、第2項はせん断力、第3項は体積粘性を表しています。面に水平な方向\(\tau_{xy}\)は以下のように求められます。

\[ \tau_{xy}=\mu\left( \frac{\partial u}{\partial y}+\frac{\partial v}{\partial x}\right)\]

せん断速度とせん断応力の関係(ニュートンの粘性法則)によるものです。（せん断速度の導出は少し面倒なのでここでは割愛しますが、二次元で正方形からひし形への変形を考えれば導出できます。）
ここで、よって、応力のテンソルは

\[
  \bm{\tau}+\bm{\sigma}+\bm{\theta} = 
  \mu \bm{\dot{\gamma}}+ (\rm{div}\bm{u}-p)\bm{I} =
  \mu\left(
    \begin{array}{ccc}
      2\frac{\partial u}{\partial x} & \frac{\partial u}{\partial y}+\frac{\partial v}{\partial x} & \frac{\partial u}{\partial z}+\frac{\partial w}{\partial x} \\
      \frac{\partial u}{\partial y}+\frac{\partial v}{\partial x} & 2\frac{\partial v}{\partial y} & \frac{\partial w}{\partial y}+\frac{\partial v}{\partial z} \\
      \frac{\partial u}{\partial z}+\frac{\partial w}{\partial x} & \frac{\partial w}{\partial y}+\frac{\partial v}{\partial z} & 2\frac{\partial w}{\partial z}
    \end{array}
  \right)-p
  \left(
    \begin{array}{ccc}
      1 & 0 & 0 \\
      0 & 1 & 0 \\
      0 & 0 & 1 
      \end{array}
  \right)+
  \lambda \left(
    \begin{array}{ccc}
      \rm{div \bm{u}} & 0 & 0 \\
      0 & \rm{div \bm{u}} & 0 \\
      0 & 0 & \rm{div \bm{u}} 
      \end{array}
  \right)
\]

と書き表せます。第2項は圧力として計算されます。第3項は体積粘性と呼ばれる部分ですが、本ソルバーでは非圧縮性流体を用いているので\(\rm{div} \bm{u}=0\)なので0となり、計算する必要がなくなります。よって、重要な部分は

\[
  \bm{\tau}= 
  \mu\left(
    \begin{array}{ccc}
      2\frac{\partial u}{\partial x} & \frac{\partial u}{\partial y}+\frac{\partial v}{\partial x} & \frac{\partial u}{\partial z}+\frac{\partial w}{\partial x} \\
      \frac{\partial u}{\partial y}+\frac{\partial v}{\partial x} & 2\frac{\partial v}{\partial y} & \frac{\partial w}{\partial y}+\frac{\partial v}{\partial z} \\
      \frac{\partial u}{\partial z}+\frac{\partial w}{\partial x} & \frac{\partial w}{\partial y}+\frac{\partial v}{\partial z} & 2\frac{\partial w}{\partial z}
    \end{array}
  \right)
\]

です。この発散を取ると、\(\mu \nabla^2\bm{u}\)になることが分かります。

## 局所体積平均法
局所体積平均法では、上述のせん断応力テンソルに空隙率がかかります。[^1]
よって、粘性項は

[^1]:この議論は難しいので、Anderson and Jackson 1967を読むしかないのですが、わりと\(\nabla\cdot(\varepsilon\bm{\tau})\)なのか\(\varepsilon\nabla\cdot(\bm{\tau})\)なのかは怪しいところな気がしているので、論文を読んで確認しなければならないと思っています。

\[\nabla \cdot(\varepsilon \bm{\tau})=\nabla\cdot( \varepsilon \mu  \dot{\bm{\gamma}})\]

です。せん断速度\(\dot{\bm{\gamma}}\)は以下のようにも表現できますので、この表記になっている論文もあります。

\[ \dot{\bm{\gamma}}=\nabla \bm{u}+\nabla \bm{u}^T\]

## 非ニュートン流体

非ニュートン流体ではせん断速度とせん断応力の関係が非線形になります。テンソルの関数とすると複雑ですが、必要なのは大きさについての関数ですので、よく用いられる指数則モデルでは

\[ \mu = \mu_0|\dot{\bm{\gamma}}|^{n-1}\]

\[ \bm{\tau} = \mu_0|\dot{\bm{\gamma}}|^{n-1} \dot{\bm{\gamma}}\]

となります。(n=1のときにニュートン流体となります)

\(|\dot{\bm{\gamma}}|\)は以下のように求められます。

\[ |\dot{\bm{\gamma}}|=\sqrt{2\bm{D:D}}\]

\[\bm{D}=\frac{1}{2}(\nabla \bm{u}+\nabla \bm{u}^T)\]

\(\bm{D:D}\)は各要素の掛け算の和(\( a_{11}b_{11}+a_{12}b_{12}+…\))(テンソルの内積)を示しています。