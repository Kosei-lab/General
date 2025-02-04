# アルゴリズムの詳解 (DEM)
なぜDEMなのか、DEM以外にどんな手法があるのか、などは[DEM以外の手法](algorithm_DEM_anotherMethod.md)を参照のこと

## DEMについて
DEMによる粉体の数値解析手法は、[Cundall and Strack,1979](https://doi.org/10.1680/geot.1979.29.1.47)[^1]に端を発しています。この手法における粉体粒子の支配方程式は並進・回転運動に対するニュートンの運動方程式で、以下のようになります。

[^1]: 引用数1万を越える有名論文ですが、綺麗な元論文が(東大では)見られません。この論文は古い論文によくある「今と記法が違って読みにくい！」ということがなく、比較的わかりやすいと思いますので、読んでみるのもよいかと思います。

\[ m \frac{d\bm{v}}{dt}      = \sum{\bm{F_c}}+m \bm{g} \]
\[ I \frac{d\bm{\omega}}{dt} = \sum{\bm{T}} \]

ここで、\( m,\bm{v},\bm{F_c},\bm{g},I,\bm{\omega},\bm{T}\)は、それぞれ[^2]、粒子の質量、速度、接触力、重力加速度、慣性モーメント、角速度、トルクです。粒子の質量、慣性モーメントは定義どおり、以下のように計算されます。

\[ m= \rho \frac{\pi}{6}d^3\]
\[ I= \frac{1}{10} m d^2\]

これらはコード上では
```cpp:init_dem.cpp
deriveVariableParticleProperty()
```
で計算されます。

ここで、\( d,\rho\)は、それぞれ、粒子径と粒子密度を表します。DEMでは粒子径、つまり直径を用いることが多いので、これらの計算式も直径を用いて構成されていることが多いです。
以下ではまずこの式の時間離散化手法を述べ、その後接触力・トルクの計算について述べます。

[^2]:完全に余談ですが、研究室では文字の説明は「～,～,～は、それぞれ、～～～」という形(句読点もそのまま)に揃えるようにしています。

## 時間離散化(時間進行)
並進運動の運動方程式を時間離散化すると、式は以下のようになります。

\[ m \frac{\bm{v^{n+1}}-\bm{v^{n}}}{\Delta t} = \sum{\bm{F_c^{n}}}+m \bm{g}  \]

よって次のステップの速度\(\bm{v^{n+1}}\)は以下の式によって求められます。

\[ \bm{v^{n+1}}=\bm{v^{n}}+\frac{\Delta t}{m} \left( \sum{\bm{F_c^{n}}}+m \bm{g} \right) \]

したがって次のステップでの位置\(\bm{x^{n+1}}\)は以下の式によって求められます。

\[ \bm{x^{n+1}}=\bm{x^{n}}+{\Delta t} \bm{v^{n+1}} \]

上に示したように、本ソルバーで実装されているのはオイラー法です。ただし、ここで重要なのは速度の更新の際に次のステップの速度\(\bm{v^{n+1}}\)を用いていることです。よってこの手法はオイラー法の中でも**シンプレクティックオイラー法**と呼ばれる手法で、シンプレクティック性(簡単に言えば誤差の蓄積が少ない性質)を持つ手法になっています[^3]。

[^3]: ちなみにこの部分が\(\bm{v^{n}}\)であればオイラー陽解法となり、非常に不安定な手法になります。この違いはコード上では位置の更新と速度を更新のどちらが先か、という**2行の順番の違いしかありません**。ご自分でコードを書かれるときにはご注意を…

回転の更新についても同様に、以下の2式のように更新がなされます。

\[ \bm{\omega^{n+1}}=\bm{\omega^{n}}+\frac{\Delta t}{I} \sum{\bm{T^{n}}} \]
\[ \bm{\theta^{n+1}}=\bm{\theta^{n}}+{\Delta t} \bm{\omega^{n+1}} \]

これらはコード上では
```cpp:simulation_dem.cpp
update_part()
```
で計算されています。

## 接触判定
接触判定格子を使う(未執筆)


## 接触力計算
DEMにおいて最も重要なのは接触力\( \bm{F_C}\)の計算です。接触力のモデル化には様々な手法がありますが、とりあえずはコード内に実装されている、線形モデルについて説明します。

接触は、接触の法線方向と接線方向に分解して計算します。

\[ \bm{F_C} = \bm{F_n}+\bm{F_t}\]

### 法線方向
まずは法線方向の計算について見ていきましょう。法線方向の接触力はバネとダッシュポットを並列に並べたVoigtモデルによって計算されます。

\[\bm{F_n}=-k\bm{\delta_{n}}-\eta \bm{v_n}\]

ここで、\(k,\eta\)はばね定数と粘弾性係数で、物性値から決められる定数です。粘弾性係数は反発係数等から計算されるのですが、煩雑なので[別ページ](algorithm_DEM_eta.md)に書きます。

また\(\bm{\delta_{n}},\bm{v_n}\)は、それぞれ、オーバーラップと速度の法線方向成分です。これらの値を求めるには、まず法線方向の単位ベクトル\(\bm{e_n}\) を求める必要があります。\(\bm{e_n}\) は非常に簡単で、2粒子の相対位置ベクトルと同じ方向です。単位ベクトルなので、正規化を行い、以下の式になります。

\[ \bm{e_n}=\frac{\bm{x_i}-\bm{x_j}}{|\bm{x_i}-\bm{x_j}|}\]

これを用いて、\(\bm{\delta_{n}},\bm{v_n}\)は以下のように求められます。

\[ \bm{\delta_{n}}=\left( |\bm{x_i}-\bm{x_j}| - (d_i+d_j)/2 \right)\bm{e_n}\]

\[ \bm{v_n}=\left(\bm{e_n}\cdot\bm{v_r}\right)\bm{e_n}\] 

\(\bm{v_n}\)の計算には正射影ベクトルの公式を用いています。ここで\(\bm{v_r}\)は相対速度で、以下の式によって求められます。

\[\bm{v_r}=\left(\bm{v_i}-\bm{v_j}\right)+\bm{v_{surf}}\]

今必要なのは「粒子表面間の速度」なので、粒子の回転による速度\(\bm{v_{surf}}\)も追加しなければならないことに注意してください。

\[\bm{v_{surf}}=\left(\bm{x_c-x_i}\right)\times\bm{\omega_i}-\left(\bm{x_c-x_j}\right)\times\bm{\omega_j}\]

\(\bm{x_{c}}\)は二粒子の接触点(接触面と二粒子の中間点を結んだ線分の交差点)で、二粒子間の内分点なので以下の式で求められます[^4]。

\[\bm{x_{c}}=\frac{d_j\bm{x_i}+d_i\bm{x_j}}{d_i+d_j}\]

[^4]:実際にはオーバーラップがある分この式では値が正しくありませんが、オーバーラップは直径からは非常に小さいため、この式で近似できます。

この式はよくDEMで用いられる式は\(\bm{v_{surf}}=\left( \frac{d_i}{2} \bm{\omega_i}- \frac{d_j}{2} \bm{\omega_j} \right)\times \bm{e_n} \)ですが、これはオーバーラップが0に近いときに同じ値になります。

### 接線方向
接線方向は法線方向と比較して多少複雑です。しかし、基本的な接触力の求め方は法線方向と同じです。

\[\bm{F_t}=-k\bm{\delta_{t}}-\eta \bm{v_t}\]

ここで、\(k,\eta\)は法線方向と接線方向で異なる値を設定することもありますが、このソルバーでは同一としています。

さて、\(\bm{\delta_{t}},\bm{v_t}\)はどうなるでしょうか？より簡単なのは\(\bm{v_t}\)です。これは相対速度ベクトルから法線方向の速度ベクトルを引けばよいです。

\[ \bm{v_t}=\bm{v_r}-\bm{v_n}\] 

回転速度の追加はコードでは ``` dem_surf_rel_vel()```に分離してあるので注意してください。


\(\bm{\delta_t}\)はどうなるでしょうか？DEMでは、接線方向の力は接触した点から今いる点までの距離を\(\bm{\delta_t}\)として計算します。
しかし、その大きさを単純に求めるためには接触した瞬間の点を保存して置かなければなりません。さらに接触した瞬間からも粒子は移動しますし、回転もしますので、接触した瞬間の点だけからは求められません…。ではどうするのかというと、\(\bm{\delta_t}\)を保存し、逐次更新していく手法を用います。つまり、蓄積変位\(\bm{\delta_t}\)を

\[\bm{\delta_t}^{n}=\bm{\delta_t}^{n-1}+\bm{v_t}^{n}\Delta t\]

として計算します(粒子が離れた際に0にリセットします)。

最後に、滑り条件を確認します。法線方向と違い、接線方向は摩擦により静止する可能性がありますので、滑っている場合には接線方向の接触力を動摩擦力に置き換え、また蓄積変位は蓄積されません。滑る条件は以下の式で表現されます。

\[|\bm{F_t}|>\mu |\bm{F_n}|\]

\(\mu\)は静止摩擦係数で、おおよそ0.3程度を設定する場合が多いです。この際、接線方向の力は以下のように変更されます。

\[\bm{F_t}=\mu |\bm{F_n}|\bm{e_t}\]

\(\bm{e_t}\)は接線方向の単位ベクトルで、

\[\bm{e_t}=\frac{\bm{v_t}}{|\bm{v_t}|}\]

で計算されます。(ただし、接線方向速度が0の場合は\(\frac{\bm{\delta_t}}{|\bm{\delta_t}|}\)を用い、これも0の場合は0として計算することになっています。)

また、蓄積変位は以下の式で求められます。

\[\bm{\delta_t}^{n}=\bm{\delta_t}^{n-1}\]

これは、コードでは、足した\(\bm{v_t}^{n}\Delta t\)を引き、元に戻すことで実現しています。これらの変更は、論文では場合分けとして書かれますが、コード上では上書きとして実装されています。

### 回転
現在のソルバーでは```DEM_ROT_FRICTION=2```が標準となっているので、そちらを説明したいと思います。
まず、通常の回転は法線方向の力によって生じるトルクによって引き起こされますので、以下のように計算されます。

\[\bm{T_f}=\left( \bm{x_c-x_i}\right)\times \bm{F_t}\]

通常の球体においてはこれで計算終了ですが、回転抵抗モデルを用いた際には回転抵抗によるトルクが発生するため、これを計算しなければなりません。

まず、```coef_BROF```を計算します。

\[\bm{C_{brof}}=C_{rf} |\bm{F_t}|\max \{d_i,d_j\}\]

また、接線方向トルクの合計値をしきい値として使用するため、これも計算します。

\[\bm{T_{rf,max}}= \sum{\bm{T_f}} \]

これらを用い、回転抵抗によるトルクは以下のように計算されます。

\[\bm{T_{rot}}= \begin{cases}
C_{brof}\frac{\bm{T_{rf,max}}}{|\bm{T_{rf,max}}|} & \left(C_{brof}\frac{\bm{T_{rf,max}}}{|\bm{T_{rf,max}}|} \leqq |\bm{T_{rf,max}}|\right)\\
\bm{T_{rf,max}} & \left(C_{brof}\frac{\bm{T_{rf,max}}}{|\bm{T_{rf,max}}|} > |\bm{T_{rf,max}}|\right)\\
\end{cases} \]

回転抵抗モデルの理論的背景はまだ論文になっていないので未検証(おそらく高畑さんの考案したモデルだと思われる)