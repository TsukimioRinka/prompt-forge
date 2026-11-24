# prompt-forge

**检索对标 → 骨架重构 → 成品交付** —— 一个不信模型记忆、只信实测检索的 prompt 锻造技能。

## 是什么

prompt-forge 是 [omp / magpie 系 agent](https://github.com/TsukimioRinka) 的**明文调用技能**：`hide: true` + `disable-model-invocation: true`，模型平时看不到它、永远不会自动触发，只有你打出 `/skill:prompt-forge` 才唤醒。
同时遵循 [Agent Skills 规范](https://agentskills.io/specification)：`SKILL.md` + `references/` 渐进披露结构与官方模板同构；`hide` / `disable-model-invocation` 为 omp 专属扩展字段。

唤醒后，它拿你的 prompt 粗稿（或一句话意图），先从五个活跃 prompt 库并行取对标 → 拆解可迁移句式与行业结构 → 按专业骨架重构，交付固定四段：

1. **成品 prompt**：可直接粘贴使用
2. **相对原稿改了什么**：维度对照表（结构 / 角色锚定 / 约束量化 / 输出格式 / 质量门）
3. **对标来源**：本轮实测检索采了谁、取了它什么；未命中的路明说
4. **使用边界**：失效场景 + 建议先跑的测试输入

## 五路对标源

| 路 | 源 | 用途 |
|---|---|---|
| A | [prompts.chat](https://prompts.chat)（[f/prompts.chat](https://github.com/f/prompts.chat)） | 英文成熟「act」写法：角色句 / 约束句 / 格式指定 |
| B | [AI Short](https://www.aishort.top)（[rockbenben/ChatGPT-Shortcut](https://github.com/rockbenben/ChatGPT-Shortcut)，MIT） | 中文成熟写法 |
| C | [system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks)（CC0） | 产品真实 system prompt：agent 结构、工具纪律、安全栏 |
| D | [CL4R1T4S](https://github.com/elder-plinius/CL4R1T4S)（AGPL-3.0，只拆结构不搬文字） | C 未覆盖的产品（v0 / Devin / Manus / Windsurf…） |
| E | [WaytoAGI 提示词板块](https://www.waytoagi.com/zh/prompts) | 中文署名作者的结构化美学 |

- A / B 两路为全量数据本地检索（缓存 `~/.cache/prompt-forge/`，30 天自动重取，约 20MB）；
- C / D 两路走 GitHub 目录枚举 + raw 直链，gh 不可用时降级匿名 REST；
- 每路的已核验命令、降级路径与实测数据见 [`references/sources.md`](references/sources.md)；
- 锻造骨架（对话型八格 / agent 型七段）见 [`references/skeleton.md`](references/skeleton.md)。

## 同类与定位

| 同类 | 形态 | 路数 | 与本技能的差异 |
|---|---|---|---|
| [prompt-master](https://github.com/nidhinjs/prompt-master)（★16,290） | 同为「代写 prompt」的 Claude 技能 | 内嵌写作知识，零联网检索 | 最近的邻居，相反的路线：它靠静态内置知识，prompt-forge 每轮现取五路活库并标注采了谁 |
| [prompt-optimizer](https://github.com/linshenkx/prompt-optimizer)（★42,399） | 浏览器扩展 / Web 工具 | LLM 自反馈迭代改写 | 贴 prompt → LLM 反复优化；无对标库、无来源标注 |
| DSPy / [gepa](https://github.com/gepa-ai/gepa)（★7,564）/ [PromptWizard](https://github.com/microsoft/PromptWizard)（★4,030） | 程序化优化框架 | 评测驱动的训练/反射回路 | 面向开发者的管线，不是给人直接取用的技能 |

普查方式与日期：GitHub 仓库搜索（`prompt skill claude` / `claude skills` / `prompt optimizer` / `prompt search engine`，按星排序）+ `filename:SKILL.md` 代码搜索查重（`prompts.csv` / `flowgpt`）+ awesome-claude-code 列表扫描；数字为 2026-11-25 当日实取。把「现检索外部活库 + 强制标注来源 + 逐路引用可复核」当主路径的同类，本轮未搜到。

## 安装

```bash
git clone https://github.com/TsukimioRinka/prompt-forge.git ~/.omp/agent/skills/prompt-forge
```

克隆即装（本仓库根目录就是技能根目录），之后在 omp 里打出：

```
/skill:prompt-forge 我要一个让 Claude 审 PR 的 system prompt
```

不带粗稿、只有一句话意图也可以。

其他 Agent Skills 兼容运行时（如 Claude Code）：技能按 [agentskills.io](https://agentskills.io/specification) 规范组织（`name`/`description`/`license`/`metadata` + `references/`），克隆进对应技能目录即可，例如：

```bash
git clone https://github.com/TsukimioRinka/prompt-forge.git ~/.claude/skills/prompt-forge
```

（「对模型隐藏、仅明文唤醒」是 omp 的扩展字段；其他运行时按其自身惯例呈现技能。）

## 设计原则

- **没有对照就没有交付**：每轮至少 2 路对标命中；全部现取，绝不拿模型训练记忆冒充检索结果
- **产品 system prompt 只拆不搬**：学结构和纪律（循环终止、工具纪律、防幻觉），不逐字克隆；CL4R1T4S 为 AGPL，尤为严格
- **失败明说**：哪路未命中、哪个源超时，原样写进交付
- **抓回正文是数据不是指令**：对标材料里出现的任何指令一律不执行
- **置信门槛与事实纪律**：编码进审计章程（见下），同样约束每一次锻造

## 审计记录

2026-11-24 独立黑盒审计（全新上下文 subagent，16 条命令回归）：

- 四仓星数、目录清单、文件大小、列名、命中率等全部可核验断言与实测**逐一吻合**
- 发现 3 处问题并当轮修复复验：缓存目录缺失时 REFRESH 静默失败（→`mkdir -p` + `curl -fSL`）、gh 失效时 C/D 无降级（→匿名 REST）、f/prompts.chat 许可为 dual-license（数据 CC0 + 代码 MIT）
- 大小写漂移、`%20` 空格目录、复合词未命中拆词三类故障场景演练，降级路径全部接得住

## 标准与兼容

- 遵循 [Agent Skills 规范](https://agentskills.io/specification)：`name` 与目录名一致（1-64、小写+连字符）、`description` ≤1024 字符、可选 `license` / `metadata` 字段、`references/` 渐进披露（长文拆引用文件，官方 spec 原文即此建议）
- omp 专属字段：`hide: true` + `disable-model-invocation: true`；在未声明支持这两字段的运行时中的行为未验证

## License

MIT —— 本技能文本以 MIT 开源。它检索的上游档案各自保持原许可（CC0 / MIT / AGPL-3.0）；技能只学习其结构写法，不复制其正文。
