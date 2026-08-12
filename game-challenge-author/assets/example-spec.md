# Dodge the Creeps!（躲避怪物）— 游戏开发题目

实现前请通读全文。规格以第 4 节为准，验收以第 5 节为准。若测试者指定的引擎是 Godot，参考附录 A；其他引擎请自行映射第 4 节规格。

---

## 1. 游戏概述

2D 俯视视角的无尽躲避游戏（竖屏）。玩家控制角色在屏幕内自由移动，躲避从屏幕边缘不断涌出的怪物，活得越久得分越高；被怪物碰到即游戏结束。目标画面参考 `img/dodge_preview.gif`。

- 单人游戏，无胜利条件，分数持续累计
- 全部游戏资产已提供（见第 3 节），**不得**联网下载任何资源
- 所有界面文本为英文，原文见各节（如 "Dodge the Creeps!"、"Game Over"、"Get Ready!"、"Start"）

## 2. 交付要求

- 在 `src/<引擎名>/` 下创建完整游戏项目（引擎由测试者指定，如 `godot`、`unity`；目录名小写、多词用 `-` 连接）
- 打开项目即可直接运行，不依赖外部下载
- 把 `docs/assets/` 中的资产复制进项目使用；`*.import` 等编辑器元数据文件**不要**复制、不要依赖
- 工程/项目名: Dodge the Creeps
- 启动时进入标题画面，**不自动开始游戏**

## 3. 资产清单（`docs/assets/`）

| 文件 | 用途 |
|---|---|
| `art/playerGrey_walk1.png`、`art/playerGrey_walk2.png` | 玩家 "walk" 动画两帧 |
| `art/playerGrey_up1.png`、`art/playerGrey_up2.png` | 玩家 "up" 动画两帧 |
| `art/enemyWalking_1.png`、`art/enemyWalking_2.png` | 怪物 "walk" 动画两帧 |
| `art/enemySwimming_1.png`、`art/enemySwimming_2.png` | 怪物 "swim" 动画两帧 |
| `art/enemyFlyingAlt_1.png`、`art/enemyFlyingAlt_2.png` | 怪物 "fly" 动画两帧 |
| `art/gameover.wav` | 玩家死亡音效 |
| `art/House In a Forest Loop.ogg` | 背景音乐（须循环播放） |
| `fonts/Xolonium-Regular.ttf` | UI 字体（连同 `fonts/LICENSE.txt` 一并复制） |

## 4. 游戏规格

### 4.1 画面与窗口

| 参数 | 值 |
|---|---|
| 视图尺寸 | 480 × 720（竖屏） |
| 窗口缩放 | 窗口尺寸可变，画面按比例缩放适配，始终保持宽高比、不变形 |
| 背景 | 全屏纯色（颜色自定，参考实现为深青色） |

### 4.2 输入

| 动作 | 按键 |
|---|---|
| 移动 | 方向键（建议同时支持 WASD） |
| 开始游戏 | Enter（同时支持点击 Start 按钮） |

### 4.3 玩家

| 参数 | 值 |
|---|---|
| 移动速度 | 400 px/s |
| 出生点 | (240, 450) |
| 动画 | "walk" 与 "up" 两套，各 2 帧，循环播放 |
| 动画播放速度 | 5 FPS |

规则：

- 8 方向移动；同时按两个方向时**速度必须归一化**，禁止斜向加速
- 位置限制在屏幕内（0..480, 0..720），不可越界
- 动画规则：水平移动 → "walk"（向左时水平翻转）；垂直移动 → "up"（向下时垂直翻转）；静止时停止播放动画
- 玩家是"被撞方"（触发器式碰撞），被怪物碰到即判定碰撞
- 默认隐藏；游戏开始时出现在出生点
- 被碰到后：玩家立即隐藏，通知主场景"游戏结束"；该事件每局**只能触发一次**（碰撞须在触发后禁用）
- 新一局开始：回到出生点、重新显示、恢复碰撞

### 4.4 怪物

| 参数 | 值 |
|---|---|
| 移动方式 | 直线匀速飞行（刚体式，不受重力） |
| 速度 | 150 ~ 250 px/s（每只均匀随机） |
| 外观 | "fly" / "swim" / "walk" 三套动画，每只随机选一套 |
| 动画速度 | 3 FPS |
| 生成位置 | 屏幕四条边的边界线上均匀随机取一点 |
| 生成朝向 | 垂直于所在边缘且指向屏幕内部的方向，再 ±45° 随机偏转 |
| 生成间隔 | 0.5 s |
| 生命周期 | 完全离开屏幕后自动删除 |

规则：

- 怪物之间**互不碰撞**，互相穿过
- 开局"准备期"（见 4.5 第 3 步）不生成怪物
- 新一局开始时清空场上所有怪物
- 玩家被撞后怪物不会被销毁，继续飞行直至离开屏幕

### 4.5 主场景与游戏流程

| 参数 | 值 |
|---|---|
| 准备时长 | 2 s（一次性计时） |
| 计分间隔 | 1 s，每次 +1 |
| 怪物生成间隔 | 0.5 s |

流程：

1. **标题画面**（启动时）：显示游戏名 "Dodge the Creeps!" 与 Start 按钮，分数显示 0，玩家隐藏，不生成怪物、不计分
2. 点击 Start 或按 Enter → **开始新一局**：Start 按钮隐藏，玩家出现在出生点，显示 "Get Ready!"，倒计时 2 s（此期间玩家可以移动）
3. 倒计时结束 → 每 0.5 s 生成一只怪物、每 1 s 计分 +1，分数实时显示
4. 玩家被怪物碰到 → **游戏结束**：停止生成与计分；玩家保持隐藏；显示 "Game Over" 2 s；背景音乐停止、播放死亡音效；分数保持显示最终值
5. "Game Over" 消失后重新显示游戏名，再停顿 1 s 后重新显示 Start 按钮 → 回到标题画面（场上剩余怪物继续飞行）
6. 再次开始新一局：分数归零、清空场上怪物、重新播放背景音乐

### 4.6 HUD（UI 覆盖层，绘制在游戏画面之上）

- 分数：屏幕顶部居中，初始 "0"，随计分实时更新
- 消息：屏幕中央，显示 "Dodge the Creeps!" / "Get Ready!" / "Game Over"；显示 2 s 后自动消失
- Start 按钮：屏幕底部居中，文本 "Start"，尺寸约 200×100
- 所有文本使用 `Xolonium-Regular.ttf`，字号 64（可随布局等比调整）
- 游戏结束时分数保持显示最终值

### 4.7 音频

- 背景音乐：`House In a Forest Loop.ogg`，**循环播放**；开始新一局时播放，游戏结束时停止
- 死亡音效：`gameover.wav`；游戏结束时播放一次

## 5. 验收标准（自测清单）

实现完成后运行游戏，逐项确认，全部通过才算完成：

- [ ] 启动后进入标题画面：显示 "Dodge the Creeps!" 与 Start 按钮，分数为 0，不自动开始游戏
- [ ] 点击 Start 或按 Enter 开始：按钮隐藏、玩家出现在 (240, 450)、显示 "Get Ready!" 2 秒
- [ ] 方向键（及 WASD）8 向移动正常；斜向不加速；不能移出屏幕；动画与翻转方向正确
- [ ] 2 秒后怪物开始每 0.5 s 生成：从屏幕边缘出现、朝向屏幕内部、速度 150~250 px/s、外观随机
- [ ] 分数每 1 秒 +1 并在 HUD 实时更新
- [ ] 怪物互不碰撞；离开屏幕后自动删除
- [ ] 玩家碰到怪物：玩家消失、生成与计分停止、显示 "Game Over"、BGM 停止、播放死亡音效
- [ ] "Game Over" 2 秒后显示游戏名，再 1 秒后 Start 按钮重新出现
- [ ] 再次开始：分数归零、场上怪物被清空、BGM 重新播放
- [ ] BGM 无缝循环；完整游玩流程无报错

## 6. 实现提示（引擎无关）

- 玩家用"区域/触发器"式碰撞（被撞方），怪物用"刚体"式碰撞（撞击方）
- 怪物刚体必须禁用重力
- 若引擎不允许在物理回调中直接修改碰撞属性（如禁用碰撞），按引擎惯例延迟处理，避免报错
- 怪物沿边缘生成时：方向 = 边缘切线方向旋转 90°（指向屏幕内）再 ±45° 随机
- 清空场上怪物用"分组/标签"机制统一删除，不要逐个遍历引用
- 角色显示比例：资产原图约 100~135 px，参考实现按 0.5 倍缩放（玩家显示约 54×68、怪物约 50~75 px），碰撞范围与显示尺寸相当；非 Godot 引擎按此比例自行缩放
- 若引擎/目标平台不支持 ogg 音频（如 MonoGame 默认管线、Safari 的 Web Audio），可转换为引擎支持的格式，但背景音乐必须保持循环播放
- 主场景必须设为启动场景（打开项目直接进入游戏）

## 附录 A：Godot 实现参考

> 以下为 Godot 4.x 的推荐结构（参考实现：Godot 4.7.1 / C#，GDScript 亦可）。使用其他引擎时忽略本附录。

### A.1 场景树

```
Player (Area2D)
├── AnimatedSprite2D        # SpriteFrames: walk=playerGrey_walk1/2, up=playerGrey_up1/2, 动画速度 5；Scale (0.5, 0.5)
└── CollisionShape2D        # CapsuleShape2D: radius 27, height 70

Mob (RigidBody2D)           # 加入组 "mobs"；gravity_scale = 0；collision_mask = 0（互不碰撞）
├── AnimatedSprite2D        # SpriteFrames: fly/swim/walk 各 2 帧, 动画速度 3；Scale (0.5, 0.5)
├── CollisionShape2D        # CapsuleShape2D: radius 24, height 68；Rotation 90°
└── VisibleOnScreenNotifier2D  # screen_exited → queue_free

Main (Node)                 # 启动场景
├── ColorRect               # 全屏 (Full Rect) 纯色背景
├── Player                  # player.tscn 实例
├── MobTimer    (Timer)     # wait_time 0.5
├── ScoreTimer  (Timer)     # wait_time 1
├── StartTimer  (Timer)     # wait_time 2, one_shot
├── StartPosition (Marker2D)  # position (240, 450)
├── MobPath (Path2D)        # 顺时针闭合矩形，顶点 (0,0)→(480,0)→(480,720)→(0,720)，可外扩 2~3 px
│   └── MobSpawnLocation (PathFollow2D)
├── HUD                     # hud.tscn 实例
├── Music (AudioStreamPlayer)      # House In a Forest Loop.ogg（Stream 须设 Loop 开启）
└── DeathSound (AudioStreamPlayer) # gameover.wav

HUD (CanvasLayer)
├── ScoreLabel  (Label)     # Xolonium 64，顶部居中，text "0"
├── Message     (Label)     # Xolonium 64，屏幕中央，Autowrap=Word，text "Dodge the Creeps!"
├── StartButton (Button)    # Xolonium 64，底部居中 200×100，text "Start"，快捷键 Enter
└── MessageTimer (Timer)    # wait_time 2, one_shot
```

### A.2 输入映射（project.godot）

- `move_left`=←（可加 A）、`move_right`=→（可加 D）、`move_up`=↑（可加 W）、`move_down`=↓（可加 S）
- `start_game`=Enter
- `run/main_scene` 指向 main.tscn；窗口 480×720、stretch mode=canvas_items、aspect=keep

### A.3 脚本要点

- **Player**：`@export var speed = 400`；自定义信号 `hit`；`_ready()` 中取屏幕尺寸并 `hide()`；`_process()` 用 `Input.is_action_pressed()` 合成方向 → `normalized() * speed` → `position += velocity * delta` → `clamp(Vector2.ZERO, screen_size)`；`body_entered` → `hide()` + `hit.emit()` + `$CollisionShape2D.set_deferred("disabled", true)`；`start(pos)` 重置位置/显示/碰撞
- **Mob**：`_ready()` 从 `sprite_frames.get_animation_names()` 随机选一个动画并 `play()`；`screen_exited` → `queue_free()`
- **Main**：`_on_mob_timer_timeout()`：实例化 mob → `mob_spawn_location.progress_ratio = randf()` → `direction = mob_spawn_location.rotation + PI / 2 + randf_range(-PI / 4, PI / 4)` → `mob.rotation = direction` → `linear_velocity = Vector2(randf_range(150.0, 250.0), 0).rotated(direction)` → `add_child(mob)`；`new_game()` 中调用 `get_tree().call_group("mobs", "queue_free")` 清场
- **HUD**：`show_message(text)` 显示消息并启动 MessageTimer；`show_game_over()`：显示 "Game Over" → `await MessageTimer.timeout` → 改显示 "Dodge the Creeps!" → `await get_tree().create_timer(1.0).timeout` → 显示 StartButton

### A.4 信号连接

| 信号 | 接收方法 |
|---|---|
| Player.`hit` → Main | `game_over` |
| HUD.`start_game` → Main | `new_game` |
| StartTimer/ScoreTimer/MobTimer.`timeout` → Main | `_on_start_timer_timeout` / `_on_score_timer_timeout` / `_on_mob_timer_timeout` |
| StartButton.`pressed` → HUD | 隐藏按钮 + 发出 `start_game` |
| MessageTimer.`timeout` → HUD | 隐藏消息 |
| Mob 的 VisibleOnScreenNotifier2D.`screen_exited` → Mob | `queue_free` |
