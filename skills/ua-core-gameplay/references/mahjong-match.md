# 麻将消除

2026-11-08 源码快照，规则以代码为准；证据路径相对工具仓库根。关键源码：`src/board.js`（生成/求解）、`src/main.js`（交互/判定）、`src/settings.js`（默认值）、`src/faces.js`（牌面）。

**玩法定位**：「Online Domino Cafe」广告同款——3D 堆叠麻将配对消除，主线目标是按序集齐单词 RELAX 的字母。

## 棋盘结构

- 默认居中金字塔 8 层：base 7×8 经 pyramidAutoGrow 扩为 8×9，逐层每侧缩进 0.5 张（少 1 列 1 行）至顶层 1×2，共 240 张=120 对（board.js:17-26；settings.js:9-21）。
- 重叠（压盖）判定：`|dx|<0.97·tw && |dy|<0.97·th`（board.js:70）；悬空牌剔除（board.js:71-79）。
- 牌面构成：插画+花/数字/字牌/筒子/条子五类权重，同类轮换；干扰字母默认 'T'（按普通对消除、不入单词，faces.js:20-23）。

## 操作与判定

- **可点（isFree）**＝存活在板 && 所有压盖它的牌都不在板（main.js:459）。只看垂直压盖，**没有左右邻牌规则**（与传统麻将连连看不同）。被压住的牌变暗不可点。
- 点击流程（main.js:554-574）：单张目标字母直接收进单词槽；托盘模式先试入托盘，托盘满则回退为板上选中；同 key 选中数 ≥ matchCount 即消除。
- 视频模式配对动画：点第一张原地变绿（按下效果），点第二张相同的→两张先向外侧飞出再弧线汇合到画面固定位置（默认游戏区 0.5/0.5）并排，爆开绿叶后缩小消失；飞行约 1.05s、放大 1.6 倍、不旋转。

## 规则分支（均可配置）

| 开关 | 默认 | 分支 |
|---|---|---|
| trayMode | `pair`（视频模式） | `tray`=托盘模式：牌飞进右上托盘，凑齐 matchCount 张相同自动消 |
| letterMode | `pair`（字母成对） | `single`=单张点击直接入托盘；**托盘模式强制单张**：`letterPairs = letterMode!=='single' && trayMode!=='tray'`（board.js:82） |
| winCondition | `word`（集齐单词） | `clear`=清空棋盘；word 为空自动退化为 clear（main.js:1636-1638） |
| matchCount | 2 | 可选 3 |

- 目标字母 R/E/L/A/X 成对出现（同 key 组 'TR:0'），配对消除后字母飞进托盘对应格（R→1…X→5）；单张模式插入单词第 idx 槽，槽被字母占则拒绝、被普通牌占则挤走（main.js:708-728、760-783）。
- 集齐 RELAX 即通关，**棋盘不必清空**。开场：棋盘压暗，RELAX 逐个弹进托盘展示后淡出再开始。

## 胜负

只有胜利无失败：word 模式 letterCount≥word.length 胜（再延 1000ms）；clear 模式全部 gone。托盘满不构成失败；AI 无解仅提示不判负。H5 isEnded 仅胜利钩子（main.js:2031）。

## 生成与可解性（正向拆解）

铺满位置后最多 80 次尝试（seed+attempt·7919）：每次从当前自由牌集随机取 m 张赋相同 key 并移除，目标字母也放到当时可拿的位置——**拆解顺序本身就是一条合法通关路线**，必定有解（board.js:128-168）。总数先调至 `(n−L)%m==0`（board.js:85-94）；字母按 letterEarly=0.3 插前段（偏上层早见）；失败抛错（board.js:169）。

## 自动通关 AI

- 棋盘未动过→直接回放生成序；否则 repairPartition→DFS solve（nodeLimit 30000/60000，组合上限 12，top-6 分支，评分=安全组 1000+解放数×2−距离×0.35+随机 0.8，board.js:179-251）。
- word 模式以目标字母全消为提前胜利目标；收字母节奏 natural（成对 p=0.6/单张 0.15 提前），extraPairChance=0.3 穿插无关配对（main.js:961-1096）。
- 拟人点击：缓动路径、随机抖动、思考停顿、黄色波纹；优先消压住目标字母的牌。

## 玩法关键默认参数

seed 20261980、layers 8、baseCols 7、baseRows 8、insetX/Y 0.5、gapX/Y 0.04/0.048、holeRate 0、trayMode pair、letterMode pair、winCondition word、slotCount 5、matchCount 2、targetWord RELAX、letterEarly 0.3、faceKinds 40、decoyLetters 'T'。

## 其他要点

- 无刚体物理，纯逻辑判定+2.5D 渲染+贝塞尔飞行动画。
- 主题：国风/中东风双主题（牌面结构一一对应，生成器/求解器不变；当前代码默认 theme='mideast'，settings.js:6）。中东风不含宗教经文/清真寺/国旗；阿拉伯语目标词不提供（逐字拆牌无法正确连写）。
- 通用部分（手指/录制/AE/3D/H5）见 toolkit-standard.md。