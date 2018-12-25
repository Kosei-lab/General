# DEMVOF -固気液混相流シミュレータ-
固気二相流/固気液三相流の数値解析を行うソフトウェアです。

markdown・Doxygen等を用いて簡単なドキュメントを作成しようと思います。
研究開発用のコードですので、数式を用いずに解説することが困難な理論的な部分をドキュメント形式で記録できることを目的としています。

---

基本的な構成(自分で書くもの)は以下のようなもので大丈夫かな？
1. 使用者向け
    1. [x] [inputファイルの記法](input.md)（必要な文言とオプション)
    2. [ ] [vtkファイルの見方](output.md)
2. 開発者向け
    1. [x] [実装されているアルゴリズム](algorithm.md)
        1. [x] [DEM](algorithm_DEM.md)
            1. [x] [DEM以外の手法](algorithm_DEM_anotherMethod.md)
            2. [x] [粘弾性係数の導出](algorithm_DEM_eta.md)
        2. [x] [DEM-SDF](algorithm_DEM_SDF.md)
        3. [ ] DEMCFD-DragForce
        4. [ ] [CFD](algorithm_CFD.md)
            1. [ ] [CFD advection and diffusion](algorithm_CFD_advecdiff.md)
                1. [ ] [拡散項の導出](algorithm_CFD_advecdiff_deriviation_diff.md)
            2. [ ] CFD IBM
            3. [ ] CFD pressure solver
        5. [ ] VOF solver(?)
        6. [ ] 
        7. [ ] VDW力
        8. [ ] 液架橋力
        9. [ ] randpack
        10. [ ] reorder
        11. [ ] heat

    2. [x] [独自機能の説明](functions.md)(書きかけ)

---

* 理論から実装までの間をどうするのかが難しい…
* 別ファイルに書いて、コード内ではそれを参照する形が良いか？

\[ 
    |I_2|= 
    \int_{0}^T \psi(t) 
        \left
        \{ u(a,t)- \int_{\gamma(t)}^a \frac{d\theta}{k(\theta,t)} 
        \int_{a}^\theta c(\xi)u_t(\xi,t)\,d\xi 
    \right\} dt 
 \]
 
 | 左詰め | 中央揃え | 右詰め |
 |:------ |:--------:| ------:|
 | 1-1    | 1-2      | 1-3    |
 
```mermaid
graph LR;
    A[データ入力]-->B[データ入力];
    B-->C[空ではないか?];    
    C-->B;
    click B "http;//google.com" "link";
```


