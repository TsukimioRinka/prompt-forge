# prompt-forge

**检索对标 → 骨架重构 → 成品交付** —— 一个不信模型记忆、只信实测检索的 prompt 锻造技能。

## 是什么

prompt-forge 是 [omp / magpie 系 agent](https://github.com/TsukimioRinka) 的**明文调用技能**：`hide: true` + `disable-model-invocation: true`，模型平时看不到它、永远不会自动触发，只有你打出 `/skill:prompt-forge` 才唤醒。

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

## 安装

```bash
git clone https://github.com/TsukimioRinka/prompt-forge.git ~/.omp/agent/skills/prompt-forge
```

克隆即装（本仓库根目录就是技能根目录），之后在 omp 里打出：

```
/skill:prompt-forge 我要一个让 Claude 审 PR 的 system prompt
```

不带粗稿、只有一句话意图也可以。

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

## License

MIT —— 本技能文本以 MIT 开源。它检索的上游档案各自保持原许可（CC0 / MIT / AGPL-3.0）；技能只学习其结构写法，不复制其正文。
