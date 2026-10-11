# 鲸鱼娘的空袭任务 · CETACEA AIR STRIKE

一个可直接在浏览器中运行的 **3D 空战轰炸小游戏**（键盘 + 鼠标操作）。驾驶战斗机在城市上空自由飞行，用机炮扫射、用炸弹轰炸，看建筑在爆炸中倒塌——撞上建筑或地面则会坠毁。

纯前端、单文件、全中文、资源全部程序化生成，双击 HTML 打开即玩。

---

## 快速开始

1. 用浏览器打开 `game.html`（桌面键鼠操作）
2. 点击「开始任务」
3. W / S 俯仰升降，A / D 横滚转向；按住鼠标左键扫射；按住右键显示预瞄环、松开投弹；滑动鼠标拖动视角；Alt 视角回正，左 Shift 加力，Tab 切换鼠标显隐

> 无需安装、无需构建、无后端。仅 Three.js 通过 CDN 加载，首次运行需联网。

---

## 操作方式

| 操作 | 控件 |
| --- | --- |
| 俯仰升降 | W / S |
| 横滚转向 | A / D |
| 机炮扫射 | 按住鼠标左键 |
| 轰炸 | 按住鼠标右键显示预瞄环，松开鼠标右键投弹 |
| 视角旋转 | 滑动鼠标 |
| 视角回正 | 按 Alt 键 |
| 加力加速 | 按住左 Shift（速度 ×1.6，尾焰更长更亮、引擎声更响） |
| 光标显隐 | 按 Tab 切换（进入任务自动隐藏，结算时恢复） |

- **摧毁建筑**：机枪命中 10 次，或炸弹在爆炸半径内命中
- **坠毁**：撞到建筑或地面后进入坠毁动画，结算得分
- **HUD**：实时显示得分 / 击毁建筑数 / 飞行高度
- **振动反馈**：开火、投弹、爆炸、撞击（设备支持时）

---

## 游戏内容

| 元素 | 说明 |
| --- | --- |
| 地形 | 4400×4400 网格，解析函数生成，颜色随高度渐变（水底→沙滩→草地→岩地→雪线） |
| 山脉 | 地图西北部，峰高约 +620 |
| 城市 | 网格街区布局，中心高楼、边缘低矮，蓝系楼体 + 暖色屋顶卡通配色 |
| 河流 | 沿 z 轴蜿蜒贯穿地图 |
| 湖泊 | 地图西南部，带谐波扰动的不规则岸线 |
| 水面 | 半透明材质，高度 4 |
| 植被 | 约 460 棵松树（InstancedMesh 批量渲染，自动避开城市 / 水面 / 高地） |
| 可摧毁建筑 | 机炮累计命中 10 次或炸弹波及即摧毁，带碎片飞散 |
| 机炮准星 | 按住左键扫射时在弹着点显示十字准星（`updateGunAim`，随距离缩放） |
| 特效 | 闪光、火球、冲击波环、火花、烟雾、建筑碎片（自动生命周期管理） |
| 音效 | 引擎（随俯仰变调）、机炮、爆炸，全部 Web Audio 实时合成 |

---

## 技术特性

- **单文件**：`game.html` 内嵌全部 CSS / JS，代码用 IIFE 包裹，全局仅暴露来自 CDN 的 `THREE`
- **CDN 依赖**：`https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js`（版本锁定）
- **零外部资源**：模型用基本几何体拼装，地形为解析函数，音效为 Web Audio 合成，不加载任何图片 / 音频 / 模型文件
- **性能优化**：`antialias:false`、`pixelRatio ≤ 1.5`、InstancedMesh 批量渲染植被、空间哈希加速建筑碰撞查询
- **键鼠操作**：WASD 飞行 + 鼠标（左键机炮 / 右键瞄准投弹 / 滑动视角 / Alt 回正），左 Shift 加力、Tab 切换鼠标光标，按住生效、松开停止
- **视觉风格**：低多边形简约卡通（MeshLambertMaterial 平面着色），与写实风格明确区分

---

## 关键参数

### 飞行与物理（`CFG`）

| 参数 | 值 | 说明 |
| --- | --- | --- |
| `speed` | 68 | 巡航速度 |
| `rollRate` | 1.9 | 横滚速率 |
| `pitchRate` | 1.35 | 俯仰速率 |
| `yawRate` | 0.75 | 横滚引起的偏航 |
| `gravity` | 48 | 炸弹重力 |
| `bombCD` | 0.6 | 投弹冷却（秒） |
| `fireCD` | 0.085 | 开火冷却（秒） |
| `bulletSpd` | 560 | 子弹初速 |
| `mapHalf` | 1850 | 地图半径边界 |
| `ceil` | 1500 | 升限 |
| `planeR` | 6 | 飞机碰撞球半径 |
| `BOOST_MULT` | 1.6 | 左 Shift 加力倍率（速度、尾焰、引擎声同步强化） |

### 战斗

| 常量 | 值 | 说明 |
| --- | --- | --- |
| `GUN_KILL_HITS` | 10 | 机枪摧毁建筑所需命中次数 |
| `BOMB_RADIUS` | 195 | 炸弹爆炸半径（预瞄环同步此值） |
| `WATER_LEVEL` | 4 | 水面高度 |

### 相机与视角

| 项目 | 值 |
| --- | --- |
| 相机 | `PerspectiveCamera(62, aspect, 1, 12000)`，机位：后方 -30 / 上方 9.5 |
| 视角偏移上限 | 水平 ±1.35，向下 -0.85，向上 1.05 |
| 鼠标灵敏度 | `MOUSE_SENS = 0.0042` rad/px（滑动鼠标拖动视角） |
| 视角回正 | 按 Alt，偏移归零（`resetView()`） |

---

## 代码结构（game.html）

按功能分块，主要函数一览：

| 模块 | 函数 |
| --- | --- |
| 配置 | `CFG` 对象、`GUN_KILL_HITS` / `BOMB_RADIUS` / `WATER_LEVEL` 常量、全局状态 `G` |
| 地形 | `terrainHeight` / `terrainColor` / `createTerrain` / `createWater` |
| 场景 | `createSky` / `createClouds` / `createVegetation` |
| 城市 | `createCity` / `makeBuilding`，空间哈希 `addToHash` / `removeFromHash` / `queryBuildings` |
| 建筑受击 | `hitByBullet`（累计命中） / `destroyBuilding` / `spawnDebris`（碎片飞散） |
| 飞机 | `createPlane` / `resetPlane`，碰撞 `checkPlaneCollision` / `sphereHitsBuilding` |
| 武器 | `fireBullet` / `updateBullets` / `dropBomb` / `onBombImpact` |
| 特效 | `spawnExplosion` / `spawnSmoke` / `spawnHitSpark` / `spawnMuzzleFlash` / `updateEffects` |
| 音效 | `Sound`（引擎 / 机炮 / 爆炸合成，`init` / `setEngine`） |
| 操作 | `bindKeyboardMouse`（WASD / 鼠标左右键 / 滑动视角 / Alt / 左 Shift / Tab），`resetView`、`setCursorHidden` |
| 相机 | `updateCamera` / `getForward`，轰炸预瞄环 `updateAim`，机炮准星 `updateGunAim` |
| 流程 | `startGame` / `startCrash` / `updateCrash` / `showGameOver` / `updateHUD` |
| 主循环 | `update(dt)` / `animate` / `init` / `onResize` |

---

## 开发说明

**锁定参数**：`CFG` 全部字段以及 `GUN_KILL_HITS`、`BOMB_RADIUS`、`WATER_LEVEL` 均经确认后锁定，请勿擅自改动；如需调整手感或数值，先与需求方确认。

**调试入口**：

- 所有数值集中在文件顶部 `CFG` 与三个常量
- 地形高度统一走 `terrainHeight(x, z)`，改地形只改这一个函数
- 键鼠输入逻辑在 `bindKeyboardMouse()`（含左 Shift 加力、Tab 光标切换），相机偏移应用在 `updateCamera()`，视角回正在 `resetView()`
- 加力倍率集中在 `BOOST_MULT`（速度 / 尾焰 / 引擎声共用）；机炮准星预测在 `updateGunAim()`
- 预瞄环更新在 `updateAim()`；炸弹威力问题先查 `BOMB_RADIUS` 与 `spawnExplosion(pos, 3.6)` 的第二个参数

**历史决策（不要回退）**：

- 「拖拽屏幕任意位置调视角」已废弃；触屏按钮 + 摇杆也已替换，现为键盘鼠标操作版本（WASD + 鼠标，Alt 回正）
- 「窗户贴图写实建筑」已废弃，与卡通飞机风格不符
- 「任务目标建筑」已移除，现为自由飞行、自由破坏

**已知限制 / 可扩展方向**：碎片只受重力、不互相碰撞；当前为键鼠版本，暂未保留触屏控件（旧触屏版备份在 `game-touch.html`）；可扩展得分飘字、连击奖励、昼夜光照、额外武器、敌机等。

---

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `game.html` | 游戏本体（键盘鼠标操作版），单文件，可直接运行 |
| `game-touch.html` | 旧版触屏操作（按钮 + 视角摇杆），转换前备份，保留备用 |
| `README.md` | 本文件，项目说明 |
| `交接.md` | 开发交接文档（对话历史与上下文简报，供接手者阅读） |
