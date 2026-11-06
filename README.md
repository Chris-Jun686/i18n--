# 第8章：《界面排版与玩家输入》

### 8.1 关于界面排版与玩家输入

本章讲的是两件容易被低估的事：第一，界面排版不能按“当前语言看起来刚好”来设计，必须为中、英等不同语言的文本伸缩、分辨率变化和书写习惯预留空间；第二，玩家输入的内容（尤其是女儿的名字）会进入大量台词，必须同时处理输入法兼容、长度约束、敏感词过滤，以及插入句子后的语法适配。目的是让界面在任何语言下不溢出、不裁切，让玩家的输入在任何语言环境下都安全、可用、不出语法错误。

---

### 8.2 规则

#### 8.2.1 文本长度余量

- **R8-01** UI 布局必须为译文膨胀预留余量，不得以源语言（如中文）的视觉长度作为固定宽度或固定高度的唯一依据。
- **R8-02** 按钮、标签等短文本控件应使用最小宽度（min-width）或内容自适应，不得使用固定宽度；长文本区域应使用最小高度（min-height），不得使用固定高度。
- **R8-03** 文本长度限制（字符数或像素数）必须在文本表中逐条标注，并区分硬限制（超出即溢出或裁切）与软限制（建议长度，超出需设计评审）。
- **R8-04** 英文的横向占据空间通常比中文/日语/韩语（CJK）大 30%–40%，UI 布局尺寸应以**英文**长度作为基准进行排布。
- **R8-05** UI 完成后，必须使用实际长句进行最终的视觉适配测试，以排查因单词长度或折行规则导致的布局问题。

#### 8.2.2 自动换行与富文本防断行

- **R8-06** 必须实现自动换行系统，严禁要求译员在文本表中手动输入换行符（`\n`）来控制普通句子的折行。
- **R8-07** 必须使用不换行空格（NBSP）或防断行标签（如 `<nobr>`），确保特定术语、数值+单位、或不可拆分的词组（如 `my wife`）在折行时保持整体，避免产生只含一个词的“孤儿行”。
- **R8-08** 富文本标签（如 `<b>`、`<color>`）必须作为模板的一部分整体存在，不得在代码中拆分标签或跨字符串拼接。
- **R8-09** 仅当排版有严格视觉要求（如路牌指示、特殊诗句）时，才允许使用专门的硬换行符（如 `<lb>`），不得用普通 `\n` 强行折行。
- **R8-10** 当自动换行后文本仍超出容器高度时，必须启用字号自动缩放、滚动条或分页机制作为兜底，严禁从单词中间直接裁切或溢出到屏幕外。
- **R8-11** 气泡对话框和底部字幕框等非刚性UI应支持自动调整大小；字幕框锚点应固定在底部，确保内容变多时向上扩展。

#### 8.2.3 默认语言与游戏内切换

- **R8-12** 游戏首次启动时，默认语言必须跟随操作系统或平台的区域设置，不得硬编码为开发团队的母语。
- **R8-13** 游戏内必须提供语言切换入口，玩家手动选择的语言必须持久化保存，下次启动时优先于系统语言。
- **R8-14** 语言切换后，所有已加载的界面、已显示的文本以及后续生成的动态文本都必须立即按新语言刷新，不得要求重启游戏才能生效。

#### 8.2.4 玩家输入：输入法、长度与敏感词

- **R8-15** 所有玩家可输入的文本字段必须启用平台原生输入法（IME）支持，确保中文（拼音）等组合输入不被游戏逻辑错误截断或提前提交。
- **R8-16** 名字类输入必须设定与最终显示场景匹配的最大长度，长度计算应按“文本元素（Text Element）”而非“字节”进行，避免多语言下截断出半个字符。
- **R8-17** 玩家输入必须经过敏感词过滤后再保存或显示；敏感词库应覆盖中、英双语词库。
- **R8-18** 玩家输入的内容（如女儿的名字）在进入系统句子时，必须作为命名占位符的值传入，其插入后引发的语法变化按第6章的规则处理。

### 8.3 前作发生了什么

#### 8.3.1 文本膨胀

<img width="600" height="280" alt="LQA-014" src="https://github.com/user-attachments/assets/53a05bc7-36fb-4dd1-9f6b-141f2c398c35" />

> 底部文字未留足充裕空间，导致处于屏幕黑边不可见

#### 8.3.2 手动换行

<img width="600" height="280" alt="LQA-014" src="https://github.com/user-attachments/assets/53a05bc7-36fb-4dd1-9f6b-141f2c398c35" />

> 由于第一句未设置自动换行，导致第二行只有wife居中出现，影响观感

#### 8.3.3 输入法与玩家输入问题

### 8.4 正确与错误示范

#### 8.4.1 文本长度余量与自适应

**示例：江湖门派的对话框气泡**

- 中文原文：少侠请留步！
- 错误写法：气泡框采用固定大小，没有为译文留任何余量。
```csharp
// 错误：固定尺寸，完全没有自适应能力
dialogBubble.GetComponent<RectTransform>().sizeDelta = new Vector2(100f, 50f);
```
> 英文译文 "Hold on, young hero!" 在 100px 宽度下直接溢出，甚至把说话人名字盖住。

- 正确写法：气泡框使用 `ContentSizeFitter` 自动调整大小，或者允许在文本表中为该语言单独配置气泡尺寸。
```csharp
// 正确：气泡自适应内容宽度和高度
var fitter = dialogBubble.gameObject.AddComponent<ContentSizeFitter>();
fitter.horizontalFit = ContentSizeFitter.FitMode.PreferredSize;
fitter.verticalFit = ContentSizeFitter.FitMode.PreferredSize;
```

#### 8.4.2 自动换行

**示例：剧情旁白字幕**

- 中文原文：今天是我与妻子相遇十周年。
- 错误写法：在文本表中写死换行，或者任由系统在不该断的地方断行。
```text
// 错误：英文由于宽度限制或手动回车，导致 "wife" 单独掉到下一行
SUBTITLE_01 = Today marks ten years since meeting my\nwife,
```
> 如果英文被断成 "Today marks ten years since meeting my" (第一行) 和 "wife," (第二行)，就会产生极其难看的“孤儿行”。

- 正确写法：整句放入文本表不包含手动换行符，对必须连在一起的词组使用不换行空格（NBSP）。
```text
// 正确：在 my 和 wife 之间使用 \u00A0 (不换行空格)
SUBTITLE_01_EN = Today marks ten years since meeting my\u00A0wife,
```
> 通过 NBSP 将 "my wife" 绑定，系统在换行时，要么两个词都在第一行，要么整体换到第二行，不会拆分。

#### 8.4.3 溢出兜底与缩放

**示例：底部旁白框**

- 错误写法：文本超出屏幕边界，直接画进黑边里。
```csharp
// 错误：超出了容器依然不管，直接渲染到屏幕外
descriptionText.overflowMode = TextOverflowModes.Overflow;
```
- 正确写法：开启自动缩放、滚动条或分页机制，并确保锚点朝上。
```csharp
// 正确：文本超出分配空间时，自动缩小字号以适配
descriptionText.enableAutoSizing = true;
descriptionText.fontSizeMin = 12;
descriptionText.fontSizeMax = 24;
// 或者挂载 Scroll View / 实现点击翻页，确保玩家能看完，且不溢出黑边
```

#### 8.4.4 默认语言跟随系统与游戏内切换

**示例：首次启动的语言选择**

- 错误写法：启动时硬编码为开发团队的语言，忽略系统设置。
```csharp
// 错误：无论玩家系统是什么语言，都显示中文
LocalizationSettings.SelectedLocale = LocaleIdentifier.CodeToLocale("zh-CN");
```
- 正确写法：使用系统 Locale Selector 检测平台语言，并保留玩家手动选择。
```csharp
// 正确：System Locale Selector 按平台 API 依次检测
// 最终以 SystemLanguage 兜底；PlayerPref Locale Selector 保存玩家选择
```

#### 8.4.5 玩家输入：输入法、长度与敏感词

**示例：给女儿起名**

- 错误写法一：在 `Update` 中实时读取输入框文本并当作最终名字保存。
```csharp
// 错误：中文输入法的拼音组合阶段被提前捕获
void Update() {
    if (inputField.isFocused) {
        daughterName = inputField.text; // 拿到 "nver" 而不是 "女儿"
    }
}
```
- 错误写法二：按字节长度截断，不处理敏感词，直接拼进句子。
```csharp
// 错误：被按字节截断，末尾出现半个字符
byte[] bytes = Encoding.UTF8.GetBytes(input);
string name = Encoding.UTF8.GetString(bytes, 0, 10); 
msg = name + "获得了新的技能。";
```
- 正确写法：

**输入层**：在组合结束（确认提交）后读取文本。  
**长度层**：按 `StringInfo` 的文本元素计数。  
**过滤层**：调用敏感词过滤接口。  
**句子层**：名字作为命名占位符传入。

```csharp
// 输入确认后读取
string rawName = inputField.text; 

// 按文本元素截断
var textElements = StringInfo.GetTextElementEnumerator(rawName);

// 敏感词过滤
string safeName = ProfanityFilter.Filter(rawName, currentLocale);

// 插入句子
Localization.Get("MSG_DAUGHTER_GAINED_SKILL", new {
    daughter_name = safeName,
    skill_name = "玉女心经"
});
```

> 文本表（中文）：`MSG_DAUGHTER_GAINED_SKILL = {daughter_name} 获得了新技能：{skill_name}。`  
> 文本表（英文）：`MSG_DAUGHTER_GAINED_SKILL = {daughter_name} learned a new skill: {skill_name}.`  

---

> 使用 Unity 引擎时需注意

**UI 自适应与文本膨胀测试**
- 使用 `LayoutElement` 的 `minWidth` / `minHeight` 配合 `ContentSizeFitter`，避免固定 `sizeDelta` 导致译文溢出。
- 对于 PC 游戏，必须测试不同分辨率下的 UI 表现，避免全屏切换或窗口拉伸时文本错位。
- Unity 提供 Pseudolocalization（伪本地化）功能，可通过 Class Expander 类工具将字符串按比例（如英文 +30%）膨胀，在开发期直接扫描布局问题。

**自动换行、防断行与溢出兜底**
- `TextMeshProUGUI` 的 `enableWordWrapping = true` 应作为正文默认配置。
- 对于禁止拆分的词组（如避免 `wife` 单独成行），可在文本表中插入 Unicode 不换行空格 `\u00A0`（NBSP）（对应 R7-07）。
- **锚点设置**：字幕/对话框的 `RectTransform` 锚点（Anchors）必须设置在屏幕底部（Bottom），并设置 `Pivot` 为底部（Y=0）。这样当文本增多时，文本框会**向上方扩展**，而不是向下长进屏幕底部的黑边里。
- **溢出处理**：TMP 的 `Overflow` 属性绝对不能设为 `Overflow`（直接画到框外）。必须设置为 `Ellipsis`（截断并显示省略号），或开启 `enableAutoSizing`（字号自动缩小以适配框体）（对应 R7-10）。
- **长文本兜底**：对于很长的剧情旁白，建议给字幕框挂载 `ScrollRect` 组件，或实现“按任意键翻页/分页显示”机制，严禁让文字硬挤在屏幕内。

**输入法与玩家输入**
- `TMP_InputField` 需正确处理 IME 组合事件；在 PC 和移动端，输入法的候选窗行为和提交时机并不一致，需要按平台验证。
- 名字长度用 `StringInfo` 按文本元素计数，不要用 `string.Length` 或 UTF-8 字节数。

**语言切换**
- Unity Localization 的 `Startup Locale Selectors` 链已经提供 `System Locale Selector` 和 `Player Pref Locale Selector` 的顺序组合，确保“玩家选过就优先用玩家的，没选过才跟系统”。
- 切换 `SelectedLocale` 后，已绑定 `LocalizedString` 的 UI 会自动刷新；代码中动态生成的文本应在切换事件中重新调用 `Localization.Get`。

---

### 8.5 检查清单

☐ 按钮和标签是否使用了固定宽度，而不是最小宽度或自适应？</br>
☐ 长文本区域是否使用了固定高度，而不是最小高度？</br>
☐ 文本表中是否有为了普通排版而手动插入的 `\n` 换行符？</br>
☐ 是否使用了不换行空格（NBSP）或防断行标签来避免产生“孤儿行”？</br>
☐ 长文本溢出容器时，是否提供了字号缩放、滚动条或分页机制作为兜底？</br>
☐ 字幕框的锚点是否在底部，确保内容增多时向上扩展而不会溢出屏幕？</br>
☐ 游戏首次启动是否跟随系统语言，而不是硬编码？</br>
☐ 玩家手动选择的语言是否持久化保存，并在下次启动时优先使用？</br>
☐ 玩家名字输入是否处理了 IME 组合输入，而不是在拼音阶段提前提交？</br>
☐ 名字长度是否按文本元素（而非字节或 UTF-16 长度）计算和截断？</br>
☐ 敏感词过滤是否覆盖了中、英双语词库？</br>
☐ 名字插入句子时，是否作为命名占位符传入，语法变化是否交给第6章的处理机制？

---

### 8.6 参考文献

- IGDA 本地化兴趣小组《游戏本地化最佳实践》 *Best Practices for Game Localization*（UI 灵活性、文本膨胀、Unicode 与字符串外置）
- Unity 官方文档 — Startup Locale Selectors（System Locale Selector 与 Player Pref Locale Selector）
- Unity 官方文档 — TextMeshPro Line Breaking & Rich Text（自动换行、防断行标签与富文本）
- Unicode 标准附件 #29 (UAX #29) — Text Segmentation（文本元素边界与多语言字符截断）
