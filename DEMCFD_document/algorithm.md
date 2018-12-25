# アルゴリズム
 基本的に酒井先生・孫博士(Dr. Sun Xiaosong)の論文を基にしたアルゴリズム群で
 構成されています。  
 固気二相流はDEM-CFD法([Tsuji,1993][tsuji1993])、
固気液三相流はDEM-VOF法([Sun and Sakai,201][SunDEMVOF])
を用いています。

[tsuji1993]:http://www.sciencedirect.com/science/article/pii/0032591093850107
[SunDEMVOF]:http://dx.doi.org/10.1016/j.ces.2015.05.059


##おおまかなアルゴリズムの流れ
```flow
st=>start: 処理開始

input=>inputoutput: データ入力
initialize=>operation: 初期化
DEM=>operation: DEM解析
iterDEM=>condition: DEM繰り返し
Drag=>operation: DragForce計算、CFD投影
CFD=>operation: CFD移流項、拡散項解析
PPE=>operation: 圧力計算
PPEconv=>condition: 収束判定
delT=>operation: 次のタイムステップへ
savecon=>condition: データ保存するか|
save=>inputoutput: データ保存
e=>end: 処理終了

st->input->initialize->DEM->iterDEM(yes)->Drag->CFD->PPE->PPEconv(yes)->delT->savecon(yes)->save->e
PPEconv(no)->PPE
iterDEM(no)->DEM
savecon(no,right)->DEM(right)
```

## 使用している離散化アルゴリズム
* 時間離散化
    1. DEM：タイムステップ上ではシンプレクティックオイラー
    2. CFD：(おそらく)オイラー陽解法
* 空間離散化
    1. DEM：Lagrange法なので、空間離散化は行わない(原理的に行われているとも言える))
    1. CFD：スタガード格子・ハイブリッド法(風上差分と二次中心差分)
* 数値積分法
    1. DEM：特筆すべきものはない
    1. CFD：(射影法/フラクショナルステップ法)・有限体積法
        * 研究室ではよくフラクショナルステップ法(Kim & Moinのもの)として扱われる手法ですが、フラクショナルステップ法はChorinの射影法を粘性項にアダムス・バッシュホース法などを適用して精度を高めたもの(だと思う)ので、実際にはChorinの射影法に近い手法が実装されていると考えたほうが良いかもしれません。
            * ただ、Chorinの論文内容をFractional-step projection methodとしている論文も多く存在しますので、フラクショナルステップ法と記しても間違いではありません。
            * 流体抗力の陰解法についてはKim & Moinのポテンシャル変換からの類推を行っています。
