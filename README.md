## 1. 文本

### 1.1 定义

玩家能看到的文字，例如按钮、道具名、任务目标、技能描述、错误提示、字幕、成就说明，不能直接写死在程序代码、UI 预制体或脚本里。要把这些文字集中放进一张“文本表”，程序只记一个固定编号（字符串 ID），显示时按编号取当前语言的文字。这样换语言时只换文本表，不改程序。

### 1.2 前作纰漏
<img width="1923" height="1127" alt="LQA-026" src="https://github.com/user-attachments/assets/4faa8ec1-c4ea-497c-a876-51bdb3edc40a" />
<img width="1923" height="1127" alt="LQA-028" src="https://github.com/user-attachments/assets/f927be14-96e5-4c12-bb6d-9cc6b15f67f6" />
<img width="1920" height="1080" alt="LQA-023" src="https://github.com/user-attachments/assets/62f394ed-f964-4bd0-b8ab-6cf120603f3d" />
>3处同样表达属性提升的文本出现3种不同的表述

### 1.3 规范

1. 所有玩家可见文本必须抽取到文本表：UI、任务、道具名、技能描述、错误提示、按钮、字幕、成就、系统提示等。
2. 代码、UI 预制体、脚本中不得出现玩家可见自然语言字面量；代码只能引用字符串 ID。
3. 文本表每条至少包含：字符串 ID、源文、译文、上下文/备注、长度限制、截图/场景。
4. 字符串 ID 使用稳定格式：`模块_对象_字段_状态`，例如 `ITEM_SWORD_IRON_NAME`、`UI_BUTTON_CONFIRM`、`QUEST_001_OBJECTIVE_01`、`ERROR_NETWORK_TIMEOUT`。
5. 字符串 ID 必须可读、稳定、不复用、不包含译文内容；不要用 `text1`、`str_023`，也不要把“铁剑”或 `Iron Sword` 当 ID。
6. 已发布的字符串 ID 不得改名或复用；新增文本必须新增 ID。
7. 同一句话在多处使用时，必须引用同一个字符串 ID，不得复制多份文本。
8. 非玩家文本不进文本表：内部调试日志、协议字段、正则表达式、文件路径等。
9. 文本表是唯一文本来源；修改文案只改文本表，不改代码。

### 1.4 正确以及错误示例

**示例 1：道具名**

- 中文原文：铁剑
- 错误写法：

```csharp
itemName.text = "铁剑";
```

- 正确写法：

文本表：

| 字符串 ID | 源文 | 译文 | 上下文/备注 | 长度限制 | 截图/场景 |
|---|---|---|---|---|---|
| `ITEM_SWORD_IRON_NAME` | 铁剑 | Iron Sword | 物品名称，背包列表显示 | 20 字符 | 背包界面 |

代码：

```csharp
itemName.text = Localization.Get("ITEM_SWORD_IRON_NAME");
```

**示例 2：按钮文字**

- 中文原文：确认
- 错误写法：

```csharp
confirmButton.text = "确认";
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

---

## 2. 句子

### 2.1 定义

一句给玩家看的话不能拆成“你获得了”“3 个”“铁剑”这种碎片再用代码拼起来。不同语言的语序、复数、量词、空格和标点都不同；应该把整句话放进文本表，用占位符留空，程序只填数量、名字、时间等参数。翻译人员必须看到完整句子，才能调整语序。

### 2.2 前作纰漏

<img width="1920" height="1080" alt="LQA-020" src="https://github.com/user-attachments/assets/78e6c8f9-b846-45f1-bbe8-54fa463d7517" />
<img width="1920" height="1080" alt="LQA-022" src="https://github.com/user-attachments/assets/fa6d5bdb-8c39-46a6-9554-a4a6ab90caad" />
<img width="1920" height="1080" alt="LQA-024" src="https://github.com/user-attachments/assets/7f092f1e-8129-4ef5-aaa9-d22fa08c9de8" />
> 由于拼接句子造成句中出现错误大写/缺少空隔

### 2.3 规范

1. 玩家可见句子必须整句放进文本表，使用完整句子模板和命名占位符。
2. 禁止用字符串相加、条件判断、前后缀拼接生成玩家可见自然语言。
3. 禁止对译文做 `+ "s"`、截取、替换词序等语言相关操作。
4. 占位符必须命名，例如 `{count}`、`{itemName}`、`{time}`；占位符顺序必须允许翻译人员调整。
5. 代码按占位符名称传参，不得假设占位符顺序。
6. 数量、名字、时间、类型等动态内容必须通过占位符传入；占位符中的名称若玩家可见，也必须用文本 ID 查当前语言，不得传硬编码中文。
7. 单复数、性别、选择必须使用 ICU MessageFormat 或本地化平台等价机制；不得在代码里写 `count == 1 ? "message" : "messages"`。
8. 文本表每条句子必须填写上下文/备注，说明占位符含义和示例值。
9. 禁止隐蔽拼接，例如 `"New " + itemType + " available"`、`"Player " + name + " has joined"`、`"剩余" + time + "秒"`。
10. 拼接后只用于内部日志或协议字段的，不得直接显示给玩家。

### 2.4 正确以及错误示例

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

**示例 2：新消息/飞鸽传书**

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

---

## 3. 存档

### 3.1 定义

存档只记录“这是哪个物品、任务、成就”的固定编号，以及数量、状态等数据，不记录玩家看到的文字。显示时用编号按当前语言查文本表。否则玩家切换到英文后，旧存档里存的中文“铁剑”还会继续显示“铁剑”。

### 3.2 规范

1. 存档只存稳定 ID、数值、状态、布尔值、时间戳等；不得存玩家可见文本、源文、译文、显示名、描述。
2. 物品存档存 `itemId` 和 `count`；任务存档存 `questId`、目标索引和状态；成就存档存 `achievementId`；技能存档存 `skillId` 和 `level`。
3. 显示存档内容时必须用 ID 查配置表或文本表得到当前语言文本；不得把存档里的文字直接显示。
4. 存档不得存拼接后的句子、格式化后的错误提示、带语言的名称。
5. 存档中的状态值使用稳定枚举或稳定 ID，例如 `Completed`，显示时再查 `QUEST_STATE_COMPLETED` 对应文本。
6. ID 发布后不得修改或复用；必须修改时提供旧 ID 到新 ID 的迁移映射。
7. 文本修订只改文本表，不改存档；切换语言只重新查表，不迁移存档文字。
8. 玩家自定义名称如需保存，必须单独字段保存原文，并明确不翻译；不得与系统物品名、任务名混用。

### 3.3 正确以及错误示例

**示例 1：背包物品**

- 中文原文：铁剑
- 错误写法：

```json
{ "itemName": "铁剑", "count": 1 }
```

- 正确写法：

```json
{ "itemId": "ITEM_SWORD_IRON", "count": 1 }
```

显示时：

```csharp
itemName.text = Localization.Get("ITEM_SWORD_IRON_NAME");
```

或通过物品配置把 `ITEM_SWORD_IRON` 映射到名称文本 ID `ITEM_SWORD_IRON_NAME`，再查当前语言文本。

**示例 2：任务目标**

- 中文原文：击败黑风寨山贼
- 错误写法：

```json
{ "objective": "击败黑风寨山贼", "state": "已完成" }
```

- 正确写法：

```json
{ "questId": "QUEST_001", "objectiveIndex": 1, "state": "Completed" }
```

显示时：

```csharp
objectiveText.text = Localization.Get("QUEST_001_OBJECTIVE_01");
stateText.text = Localization.Get("QUEST_STATE_COMPLETED");
```
