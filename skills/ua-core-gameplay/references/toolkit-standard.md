# 核玩工具通用规范（TOOLKIT STANDARD）

2026-11-08 快照，最新版《核玩工具通用规范》（含 §7/§8）。以麻将消除工具为基准整理，打螺丝 v1.3.1 起遵守。**之后制作的每一个核玩工具都必须遵守本规范。**

## 硬性要求（用户确认的长期规则）

- 每个工具**默认内置**「导出 AE 工程」和「导出 3D 动画（glTF/FBX/Blender，支持 UE5/C4D）」，每个元素都是带关键帧的独立图层/节点；导出包结构、图层命名、设置键名与麻将工具一致。
- 适用时支持**导入任意透明通道图片生成关卡**。
- 默认手指统一使用麻将工具的手图（各工具 `src/default_hand.js`，含其旋转/翻转/大小/指尖锚点默认值），不再自绘手指；用户可上传 PNG 覆盖。（个别旧工具如 screw 默认无手图，属历史例外。）
- **手指与鼠标互斥**（hideCursor，默认 true）：手指图显示在游戏画面上时（自动通关可见/手动跟随/拖动中，含指针捕获期间）画布 `cursor:none`；面板/按钮上保持正常鼠标；无手指时画布 `cursor:pointer`。手动跟随时指尖锚点严格落在指针位置（0 偏移，pointermove 当帧更新），鼠标隐藏时手指立即显示不淡入。系统鼠标本来就不被录制/AE/3D 采集。测试：`npm run test:cursor`。
- 每个工具默认内置「导出 H5 试玩广告」面板（在「导出 3D」之后、「结束页」之前）：一键导出自包含试玩 ZIP，**不得带编辑器 UI**，仅游戏画面+CTA，多尺寸自适应（§5）。

## 1. 单文件与设置

- `npm run build` → 单文件 `index.html`（离线可用，无 CDN）；设置存 localStorage，带 `version`；`migrate()` 只做：补默认值、把**仍等于旧默认值**的项换成新默认、旧键→新键（含单位换算）。**不得覆盖用户改过的值**；导入的 JSON 原样采用。
- 顶部按钮顺序/文字一致：`▶ 自动通关`（运行中 `■ 停止`）、`⏺ 开始录制`（`■ 停止录制`）、`↻ 重新开始`、`🎲 换一局`、[工具专有按钮]、`📷 截图`、`导出设置`、`导入设置`、`恢复默认`。
- 导出设置 JSON：`{ app: '<tool-id>', version, settings }`。

## 2. 通用参数（键名/中文标签/分组/顺序）

面板分组顺序：…工具专有分组… → `音效` → `手指 / 自动通关` → `录制` → `导出 AE 工程` → `导出 3D 动画（UE5 / C4D / Blender）` → `导出 H5 试玩广告` → `结束页(End Card)`。

| 分组 | 键 | 标签 | 类型/范围 | 麻将默认 | 螺丝默认 |
|---|---|---|---|---|---|
| 音效 | sfxMute | 静音 | bool | false | false |
| 音效 | sfxVolume | 总音量 | 0~1.5 | 0.7 | 0.8 |
| 手指/自动通关 | autoSpeed | 整体速度倍率 | 0.25~6 | 1 | 1 |
| | moveMin/moveMax | 移动耗时最小/最大(ms) | 50~2000/50~3000 | 380/700 | 220/300 |
| | delayMin/delayMax | 点击间隔最小/最大(ms) | 0~2000/0~4000 | 120/420 | 150/350 |
| | thinkChance/thinkMax | 思考停顿概率/最长(ms) | 0~0.6/0~4000 | 0.12/900 | 0/900 |
| | jitter | 落点随机抖动(px) | 0~40 | 10 | 4 |
| | handSize | 手指大小(画面宽比例) | 0.1~0.9 | 0.42 | 0.42 |
| | handAnchorX/Y | 指尖锚点(0~1) | 0~1 | 工具图而定 | 0.05/0.05 |
| | handImage | 上传手指图片(透明PNG) | file(dataURL) | 内置手 | 空 |
| | pressDepth | 按压幅度 | 0~0.3 | 0.08 | 0.07 |
| | pressEnabled | 点击按下效果 | bool | true | true |
| | tapRipple | 点击处显示波纹圈（参考视频没有） | bool | false | false |
| | hideCursor | 手指出现时隐藏鼠标 | bool | true | true |
| 录制 | recFps/recFormat/recBitrate | 帧率/格式(自动优先MP4)/码率(Mbps) | | 30/auto/8 | 30/auto/8 |
| | autoRec | 自动通关开始时自动录制（**默认关**） | bool | false | false |
| | autoStop | 通关后停止录制并下载（仅正在录制时） | bool | true | true |
| | tailSec | 通关后继续录制(秒) | 0~10 | 2.5 | 1.2 |
| AE | _aeToggle / _aeExport | `● 开始记录`/`■ 停止并导出`、`⬇ 导出 ZIP` | button | | |
| | aeWithRec | 录制视频时同步记录，停止后自动导出 | bool | true | true |
| | aeFps | AE 合成帧率 | 24/25/30/60 | 30 | 30 |
| | aeSize | AE 合成尺寸 | canvas/720x1280/1080x1920 | canvas | canvas |
| | aeSampling | 关键帧方式 | reduce/bake | reduce | reduce |
| | aeNames | 图层命名 | zh/en | zh | zh |
| | aeAudio | 音频 | both/mixed/events/none | both | both |
| | aeSaveAep | 脚本跑完提示另存 .aep | bool | true | true |
| 3D | _export3d | `导出 3D 动画 (glTF/FBX/Blender)` | button | | |
| | export3dFps | 3D 动画帧率 | 24/25/30/60 | 30 | 30 |
| | export3dHand | 包含手指（贴图平面） | bool | true | true |
| H5 | _exportH5 | `⬇ 导出 H5 试玩 ZIP` | button | | |
| | h5Format | 渠道（多选 CSV） | 8 渠道，默认全部 | all | all |
| | h5Orientation | 试玩方向 | 0 双端/1 竖屏/2 横屏 | 1 | 1 |
| | h5Device | 预览机型 | adaptive/机型/custom | adaptive | adaptive |
| | h5DeviceW/H | 预览宽/高(px) | 240~1440/400~2560 | 390/844 | 390/844 |
| | h5CtaText | CTA 按钮文案 | text | 立即下载 | 立即下载 |
| | h5StoreUrl | 商店/落地页 URL | text | placeholder | placeholder |
| | h5GameName / h5Icon | 游戏名/应用图标（可选） | text/file | 工具中文名/空 | 同 |
| | h5MaxMb | 体积告警阈值(MB) | number | 5 | 5 |
| | h5AutoPlay | 试玩打开后自动通关演示 | bool | **false** | **false** |
| 结束页 | endCard | 显示结束页 | bool | true | false（参考无结束页） |
| | endCardHand | 结束页显示手指 | bool | false | false |
| | endDelay | 通关后出现延迟(ms) | 0~5000 | 900 | 600 |
| | endTitle | 标题 | text | | |

> 注：旧版规范表格中 h5AutoPlay 写 true，已被 §5.3 与全部工具最新版本（「试玩由玩家自己玩」）推翻，**以 false 为准**。

工具专有参数放对应分组末尾或单独分组（例：麻将 aeLeaves/export3dParticles/handRotation；螺丝 export3dScrew/autoSelectGap/ratchet*）。

## 3. AE 导出 ZIP

```
build_ae_project.jsx   timeline.json   README_AE.md
assets/<英文id>.png    （AE 中显示名随 aeNames）
audio/mix.wav          audio/sfx_<事件>.wav
```

- 文件名 `<tool>_ae_export_YYYYMMDD_HHMMSS.zip`；`.aep` 名 `meta.aepName = <tool>_<stamp>.aep`。
- `.aep`/`.aepx` 是未公开二进制格式，无法直接生成；正确做法是导出工程包，在 AE（CC 2018+）里「文件→脚本→运行脚本文件」执行 `build_ae_project.jsx`（ES3 语法、自带 JSON 解析、路径相对脚本自身），由 AE 官方脚本 API 建工程后另存 .aep。
- `timeline.json = { meta, folders, assets, comps, mainComp:'main' }`：
  - meta：`tool, format:1, created, fps, width, height, duration, canvas:[w,h], scale(K), sampling, saveAep, aepName, newProject:true, stats, note` + 工具专有字段。
  - folders：root/img/界面/aud/pre 等，名称中/英随 aeNames。
  - 图层：`{name, src:{asset|comp|solid}, in, out, start, label, blend, markers, audioDb, tr:{anchor,position,scale,rotation,opacity}}`；动画属性 `{t:[…],v:[…],hold?}`；图层**从下到上**排列；时间秒、位置像素、缩放百分比。
- 主合成名 `主合成_<工具中文名>`/`Main_<Tool>`；预合成放 pre 文件夹；音频=混合音轨图层+逐事件预合成（每事件一层，start=in=事件时刻），主合成每个音效事件一个标记。
- 固定图层名：手指/Hand、结束页/End card、免责声明/Disclaimer、背景/Background。
- jsx 行为：检查素材缺失（中文提示并中止）、新建工程、建文件夹/合成/图层/关键帧/标记、弹统计、按 saveAep 询问另存；测试钩子 `$.global.__<TOOL>_BUILD_STATS`。

## 4. 3D 导出 ZIP

```
scene.glb  scene.fbx  scene3d.json  blender_import.py  README_3D.md  timeline.json
audio/mix.wav  audio/sfx_*.wav
```

- 复用 AE 记录：以 `aeFps=export3dFps, aeSize=canvas, aeSampling=bake, aeNames=en, aeAudio=both` 重新导出时间轴后生成。
- `scene3d.json`：`tool, format('export3d/1'), fps, frames, frameStart, frameEnd, duration, width, height, units, camera, nodes, visibility, opacity, audio, mix, stats` + 工具专有字段。
- 麻将约定：1 画布像素=1mm，Y 轴向上；显示/隐藏用 STEP 缩放 1/0。FBX 由 Blender 无头转换（环境无 Blender 时可缺 scene.fbx，其余照常）。
- 结束页是否显示手指遵守 endCardHand。

## 5. H5 试玩广告 ZIP

共享实现：各工具 `src/h5/` 拷贝同一份。

### 5.1 包结构

```
<tool>_h5_playable_YYYYMMDD_HHMMSS.zip
  README_H5.md
  applovin/index.html         MRAID 2.0
  facebook/index.html         ≤2MB
  google/index.html
  mintegral/index.html
  moloco/index.html
  tiktok/index.html
  tiktok/config.json          {"playable_orientation":1}
  unity/index.html            MRAID (injected)
  generic/index.html
  generic/README.txt
```

- 只含 h5Format 勾选渠道；默认全部 8 个；旧默认值 `tiktok,google,mintegral,generic` 视为「全部」。
- 每渠道 index.html 为**自包含**单文件：内嵌当前设置（`window.__PLAYABLE__`），不显示设置面板/工具栏，仅游戏画布+CTA 条。
- **多尺寸自适应**：默认 contain/letterbox，保持设计画幅比例居中缩放，letterbox 与背景色一致；全视口 `100vw/100dvh` + safe-area-inset；CTA 底部 sticky 预留 safe-area。
- 单渠道与总 ZIP 尽量 < h5MaxMb（默认 5MB），超限 UI 警告；可 minify、去 AE 录制器逻辑、压缩贴图。

### 5.2 渠道 CTA

| 渠道 | CTA 行为 |
|---|---|
| TikTok/Pangle | `window.openAppStore()`（若存在），否则 `location.href=storeUrl` |
| Google Ads | `ExitApi.exit()`（附 stub/注释）；相对路径；portrait meta |
| Mintegral | `window.install()` → `window.gameEnd()` → `location.href=storeUrl` |
| AppLovin | MRAID 2.0：head 引 MRAID 脚本；等 ready；`mraid.open(storeUrl)`；≤5MB |
| Meta/Facebook | 单 HTML ≤**2MB**，全内联无外部请求；`FbPlayableAd.onCTAClick()` |
| Moloco | 单 HTML <5MB；**不得**引 MRAID 脚本/XHR；`FbPlayableAd.onCTAClick()` |
| Unity Ads | MRAID 由 webview 注入（不加脚本标签）；ready+viewableChange 可见后再开始；`mraid.open`；<5MB |
| Generic | 点击打开 h5StoreUrl，无跳转环；附 README.txt |

### 5.3 试玩壳（playable shell）

- 注入 `window.__PLAYABLE__ = { toolId, format, settings, ctaText, storeUrl, gameName, icon, orientation, autoPlay }`；主程序检测到它则应用 settings、隐藏 #panel、舞台全屏、挂 CTA。
- **禁止**在试玩包暴露完整编辑器（导入/导出设置、AE/3D 按钮、参数面板）。
- meta viewport：`width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover`。
- orientation：1 竖屏/2 横屏/0 双端；双端时横屏设备仍 contain 放下竖屏内容，勿写死仅竖屏。
- h5Device/W/H **预览与导出共用**：adaptive→设计画幅=canvasW×canvasH 全机型 contain；指定机型/custom→设计画幅锁定 W×H 并烘焙进导出设置；编辑器非 adaptive 显示设备预览框。机型分辨率对齐 Chrome DevTools（iPhone 14 Pro Max 430×932、Z Fold 5 外屏 344×882、iPad Pro 13 1032×1376）。
- resize/orientationchange/visualViewport → `window.__h5FitStage()` 重算；`window.__h5StageLayout` 暴露布局；2D 画布靠 CSS 缩放（点击用 getBoundingClientRect）；three.js 工具在 applyPlayableMode({onResize}) 里 setSize+更新相机。
- **试玩由玩家自己玩**：包不得包含/绘制手指（stripHandForPlayable 清空 handImage/handShow/manualHand/endCardHand/hideCursor，stripDataUrls 去掉 bundle 内默认手指 data URL；工具侧 isPlayableNoHand() 守卫）。**h5AutoPlay 默认 false**。工具须提供 state() 与 isEnded()；结束后 CTA 呼吸高亮，点舞台也触发渠道 CTA。
- **注入/剥离只能作用于 `<script>` 之外的标记**（editOutsideScripts）：bundle 内含本模块模板字符串，全文正则会切断主脚本 → 舞台全黑（曾发生事故）。试玩壳不 remove() 编辑器面板（改隐藏），避免自动演示时 updateButtons 报错。
- 测试 `npm run test:h5`：解压 ZIP、每渠道起 http 服务用 Playwright 打开，断言无 console/page error、舞台**非黑**（采样像素）、CTA 钩子存在、无手指、autoPlay 关、鼠标+触屏真实点击改变游戏状态（含 letterbox 缩放）、结束后点舞台/CTA 触发渠道 CTA；多视口冒烟（375×667、390×844、430×932、768×1024、1032×1376、344×882、1024×600）：stage 可见、CTA 不裁切、等比缩放无横向溢出。

### 5.4 共享 API

```js
buildPlayableZip({ toolId, formats, baseHtml, opts })
makePlayableHtml(baseHtml, { format, ...opts })
parseFormats(h5Format)  // → ['tiktok',…]
H5_DEVICE_PRESETS / resolveH5Device(S) / bakeH5DeviceIntoSettings(S)
applyEditorDevicePreview(S) / bindEditorDevicePreview(getS)
```

## 6. 测试矩阵

`npm run test:logic`（逻辑/生成器）、`npm test`（自动通关+截图）、`npm run test:ae`（导出+jsx mock+合成 PSNR）、`npm run test:3d`（Blender 导入+渲染对比）、`npm run test:h5`（ZIP 结构+Playwright 多视口冒烟）。

## 7. 侧面控制台分层（长期规则）

- **外层**只放：基础调整（关卡、速度、颜色、手指、音效等常用项）+ 一切影响动画画面或导出结果的参数（画布尺寸、节奏/时长、思考停顿、录制/AE/3D/H5/结束页参数）。
- **高级设置**：不常用参数（物理系数、碰撞粗细、判定余量、映射 JSON、关卡 JSON、锚点校准、H5 预览尺寸、告警阈值等）在 SCHEMA 标 `extra.adv=true`，统一收进**最后一个、默认折叠**的 `<details id="advSettings">`，内部按原分组小标题排列。
- 分层只改显示位置：标准 id、按钮顺序、设置 key、导出结构不变；外层分组顺序（音效<手指<录制<AE<3D<H5<结束）保持。
- 新增参数先判断层级；改动层级需升 SETTINGS_VERSION 并迁移。
- 「主题」分组位于「关卡与玩法」之上，数据驱动（src/themes.js），切换实时生效并持久化，屏幕/AE/3D/H5 全部使用当前主题；分主题保存外观改动（themeStore），玩法/导出设置跨主题共用。

## 8. 开口环缺口安全区（ring-belt 专有，已并入规范）

- 开口(C 形)环缺口两侧 `缺口半角 + GAP_SAFE_DEG(15°)` 范围内禁止放夹扣/铰接颈；`gapSafeViolations()` 在 test:logic 对全部关卡校验。
- 层级：父环（夹扣同色环）压在子环之上；颈部在父环之下，夹块在父环之下、子环之上。