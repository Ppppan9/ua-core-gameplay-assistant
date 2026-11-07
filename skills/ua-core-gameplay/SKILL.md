---
name: ua-core-gameplay
description: 买量游戏核玩（核心玩法）规则与工具规范助手。当用户询问麻将消除、打螺丝、咖啡分拣、方块射击、祖玛、拔钉救人、箭头挪出、体素射击、圆环传送带这 9 类广告玩法的规则/判定逻辑/胜负条件/关卡生成/可解性/自动通关策略，或要复刻买量广告玩法、开发新的核玩素材工具（录屏、AE/3D/H5 试玩导出）时使用。
---

# 买量游戏核玩助手

本 Skill 是用户 9 个核玩工具仓库的规则知识库（2026-11-08 快照）。核玩工具 = 复刻买量视频广告核心玩法的单文件离线网页工具，用于录制广告素材、导出 AE 工程 / 3D 动画 / H5 试玩广告。

## 路由：先识别游戏，再读对应参考

| 用户提到的游戏/别名 | 读这个文件 |
|---|---|
| 麻将、麻将消除、Domino Cafe、RELAX | [references/mahjong-match.md](references/mahjong-match.md) |
| 螺丝、打螺丝、拧螺丝、折螺丝 | [references/screw.md](references/screw.md) |
| 咖啡、咖啡分拣、杯托、Pop-it | [references/coffee-sort.md](references/coffee-sort.md) |
| 方块射击、炮台打方块（横屏库存队列式） | [references/block-shooter.md](references/block-shooter.md) |
| 祖玛、青蛙吐球 | [references/zuma.md](references/zuma.md) |
| 拔钉、拔针、拔钉救人、pin rescue | [references/pin-rescue.md](references/pin-rescue.md) |
| 箭头、箭头挪出、箭头逃脱 | [references/arrow-escape.md](references/arrow-escape.md) |
| 体素射击、旋转体素目标（竖屏库存网格式） | [references/voxel-shoot.md](references/voxel-shoot.md) |
| 圆环、传送带、解环、ring belt | [references/ring-belt.md](references/ring-belt.md) |
| 通用规范：按钮/设置键名/手指/录制/AE/3D/H5 导出/测试 | [references/toolkit-standard.md](references/toolkit-standard.md) |

- 只读需要的文件，不要全部加载。跨游戏对比时读相关的几个。
- 「方块射击」(block-shooter) 与「体素射击」(voxel-shoot) 是两个独立游戏：前者 6 列队列+5 槽位、正面外露判定；后者库存网格+旋转 3D 目标、严格同色+镜头可见判定。用户混淆时先澄清。
- 开发任何新核玩工具或修改通用部分（导出、手指、面板、设置迁移）时，toolkit-standard.md 必读。

## 回答规则问题的约束

1. **以代码为准**：参考文件中的规则提取自源码（附 文件:行号，路径相对各仓库根）。README/注释可能过时（已知 ring-belt README 的「导出未做」已过时）。若用户本地有对应仓库且问题涉及精确数值/判定，先核对当前代码再回答。
2. **区分默认值与可配置分支**：很多规则有模式开关（如麻将 pair/tray、螺丝 tap2/单击、祖玛 gapRule always/match）。回答时给出默认行为，并注明可切换的分支。
3. **区分确证与补全**：参考文件标注「工具补全」的行为是参考视频未确认、工具按同类玩法补全的设计，回答时不得说成原广告的确证行为。
4. **可解性是硬约束**：9 个工具的关卡生成器都保证有解（算法各不同，见各参考）。设计新关卡/新生成器时必须保持同等保证，并说明验证方式（求解器实测）。
5. 不知道就说不知道，回仓库查证；不要编造判定逻辑、参数值或渠道规范。

## 开发新核玩工具的工作流

1. 读 [references/toolkit-standard.md](references/toolkit-standard.md)，通用部分（单文件构建、按钮顺序、设置键名与迁移、手指/hideCursor、录制触发、AE/3D/H5 导出包结构、渠道 CTA、测试矩阵）按规范实现，不自创规格。
2. 玩法部分参考最接近的游戏文件：判定逻辑、物理参数、生成器可解性算法、自动通关 AI 都有已验证的实现范式。
3. 工具专有参数放在对应分组末尾或单独分组；不常用参数标 `extra.adv=true` 收进默认折叠的高级设置。
4. 交付前跑规范 §6 的测试矩阵（test:logic / npm test / test:ae / test:3d / test:h5）。
5. 参考视频未确认的行为，在文档中显式标注为补全/近似。

## 素材与导出类问题的约束

- H5 试玩包：不带手指、默认不自动通关（h5AutoPlay=false）、无编辑器 UI、仅游戏+CTA、contain/letterbox 自适应；注入/剥离只能作用于 `<script>` 之外的标记（全文正则曾切断主脚本导致黑屏事故）。
- 录制只由「⏺ 开始录制」触发，autoRec 默认关；AE 记录与录制可同步（aeWithRec）。
- 渠道体积红线：Facebook 单 HTML ≤2MB；AppLovin/Moloco/Unity <5MB；Moloco 不得引 MRAID 脚本/XHR。

## 维护与迭代（长期约束）

- **隐私**：插件任何文件不得写入来源仓库地址/链接/归属信息；证据只保留相对路径 文件:行号。
- 规则演进或确认新事实时，更新对应 references 文件并升 plugin.json 版本号；新增游戏时在 references/ 加文件并在上方路由表加一行。
- 发布前自检：manifest 可解析、frontmatter 合法、引用链接存在、无来源地址与凭据。