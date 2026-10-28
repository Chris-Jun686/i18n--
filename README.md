# 文本管理与存档

## 1. 关于文本管理与存档

游戏里给玩家看的每一句话、每个按钮名和道具名，都不要直接写在程序里，而是集中记在一张清单上，程序只记一个固定编号，显示时按编号去清单里取当前语言的文字。一句话要整句记，数字和名字留空由程序填，不能把句子拆开拼。存档时也只记物品、任务等的编号和数量，不记玩家看到的文字，这样换语言后旧存档仍能显示正确语言。

## 2. 前作纰漏

### 2.1 文本表与统一表述

> *以下 3 处同样表达属性提升的文本出现 3 种不同表述。*

<img width="1920" height="1080" alt="LQA-023" src="https://github.com/user-attachments/assets/62f394ed-f964-4bd0-b8ab-6cf120603f3d" />

> 问题编号-23
<img width="1923" height="1127" alt="LQA-026" src="https://github.com/user-attachments/assets/4faa8ec1-c4ea-497c-a876-51bdb3edc40a" />

> 问题编号-28

<img width="1923" height="1127" alt="LQA-028" src="https://github.com/user-attachments/assets/f927be14-96e5-4c12-bb6d-9cc6b15f67f6" />

> 问题编号-30

### 2.2 句子拼接
> *以下句子由于拼接出现句中错误大写、缺少空格、。*
<img width="1920" height="1080" alt="LQA-020" src="https://github.com/user-attachments/assets/78e6c8f9-b846-45f1-bbe8-54fa463d7517" />

> 问题编号-22
<img width="1920" height="1080" alt="LQA-022" src="https://github.com/user-attachments/assets/fa6d5bdb-8c39-46a6-9554-a4a6ab90caad" />

> 问题编号-24

<img width="1920" height="1080" alt="LQA-024" src="https://github.com/user-attachments/assets/7f092f1e-8129-4ef5-aaa9-d22fa08c9de8" />

> 问题编号-26

## 3. 规范

### 3.1 文本表与字符串 ID

1. 所有玩家可见文本必须抽取到文本表：UI、任务、道具名、技能描述、错误提示、按钮、字幕、成就、系统提示等。
2. 代码、UI 预制体、脚本中不得出现玩家可见自然语言字面量；代码只能引用字符串 ID。
3. 文本表每条至少包含：字符串 ID、源文、译文、上下文/备注、长度限制、截图/场景。
4. 字符串 ID 使用稳定格式：`模块_对象_字段_状态`，例如 `ATTR_EMOTION_GAINED`、`UI_BUTTON_CONFIRM`、`QUEST_001_OBJECTIVE_01`、`ERROR_NETWORK_TIMEOUT`。
5. 字符串 ID 必须可读、稳定、不复用、不包含译文内容；不要用 `text1`、`str_023`，也不要把“情绪增加了{count}”或 `Emotion increased by {count}` 当 ID。
6. 已发布的字符串 ID 不得改名或复用；新增文本必须新增 ID。
7. 同一句话在多处使用时，必须引用同一个字符串 ID，不得复制多份文本。
8. 文本表中含动态数值的条目必须使用命名占位符，例如 `{count}`；不得把示例数值写死进源文。
9. 代码必须按占位符名称传参，不得用字符串相加把数值拼进句子；不得假设占位符顺序。
10. 文本表是唯一文本来源；修改文案只改文本表，不改代码。

### 3.2 句子与占位符

1. 玩家可见句子必须整句放进文本表，使用完整句子模板和命名占位符。
2. 禁止用字符串相加、条件判断、前后缀拼接生成玩家可见自然语言。
3. 禁止对译文做 `+ "s"`、截取、替换词序等语言相关操作。
4. 占位符必须命名，例如 `{count}`、`{itemName}`、`{time}`；占位符顺序必须允许翻译人员调整。
5. 数量、名字、时间、类型等动态内容必须通过占位符传入；占位符中的名称若玩家可见，也必须用文本 ID 查当前语言，不得传硬编码中文。
6. 单复数、性别、选择必须使用 ICU MessageFormat 或本地化平台等价机制；不得在代码里写 `count == 1 ? "message" : "messages"`。
7. 文本表每条句子必须填写上下文/备注，说明占位符含义和示例值。
8. 禁止隐蔽拼接，例如 `"New " + itemType + " available"`、`"Player " + name + " has joined"`、`"剩余" + time + "秒"`。
9. 拼接后只用于内部日志或协议字段的，不得直接显示给玩家。

### 3.3 哪些文本不翻译、不进文本表

1. 内部调试日志、协议字段、正则表达式、文件路径等非玩家文本不进入文本表，不翻译。
2. 玩家自定义名称如需保存，必须单独字段保存原文，并明确不翻译；不得与系统物品名、任务名混用。
3. 玩家自定义名称可以作为占位符传入已翻译的句子模板，但名称本身不得作为文本表条目被翻译。
4. 非玩家文本即使包含中文，也不得为了“统一”而放进文本表并送翻。

### 3.4 存档

1. 存档只存稳定 ID、数值、状态、布尔值、时间戳等；不得存玩家可见文本、源文、译文、显示名、描述。
2. 物品存档存 `itemId` 和 `count`；任务存档存 `questId`、目标索引和状态；成就存档存 `achievementId`；技能存档存 `skillId` 和 `level`。
3. 显示存档内容时必须用 ID 查配置表或文本表得到当前语言文本；不得把存档里的文字直接显示。
4. 存档不得存拼接后的句子、格式化后的错误提示、带语言的名称。
5. 存档中的状态值使用稳定枚举或稳定 ID，例如 `Completed`，显示时再查 `QUEST_STATE_COMPLETED` 对应文本。
6. ID 发布后不得修改或复用；必须修改时提供旧 ID 到新 ID 的迁移映射。
7. 文本修订只改文本表，不改存档；切换语言只重新查表，不迁移存档文字。

## 4. 正确与错误示范

### 4.1 文本表、字符串 ID 与动态数值

**示例 1：属性提升提示**

- 中文原文：情绪增加了3（其中 `3` 为动态数值，实际显示值由运行时传入）
- 错误写法一：把整句连同数值写死在代码里。

```csharp
emotionText.text = "情绪增加了3";
```

- 错误写法二：把固定文案和动态数值用字符串相加拼起来。

```csharp
emotionText.text = "情绪增加了" + count;
```

- 正确写法：

文本表：

| 字符串 ID | 源文 | 译文 | 上下文/备注 | 长度限制 | 截图/场景 |
|---|---|---|---|---|---|
| `ATTR_EMOTION_GAINED` | 情绪增加了{count} | Emotion increased by {count} | 属性提升提示，角色属性面板显示；`{count}` 为本次增加的情绪值，整数，示例值 3 | 源文 20 字符；译文 40 字符 | 角色属性面板 |

代码：

```csharp
emotionText.text = Localization.Get("ATTR_EMOTION_GAINED", count);
```

其中 `count` 为运行时传入的数值，例如 3。文本表保存完整句子模板，翻译人员可调整 `{count}` 在句中的位置；代码只按占位符名称传参，不拼接句子。

**示例 2：按钮文字与字符串 ID 命名**

- 中文原文：确认
- 错误写法：代码写死文字，或使用不可读 ID。

```csharp
confirmButton.text = "确认";
```

```text
text1 = 确认
str_023 = 确认
确认 = 确认
```

- 正确写法：

```csharp
confirmButton.text = Localization.Get("UI_BUTTON_CONFIRM");
```

**示例 3：任务目标**

- 中文原文：击败黑风寨山贼
- 错误写法：

```csharp
objectiveText.text = "击败黑风寨山贼";
```

- 正确写法：

```csharp
objectiveText.text = Localization.Get("QUEST_001_OBJECTIVE_01");
```

### 4.2 不拼接句子

**示例 1：获得物品**

- 中文原文：少侠获得了 3 柄铁剑
- 错误写法：

```csharp
msg = "少侠获得了" + count + "柄" + itemName;
```

- 正确写法：

文本表：

```text
MSG_ITEM_GAINED = 少侠获得了{count}柄{itemName}
```

英文文本表：

```text
MSG_ITEM_GAINED = You obtained {count} {itemName}.
```

代码：

```csharp
Localization.Get("MSG_ITEM_GAINED", count, Localization.Get("ITEM_SWORD_IRON_NAME"));
```

**示例 2：飞鸽传书与单复数**

- 中文原文：你还有 5 封飞鸽传书
- 错误写法：

```csharp
msg = "You have " + count + " new " + (count == 1 ? "letter" : "letters");
```

- 正确写法：

中文文本表：

```text
MSG_NEW_LETTERS = 你还有{count}封飞鸽传书
```

英文文本表使用 ICU MessageFormat：

```text
MSG_NEW_LETTERS = {count, plural,
  =0 {You have no new letters.}
  one {You have # new letter.}
  other {You have # new letters.}
}
```

代码：

```csharp
Localization.Get("MSG_NEW_LETTERS", count);
```

**示例 3：倒计时提示**

- 中文原文：还需 10 秒
- 错误写法：

```csharp
msg = "剩余" + time + "秒";
```

- 正确写法：

文本表：

```text
MSG_COOLDOWN_REMAIN = 还需{time}秒
```

英文文本表：

```text
MSG_COOLDOWN_REMAIN = {time} seconds remaining
```

代码：

```csharp
Localization.Get("MSG_COOLDOWN_REMAIN", time);
```

### 4.3 哪些文本不翻译

**示例 1：内部调试日志**

- 中文原文（内部调试）：玩家坐标 (10, 3)
- 错误写法：把内部日志放进文本表并送翻。

```text
DEBUG_PLAYER_POS = 玩家坐标{pos}
```

- 正确写法：内部日志保留在代码中，不进入文本表，不翻译。

```csharp
Debug.Log($"玩家坐标 {pos}");
```

**示例 2：玩家自定义名称**

- 中文原文：风清扬加入了队伍
- 错误写法：把玩家名“风清扬”当成文本表源文翻译。

```text
MSG_PLAYER_JOINED = 风清扬加入了队伍
```

- 正确写法：只翻译句子模板，玩家名单独保存原文、不翻译，作为占位符传入。

中文文本表：

```text
MSG_PLAYER_JOINED = {playerName}加入了队伍
```

英文文本表：

```text
MSG_PLAYER_JOINED = {playerName} joined the party.
```

代码：

```csharp
Localization.Get("MSG_PLAYER_JOINED", playerName);
```

### 4.4 存档只存 ID、不存文字

**示例 1：背包物品**

- 中文原文：铁剑
- 错误写法：存档保存显示文字。

```json
{ "itemName": "铁剑", "count": 1 }
```

- 正确写法：存档保存物品 ID 和数量，显示时查文本表。

```json
{ "itemId": "ITEM_SWORD_IRON", "count": 1 }
```

显示时：

```csharp
itemName.text = Localization.Get("ITEM_SWORD_IRON_NAME");
```

或通过物品配置把 `ITEM_SWORD_IRON` 映射到名称文本 ID `ITEM_SWORD_IRON_NAME`，再查当前语言文本。这样切换到英文后，旧存档会显示 `Iron Sword`，不会继续显示“铁剑”。

**示例 2：任务目标与状态**

- 中文原文：击败黑风寨山贼；状态：已完成
- 错误写法：存档保存任务目标和中文状态。

```json
{ "objective": "击败黑风寨山贼", "state": "已完成" }
```

- 正确写法：存档保存任务 ID、目标索引和稳定状态值，显示时查文本表。

```json
{ "questId": "QUEST_001", "objectiveIndex": 1, "state": "Completed" }
```

显示时：

```csharp
objectiveText.text = Localization.Get("QUEST_001_OBJECTIVE_01");
stateText.text = Localization.Get("QUEST_STATE_COMPLETED");
```
