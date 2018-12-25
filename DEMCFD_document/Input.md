# inputファイルの記述
# common.cfg
## ドメインの設定
```cpp
//common.cfg
domain.auto_merge: [int=1],[int=1],[int=1]
domain.min: [double=DBL_MAX],[double=DBL_MAX],[double=DBL_MAX]//計算領域
domain.max: [double=DBL_MAX],[double=DBL_MAX],[double=DBL_MAX]
```

## DEMonly, 重力
```cpp
//common.cfg
gravity:[double=0.0],[double=9.81],[double=0.0]//重力
demonly:[bool=false]//DEMonly
```

## オブジェクト設定
```cpp
//common.cfg
numobjects: [int=0]//オブジェクトの数
object[0,1,2...].eval_fluid_action: [int=0]
object[0,1,2...].file:              [string]//ファイル名
object[0,1,2...].scale:             [double=1.0]//スケール
object[0,1,2...].scale.x:           [double=1.0]//各方向のスケール
object[0,1,2...].scale.y:           [double=1.0]
object[0,1,2...].scale.z:           [double=1.0]
object[0,1,2...].rotate.angle:      [double=0]
object[0,1,2...].rotate.axis:       ["x","y","z"]
object[0,1,2...].rotate.base:       [double=0],[double=0],[double=0]
object[0,1,2...].rot.axis:          ["x","y","z"]
object[0,1,2...].offset:            [double=0],[double=0],[double=0]//初期位置の移動
object[0,1,2...].activecfd:         [bool=true]
object[0,1,2...].activedem:         [bool=true]
object[0,1,2...].move:              [bool=false]
object[0,1,2...].move.fixed:        [bool=false]
object[0,1,2...].rot.axis.base:     [double=0],[double=0],[double=0]
object[0,1,2...].translation.limit: [double=-1.0]//移動の最大距離(移動した後止まる)
object[0,1,2...].rot.speed:         [double=0.0]// *2pi/60=RPMなので、rad/s？
object[0,1,2...].rot.axis:          [double=rotAxisDirection.x],[double=rotAxisDirection.y],[double=rotAxisDirection.z]
object[0,1,2...].sdf.ptclscale:     [double=0.25] //SDFのスケール(粒子径に対する比)
object[0,1,2...].sdf.resolution_abs:[double=resolution]
object[0,1,2...].sdf.range:         [double=2*ptclDiameter]//SDFの距離
//special mode
object[0,1,2...].special_mode:      [string="none"]//"shaker","pendulum","oscillation"
//shaker
object[0,1,2...].shaker.max_yaw:    [double]
object[0,1,2...].shaker.max_pitch:  [double]
object[0,1,2...].shaker.omega_roll: [double]
object[0,1,2...].shaker.omega_yaw:  [double]
object[0,1,2...].shaker.omega_pitch:[double]
//pendulum
object[0,1,2...].pendulum.max_angle:[double]
object[0,1,2...].pendulum.frequency:[double]
object[0,1,2...].pendulum.radius:   [double]
object[0,1,2...].pendulum.spin:     [double]
//oscillation
object[0,1,2...].oscillation.amp:   [double]
object[0,1,2...].oscillation.freq:  [double]
object[0,1,2...].oscillation.axis:  [double],[double],[double]
```

# input_cfd.dat

## 時間刻み
```cpp
//input_cfd.dat
elapsedtime:[double]//経過時間
cfd.courant:[double]//クーラン数
cfd.itelimit:[int=20]//DEMに対するCFDの反復回数(最大)
```

## CFD全般
```cpp
//input_cfd.dat
density:            [double]//密度
viscoity:           [double]//粘度
cfd.inv_sound_v:    [double=1.0e-3]//音速の逆数
matrix.epsilon:     [double=1.0e-8]//残差の上限
cfd.velocity_solver:[int=0]//移流拡散項の解き方(0:陽的 1:semi-implicit 2:移流項のみ陰的)
```

## DEM-CFD関係フラグ
```cpp
//input_cfd.dat
demcfd.use_reconnected_voidage: [int=0]
demcfd.use_implicit_drag:       [int=0]
demcfd.calc_dem_gradp:          [int=1]//0なら圧力を計算しない
demcfd.use_biased_grad_pres:    [int=1]//0なら中心差分、1なら風上差分
demcfd.use_extended_pres:       [int=1]//1ならibmの圧力をextendする？
demcfd.drag_vel_interp_type:    [int=2]//0=cell_const,1=cell_splitlinear,2=stag_linear
demcfd.use_biased_drag_vel:     [int=1]
demcfd.use_one_way_coupling:    [int=0]
```

## グリッド数設定
```cpp
//input_cfd.dat
grid:   [int],[int],[int]//グリッド数
```

## 壁面設定
```cpp
//input_cfd.dat
numboundaries: [int=0]//壁面ごとの設定(szobjは壁面のID)
boundary[0,1,2...].type: [int]//PhysBC_None = -1,PhysBC_Wall = 1,PhysBC_Inlet = 2,PhysBC_Outlet = 3,PhysBC_Slip = 5
boundary[0,1,2...].min:  [int],[int],[int]//グリッドIDとしての壁面の位置
boundary[0,1,2...].max:  [int],[int],[int]
boundary[0,1,2...].vel:  [double=ubc],[double=vbc],[double=wbc]//速度境界//velocity.yplusなどで設定される値がデフォルト
boundary[0,1,2...].pres: [double=0.0]//圧力境界
```

## 境界流速設定
```cpp
//input_cfd.dat
velocity.yplus: [double],[double],[double]
velocity.yminus:[double],[double],[double]
velocity.xplus: [double],[double],[double]
velocity.xminus:[double],[double],[double]
velocity.zplus: [double],[double],[double]
velocity.zminus:[double],[double],[double]
```

## 時間に関する特殊設定
south2は特殊な時間が使われている場合にのみ使用されるようだが、機能は未考察
```cpp
//input_cfd.dat
south2:             [double=0.0],[double=0.0],[double=0.0]
startup.2.time:     [double=0.0]
startup.4.time:     [double=0.0]
startup.4.time_init:[double=0.0]
```

## 乱流モデル
```cpp
//input_cfd.dat
turbulence.model:                   [string="laminar"]//"les_smagorinsky"
turbulence.les_smagorinsky.delta:   [double=pow(grid->volume, 1.0/3.0)]//1辺の長さ
turbulence.les_smagorinsky.cs:      [double=0.173]
turbulence.les_smagorinsky.damp:    [int=0] 
turbulence.wallfunc.active:         [bool=false]
```

## 多孔質体(porus_media)
```cpp
//input_cfd.dat
porosity.cutoff:                [double=0.05]
porosity.zone[0,1,2...].active: [int=0]//1=active
porosity.zone[0,1,2...].porosity: [double]//多孔質体の空隙率
porosity.zone[0,1,2...].model: [string]//"free_draining""ergun"//free_drainingは圧損なし
porosity.zone[0,1,2...].dp_eff:[double]//ergun式の計算の際に使用する等価粒径
```

## IBM設定
```cpp
//input_cfd.dat
ibm.forcing_type:       [int=0]// IB method, 0=volume average, 1=distance interpolation
ibm.use_sdf_sign:       [int=1]// do not touch, always keep this =1
ibm.extend_thickness:   [double=1.5]
ibm.do_pre_forcing:     [int=0]
ibm.do_post_forcing:    [int=1]
ibm.dens_scale:         [double=1.0]//密度スケーリング
```

## IBMにおけるヘビサイド関数。※現在使用されない
```cpp
//input_cfd.dat
ibm.heaviside.shift:    [double]
ibm.heaviside.thickness:[double]
```

## CFD_圧力ソルバーフラグ
```cpp
//input_cfd.dat
demcfd.depsi_max:               [double=1.0]
mac.mg.use_full_smooth:         [int=1]
mac.mg.use_anisotropic_coarsen: [int=1]
mac.cg.verbose:                 [int=1]
mac.mg.bot_tol_rel:             [double=1.0e-3]
mac.mg.pre_smooth:              [int=2]
mac.mg.post_smooth:             [int=2]
```

## VOF設定
```cpp
//input_cfd.dat
vof.dens_liquid:    [double]//液体密度
vof.visc_liquid:    [double]//液体粘度
vof.dens_gas:       [double]//気体密度
vof.visc_gas:       [double]//気体粘度
vof.nextend:        [int=4]
vof.extend_method:  [int=1]
//表面張力
vof.tension.coef:               [double=0.0]
vof.tension.nsmooth:            [int=2]
vof.tension.do_bed_curv_corr:   [int=0]
vof.tension.bed_voidage:        [double=0.8]
vof.tension.cangle:             [double=90.0]

vof.hxlo:   [double=-99999]
vof.hylo:   [double=-99999]
vof.hzlo:   [double=-99999]
vof.hx:     [double=99999]
vof.hy:     [double]
vof.hz:     [double]

vof.init.nblock:    [int=0]//液体・気体を設定するためのブロック
vof.init.block[0,1,2...].sign:      [double=1]// sign of color, liquid=1, gas=-1
vof.init.block[0,1,2...].vel:       [double=0.0],[double=0.0],[double=0.0]
//BlockType
vof.init.block[0,1,2...].shape:     [string]//"rect","cyl","sphere","custom"
//rect
vof.init.block[0,1,2...].rectlo:    [double],[double],[double]//直方体の最大・最小位置
vof.init.block[0,1,2...].recthi:    [double],[double],[double]
//cyl
vof.init.block[0,1,2...].cyla:      [double],[double],[double]//円柱の上円中心
vof.init.block[0,1,2...].cylb:      [double],[double],[double]//円柱の下円中心
vof.init.block[0,1,2...].cylr:      [double]//円柱の半径
//sphere
vof.init.block[0,1,2...].sphcen:    [double],[double],[double]//球中心
vof.init.block[0,1,2...].sphrad:    [double]//球半径
//custom
vof.init.block[0,1,2...].file:      [string]//ファイル名

vof.do_global_cons: [int=0]//special global conservation？
```

# input_dem.dat
## DEM全般
```cpp
//input_dem.dat
particle.maxnumber:             [int=granular->numRequiredPart]//必要な粒子数？
particletype[0,1,2...].density: [double=granular->partRho]//デフォルト密度
init.check_wall_dist:           [double=0.0]
timestep:                       [double]//DEMの時間刻み
particle.number:                [int]//粒子数
particle.diameter:              [double]//粒子径
particle.density:               [double]//粒子密度
particle.spring:                [double]//ばね定数
particle.restitution:           [double]//反発係数
particle.friction:              [double]//摩擦係数
particle.rotfriction:           [double]//回転抵抗
```

## 計算時間、出力回数
```cpp
//input_dem.dat
iterations.current: [int]//現在の反復回数
iterations.total:   [int]//合計反復回数
output.frequency:   [int]//出力間隔
output.count:       [int]//出力数
elapsedtime:        [double]//経過時間
```

## 出力制御
```cpp
//input_dem.dat
output.fluid_epsilon:       [int=0]//fluidの情報を粒子に出力する
output.fluid_drag:          [int=0]
output.fluid_gradp:         [int=0]
output.rot_speed:           [int=1]
output.coordination_num:    [int=0]//配位数
output.overlap_mag_n:       [int=0]//法線方向のオーバーラップ
output.overlap_mag_t:       [int=0]//接線方向のオーバーラップ
output.export_contact_info: [int=0]//接触力に関する情報を別ファイルに出力する
```

## DEMパラメーター(壁面との)
```cpp
//input_dem.dat
partwall.friction:          [double=granular->coefFriction]//粒子とのものがデフォルト
partwall.restitution:       [double=granular->coefRestitution]
partwall.hamaker:           [double=granular->hamakerNumber]
partwall.hamaker.dist.min:  [double=granular->hamakerMinDist]
partwall.hamaker.dist.max:  [double=granular->hamakerMaxDist]
```

## DEMの計算領域(contact gridの範囲)など
```cpp
//input_dem.dat
demgrid.min:    [double= vRegionMin.x],[double= vRegionMin.y],[double= vRegionMin.z]//デフォルトでオブジェクトの最大・最小座標
demgrid.max:    [double= vRegionMax.x],[double= vRegionMax.y],[double= vRegionMax.z]
init.file:      [string]//DEMのinitにファイルが使える？
init.velocity:  [double],[double],[double]//初期配置時の速度
```

## DEM付着力
```cpp
//input_dem.dat
particle.hamaker:           [double=0.0]//ハマーカー定数
particle.hamaker.dist.min:  [double=4.0e-10]//最大・最小のカットオフ距離
particle.hamaker.dist.max:  [double=1.5e-8]
```

## DEM粗視化モデル
```cpp
//input_dem.dat
particle.coarseratio:[double=1.0]//粗視化率
```

## 粒子流入・流出
```cpp
//input_dem.dat
inject.npatch:  [int=0]//流出境界の数
inject.patch[0,1,2...].part_type:       [int=0]
inject.patch[0,1,2...].part_diameter:   [double=granular->partDiameter]
inject.patch[0,1,2...].spacing_rel:     [double=1.5]
inject.patch[0,1,2...].interval_rel:    [double=1.5]
inject.patch[0,1,2...].vel:             [double]
inject.patch[0,1,2...].x0:              [double],[double],[double]
inject.patch[0,1,2...].x1:              [double],[double],[double]
inject.patch[0,1,2...].x2:              [double],[double],[double]
remove.frequency:   [int=-1]//削除間隔
remove.min:         [double=demGrid->coordinate_min.x],[double=demGrid->coordinate_min.y],[double=demGrid->coordinate_min.z]
remove.max:         [double=demGrid->coordinate_max.x],[double=demGrid->coordinate_max.y],[double=demGrid->coordinate_max.z]
```

## ランダムパッキング
```cpp
//input_dem.dat
randpack.initial_shrink:    [double]//はじめ粒子径に対してどのくらい小さくするか
randpack.growth_rate:       [double]//成長率
randpack.check_init_overlap:[int=0]
```

## 粒子初期配置の方向
```cpp
//input_dem.dat
init.scandir: [string="x"]//x,y,z
```

## DEM初期配置
```cpp
//input_dem.dat
init.block_num: [int=0]//初期配置のブロック数
init.min:       [double=vMin.x],[double=vMin.y],[double=vMin.z]//体系の大きさがデフォルト
init.max:       [double=vMax.x],[double=vMax.y],[double=vMax.z]//体系の大きさがデフォルト
init.block[0,1,2...].min:               [double=vMin.x],[double=vMin.y],[double=vMin.z]
init.block[0,1,2...].min:               [double=vMax.x],[double=vMax.y],[double=vMax.z]
init.block[0,1,2...].particle_num:      [int]//粒子数
init.block[0,1,2...].particle_diameter: [double]//粒子径
init.block[0,1,2...].particle_type:     [int]//タイプ
```

## reorder
```cpp
//input_dem.dat
reorder.frequency:[int=-1]
```

## 液架橋関連
```cpp
//input_dem.dat
liquidbridge.total_volume:  [double=0.0]//液量
liquidbridge.gamma:         [double=0.0]//係数γ
liquidbridge.theta:         [double=0.0]//接触角θ
liquidbridge.theta_pp:      [double=0.0]
liquidbridge.theta_pw:      [double=0.0]
liquidbridge.theta_pw[0,1,2...]:   [double=0.0]//壁面のID
liquidbridge.min_dist:      [double=0.0]
liquidbridge.force_model:   [int=2]
liquidbridge.use_table:     [int=1]
liquidbridge.output_links:  [int=0]
```

## DEM固定
```cpp
//input_dem.dat
dem.fixed_motion:[int=0]
```

# restartファイル
## CFD
```cpp
//input_cfd.dat
[int=type],[double=u],[double=v],[double=w],[double=p],[double=vol_frac]
```

## DEM
3通りのいづれかであれば認識される
```cpp
//input_dem.dat
[int=type],[double=diameter],[double=CGR],[double=x],[double=z],[double=z],[double=u],[double=v],[double=w],[double=omegax],[double=omegay],[double=omegaz]
```
```cpp
//input_dem.dat
[int=type],[double=diameter],[double=CGR],[double=x],[double=z],[double=z],[double=u],[double=v],[double=w]
```
```cpp
//input_dem.dat
[int=type],[double=diameter],[double=CGR],[double=x],[double=z],[double=z],
```