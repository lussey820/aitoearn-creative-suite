# AiToEarn Trae Skills

在 [Trae IDE](https://trae.ai)（字节跳动 AI IDE）中实现端到端内容创作自动化：创意指导 Agent 理清方向 -> 下游 skill 执行生成 -> [AiToEarn](https://aitoearn.cn) 平台自动发布抖音。

## 这是什么

四个 Trae 技能文件（`.md`），以**创意指导 Agent**为统一上游入口，下游 skill 为纯执行器：

| 技能           | 文件                              | 角色     | 做什么                                                     |
| -------------- | --------------------------------- | -------- | ---------------------------------------------------------- |
| 创意指导 Agent | `aitoearn-creative-director.md`   | 统一入口 | 意图检测 -> 三档评分 -> 自适应追问 -> 输出 brief JSON -> 可选双模式润色 |
| 图文执行       | `aitoearn-content-publisher.md`   | 执行器   | 接收 brief -> 英文结构化视觉指令转译 -> 用户确认 -> AI 生成图文 -> 发布抖音 |
| 脚本创作       | `aitoearn-script-writer.md`       | 执行器   | 接收 brief -> Cinematography 词汇表生成分镜 -> 输出脚本 JSON |
| 视频生成       | `aitoearn-video-publisher.md`     | 执行器   | 加载脚本 JSON -> AI 生成视频 -> 发布抖音                     |

### 架构

```
用户输入
    │
    ▼
创意指导 Agent（aitoearn-creative-director）
    │
    │  Step 0: 意图检测（闲聊 vs 创作请求）
    │  Step 1: 6维三档评估（0/0.5/1）
    │  Step 2: 自适应追问（🔴硬性规则 + 🟢软性指南）
    │  Step 3: 命名美学方向 -> 用户确认
    │  Step 3.5: manifesto ↔ guidance 一致性自查（静默）
    │  Step 4: 输出 JSON brief -> 保存 brief-current.json
    │  Step 5: (可选) 双模式润色
    │
    ├──-> aitoearn-content-publisher.md（图文执行）
    │      读 brief-current.json -> 英文结构化视觉指令 -> 用户确认 -> 生成 -> 发布
    │
    └──-> aitoearn-script-writer.md（脚本创作）
           读 brief-current.json -> Cinematography 分镜 -> 输出脚本 JSON
              │
              └──-> aitoearn-video-publisher.md（视频生成）
                     加载脚本 JSON -> videoPrompt -> 生成 -> 发布
```

> 下游 skill 不再做意图解析和热点抓取。所有创作方向由创意指导 Agent 统一处理，下游只负责模板适配和执行。Agent 与下游通过 `brief-current.json` 文件传递数据，确保跨对话也能连接。

## 前置条件

- [AiToEarn](https://aitoearn.cn) 账号（需有可用积分）
- AiToEarn API Key（[获取教程](https://aitoearn.cn/zh/use/api-key)）
- [Trae IDE](https://trae.ai) 桌面版
- 已绑定的抖音账号

## 安装

### 1. 克隆仓库

```bash
git clone https://github.com/你的用户名/仓库名.git
```

### 2. 配置

复制 `config.example.json` 为 `config.json`，填入你的 API Key 和平台账号：

```json
{
  "apiKey": "你的_API_Key",
  "accountIds": {
    "douyin": "douyin__你的抖音ID"
  }
}
```

`config.json` 需要放在与对应 skill 文件相同的目录下（Trae IDE 的技能文件夹中）。

### 3. 导入技能

将 `.md` 文件复制到 Trae IDE 的技能目录：

```
C:\Users\你的用户名\.trae-cn\skills\
```

每个技能需要一个独立文件夹，`config.json` 与 `SKILL.md` 放在同一文件夹内。

## 使用示例

在 Trae IDE 对话中直接输入，所有请求都会先经过创意指导 Agent：

**快速通道（描述足够具体）：**

```
帮我生成赵怀真白衣侠客竹林中逆光金色粒子仰角低拍
-> Step 0: 意图检测 -> 匹配"生成"关键词
-> Step 1: 三档评分 -> 主体=1 情绪=1 画面=1 色彩=1 空间=1 -> 覆盖分=5
-> Step 3: 确认方向「残光武士」-> Step 3.5 自查 -> 输出 brief
```

**创意访谈（模糊需求）：**

```
我想要高级感
-> Step 1: 三档评分 -> 情绪=0.5("高级感"模糊)，覆盖分=0（0.5不计数）
-> Step 2: 追问"爱马仕克制奢华 vs 苹果冷感极简？"
-> 追问主体 -> 追问画面+空间 -> 覆盖分=4，收敛
-> 命名「冷萃纯净」-> 输出 brief
```

**换方向：**

```
用户: "换个方向试试"
-> Agent 保留主体/画面/约束，重新生成命名/manifesto/色彩/光线/情绪锚点
-> creativeConfidence 降为 medium -> 重新确认
```

**双模式润色：**

```
帮我把这段 AI 生成的文案去一下 AI 味
-> Step 5.0: 文本类型判定
-> 中文文案 -> AIGC 痕迹去除大师（角色注入 + persona）
-> 英文 prompt -> Prompt 精炼师（做减法、去抽象词）
```

## 项目结构

```
├── aitoearn-creative-director.md  # 创意指导 Agent（统一入口）
├── aitoearn-script-writer.md      # 脚本创作技能（执行器）
├── aitoearn-video-publisher.md    # 视频生成与发布技能（执行器）
├── aitoearn-content-publisher.md  # 图文生成与发布技能（执行器）
├── config.example.json            # 配置文件模板（不含真实密钥）
├── aitoearn-api-guide.md          # AiToEarn MCP API 参考文档
└── README.md
```

## 与 AI 助手内置技能的区别

|             | 通用 AI 内置技能              | 本项目的 Skill                                                                   |
| ----------- | ----------------------------- | -------------------------------------------------------------------------------- |
| 角色        | 无预设人设，机械执行指令      | 每个 skill 有明确角色人设（创意指导 / 导演 / 制作人 / 执行者），像懂行的朋友协作 |
| 模糊需求    | 直接填入 prompt，效果随机     | 创意指导 Agent **自适应追问**，三档评分+4/6收敛阈值，帮用户理清方向               |
| 闲聊误触发  | 关键词匹配即触发              | **Step 0 意图检测**，区分闲聊/讨论与创作请求，只在明确创作意图时进入评估           |
| 创意方向    | 散落在各个 skill 中，各自解析 | **统一上游入口**，Agent 产出结构化美学 brief（JSON），下游纯执行                 |
| 模糊描述    | "御三家""高级感"直接放行      | **三档评分**（0/0.5/1），主体和情绪必须达到 1 分才参与收敛，0.5 沿线索追问        |
| 矛盾需求    | "极简但华丽"直接写入 prompt   | **软性矛盾检测**，提炼融合方案给用户确认，不保留矛盾原始描述                      |
| 中途变向    | 一条路问到黑                  | **中途变向检测**，用户回答含 2+ 新维度时触发重新评估                              |
| 质量校验    | 无                             | **manifesto ↔ guidance 一致性自查**，防止宣言写得好但 JSON 跑偏                   |
| Prompt 质量 | 中文直译英文，丢失细节        | Agent 的 brief 含色彩/材质/光线/构图四维 guidance，下游直接填入专业模板          |
| AIGC 痕迹   | 不处理，生成内容有明显 AI 味  | Agent **双模式润色**：Prompt 精炼师（做减法）+ AIGC 痕迹去除大师（加人味）       |
| 分镜脚本    | 通用描述，无专业词汇          | 内嵌 **Cinematography 词汇表**（景别/运镜/镜头/光线/色调/转场/质量约束）         |
| 错误处理    | 返回错误码，用户自己解决      | 自动重试 + 模型降级，同 scope 内找替代模型                                       |
| 积分管理    | 不知道价格                    | 调用前**实时查价**，余额不足直接告知差额                                         |
| 批量任务    | 一次失败全部中断              | 逐条生成+逐条发布，临时故障跳过继续，结束时统一汇报                              |

## 核心特性

- **意图检测前置** - Step 0 区分闲聊/讨论与创作请求，只在检测到"帮我生成""做一个"等明确意图时进入 6 维评估
- **三档评分系统** - 主体和情绪维度分 0/0.5/1 三档，模糊描述（"御三家""高级感"）标记为 0.5 不参与收敛，必须追问到明确
- **硬性规则 vs 软性指南** - 🔴硬性规则 4 条（收敛条件/矛盾检测/变向检测/跳过标注）必须遵守，🟢软性指南 3 条尽力而为
- **软性矛盾检测** - 用户同时要 A 且非A 时，不打断不拒绝，提出融合方案（如"极简但华丽"->"让光本身成为装饰"）
- **中途变向检测** - 用户回答含 2+ 个与当前追问无关的新维度时，触发重新评估而非继续当前路径
- **一致性自查** - Step 3.5 静默检查 manifesto 与 guidance 四维（色彩/材质/光线/构图）是否一致，不一致时修正 guidance
- **双模式润色** - Prompt 精炼师（去抽象词、压缩冗余、统一术语）vs AIGC 痕迹去除大师（角色注入+persona+六大润色规范）
- **换方向保留机制** - 用户说"换个方向"时，保留主体/画面/约束，重新生成命名/manifesto/色彩/光线，creativeConfidence 降为 medium
- **brief 文件传递** - Agent 输出 brief 后保存为 `brief-current.json`，下游 skill 优先从文件加载，fallback 到上下文提取
- **结构化美学 brief** - direction（方向命名+美学宣言）+ guidance（色彩/材质/构图/光线）+ toneGuidelines（语感）+ outputConstraints
- **Cinematography 分镜** - 脚本 skill 内嵌完整电影摄影词汇表，每个分镜含 shotType/lens/cameraMovement/lighting/colorGrade/feltIntent/transition/constraints
- **实时查价** - 调用前通过 `getDraftGenerationPricing` 获取实时积分，不做硬编码
- **模型自动降级** - 视频生成失败时同 scope 内查找替代模型，不直接放弃
- **逐条生成+逐条发布** - 批量模式下每条生成完立即发布，不等全部完成
- **敏感信息隔离** - API Key 和 AccountId 统一在 `config.json` 管理，已加入 `.gitignore`

## 隐私提醒

- `config.json` 包含你的 API Key 和账号 ID，已通过 `.gitignore` 排除
- 本仓库不含任何真实密钥，所有示例均为占位符
- 使用前基于 `config.example.json` 创建自己的 `config.json`
