# 圆环传送带

2026-11-08 源码快照（phase49），规则以代码为准；证据路径相对工具仓库根。关键源码：`src/game.js`（物理/判定/胜负）、`src/levels.js`（关卡/安全区校验）、`src/app.js`（自动通关/导出）、`src/items.js`（z 序）、`src/settings.js`。

**玩法定位**：参考视频 ref/ref.mp4 的可玩复刻。完整循环：**拖动开口环旋转→缺口对准夹扣解锁脱环（环碎裂）→环内同色球重力洒出→经漏斗落上循环传送带→带球入同色盒子→全部球入盒过关**。环不自动转，只有玩家/AI 拖拽才转；球是纯物理实体。

## 关卡结构

- 设计分辨率 864×1920（game.js:4）；上部棋盘内框 50–814×290–1012 放环树，中部漏斗 y990 收口至 392–472@y1150，底部椭圆传送带（直段 145–720、cy1256、R38）默认 **23 孔**，再下方 2/3/4 列盒子（4 列 x=170/345/520/695）。
- 环字段 `{id,color,x,y,R,gap:[中心角°,宽°],parent,anchor,closed,hinge,balls}`（levels.js:2-3）；球色=环色，开局堆环底并预沉降 180 步。
- 盒子=每列自上而下颜色栈，**每盒容量 3**。内置 5 关+第 6 关风车 3×3（9 环 48 球 16 盒），坐标逐帧量自视频。
- 颜色映射：加载时 colorMap 统一置换环/盒色，默认 yellow→orange、pink→red、sky→cyan、lilac→purple、grey→gray；useArtGap=58° 统一覆盖缺口宽。

## 操作与判定

- 唯一操作=**按住拖拽非 anchor 环旋转**（命中取最近管身），手动角速度上限 12rad/s；子环子树随父环刚性转动（game.js:74-84,135）。
- **多夹扣锁定**：joints=被父夹(1)+存活子环数，>1 不可转，拖动只抖动+tick 声（lockMultiJoint，game.js:168-171）。
- **脱扣判定**：夹扣方向相对缺口中心角 loc，解锁窗=max(0.5°, 缺口半角−asin(夹扣占位/R)−3° 余量)，|loc|≤窗或跨帧扫过中心即脱；夹扣占位 clipClear=max(管粗T,(25+34·clampScale)·R/151)；夹扣入缺口后自动吸附转正防「悬空」（game.js:157-173）。
- **铰接颈 hinge**：hingeRelease=true（默认）时铰接环转到解锁即脱、余球洒出；闭环/锚环不参与脱扣。
- 脱落默认**碎裂**：环身折成碎片、递归碎子环，锚环在子环全脱后也碎；残留夹扣默认粘父环随转（clipFollowParent）。
- **缺口安全区（GAP_SAFE_DEG=15°）**：子环夹点（朝父）与父环颈点（朝子）均不得落在任一方「缺口半角+15°」内，gapSafeViolations() 在 test:logic 对全部关卡校验为空（levels.js:73-80）。
- z 序：父环压子环、颈在父环下、夹块在父环下子环上（items.js:71-93）。

## 球移动与分拣

- Verlet 物理：重力 4200px/s²、弹性 0.25；环壁碰撞**仅缺口可穿越**（game.js:97-116）。
- 落至带尖区→捕获进上边最近空孔；带以 900px/s 循环（上边右→左、下边左→右）。
- 球到下边时若某列**栈顶盒同色、未满（n+pend<3）且水平就近**→120ms 落盒；满 3 球盒飞走、栈下一盒上移；**无匹配盒的球永远绕带**（game.js:141-147）。

## 胜负

- 胜=全部球 state='boxed' 持续 60ms→过关，停在完成盘面、无结算页；手动须点「⏭ 下一关」，自动通关自行续关；loopLevels=true 时 1–6 关循环（game.js:155,175-179）。
- **代码无失败判定**：带满只是软堵死，phase39 起容量=孔数 23 不再限 6，错序不再硬卡。

## 关卡生成与可解性

- 关卡为手工量取数据，非随机生成。几何规范化两步：spaceJoints 沿父→子方向推远子树保证可见铰接颈（圆心距≥父R+0.27父R+颈长10+夹块半高+子R，levels.js:58-71）；fitRings 整组等比缩放进内框。
- 每色球数=盒子总容量需求（如 L1 12 红球=4 盒×3）。
- 第 6 关由 tools/l20_solver.py 设计：枚举 2520 种合法拆环序（子环先于父环）、模拟带-盒消耗并加重力约束，盒栈顺序使容量 6 时**仅 1 种顺序不堵**：br→bl→ee→ww→w→ss→s→e。
- 「换一局」=颜色置换+盒序洗牌+非锚缺口±25° 微调，数量守恒。

## 自动通关 AI 与「耐心版」

- 按关卡 plan [环id, detach/spill, 速倍, 停顿, 等落定] 执行；autoUnlockFirst=true（默认）把 spill（转缺口朝下倒球）改写为直接 detach 解环；拖拽 260°/s，detach 目标角+0.12rad 过冲，spill 时绕开解锁角防误脱，失败 3 次跳过该步（app.js:63-88）。
- **耐心版 autoPatient**（默认关）：每解一环前等传送带「消化完」——无落球/飞盒/列移位、环外无自由球、带上无「有开放盒可去」的球，且连续稳定 20 帧（上限 60s）才解下一环，防抢跑堵带（app.js:72-74；test/patient.mjs 验证开/关均通关且耐心版确有等待）。

## 玩法关键默认参数

beltSpeed 900px/s、beltSlots 23、boxCapacity 3、gravity 4200、bounce 0.25、ballRadius 24（关卡覆盖 20.5/17.5）、带上球半径 16.5、ringThick 30、dropTime 120ms、lcDelay 60ms、shatter true、unlockMargin 3°、unlockSnap true、lockMultiJoint true、hingeRelease true、clipFollowParent true、useArtGap 58°、startLevel 1、loopLevels true。

## 已知差异/限制（已核实）

- **README「第二阶段 AE/3D/H5 导出未做」已过时**：当前 app.js（phase49）三者均已实现并接线（ACTIONS app.js:194），且支持多主题批量打包。
- 真实缺口以 STANDARD_CHECK.md 为准：scene.fbx 未生成（环境无 Blender，glb/scene3d.json/blender_import.py 已齐）；H5 总 ZIP 5.69MB 超 5MB 告警阈值；透明 PNG 导入生成关卡不适用/未做。
- levels.js:41「传送带只有 6 格」为历史注释，运行时容量=23。