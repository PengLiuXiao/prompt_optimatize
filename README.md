# 🚀 Prompt Optimize Skill

**一个面向 AI Agent 的本地 Prompt 自动优化 Skill**——基于人工标注的评测集，通过迭代归因、改写、策略分组与护栏校验，系统化地提升结构化任务 Prompt 的效果。

> 本项目是一个标准的 [Gemini CLI Skill](https://github.com/google-gemini/gemini-cli)，可即装即用。同时兼容 Claude Code 等支持 Skill 协议的 AI Agent。

---

## ✨ 核心特性

| 特性 | 说明 |
|------|------|
| 🎯 **指标驱动** | 以 accuracy / F1 / recall 等确定性指标量化每一轮改进，杜绝主观判断 |
| 🔄 **全自动迭代** | Agent 自动执行「归因 → 改写建议 → 策略分组 → 候选生成 → 评分 → 护栏校验」闭环 |
| 🛡️ **护栏机制** | 内置回归检测——新 Prompt 不得破坏已正确的样本 |
| 🔌 **多模型支持** | 通过 [LiteLLM](https://github.com/BerriAI/litellm) 统一接口，支持 DeepSeek / OpenAI / Anthropic / 通义千问等 |
| 📦 **零依赖安装** | 作为 Skill 文件夹直接放入 Agent 配置目录即可，无需全局安装 |

---

## 📂 目录结构

```
prompt_optimize_skill/
├── SKILL.md                          # Skill 定义文件（Agent 读取的完整编排规范）
├── README.md                         # 本文件
├── assets/
│   ├── requirements.txt              # Python 运行时依赖
│   ├── .env.example                  # API Key 配置模板
│   └── prompts/                      # Agent 各阶段使用的 LLM 提示模板
│       ├── eval_plan_generation.md
│       ├── badcase_attribution.md
│       ├── rewrite_suggestion.md
│       ├── strategy_grouping.md
│       ├── candidate_generation.md
│       └── best_round_report.md
├── scripts/
│   ├── run_prompt_optimizer.py       # CLI 入口（可从任意目录调用）
│   └── prompt_optimizer/
│       ├── cli.py                    # 子命令分发
│       └── tools/                    # 确定性工具集
│           ├── dataset.py            # 数据集加载 & 校验
│           ├── scorer.py             # 预测评判（is_correct）
│           ├── metrics.py            # 指标计算
│           ├── guardrail.py          # 护栏检查
│           └── inference.py          # 目标模型推理调用
└── references/
    └── usage.md                      # Agent 使用参考手册
```

---

## 🔧 安装

### 方式一：Gemini CLI（推荐）

将本仓库克隆到 Gemini CLI 的 Skill 目录：

```bash
# 全局安装（所有项目可用）
git clone https://github.com/<your-username>/prompt_optimize_skill.git \
  ~/.gemini/config/skills/prompt_optimize_skill

# 或项目级安装（仅当前项目可用）
git clone https://github.com/<your-username>/prompt_optimize_skill.git \
  ./.agents/skills/prompt_optimize_skill
```

安装 Python 依赖：

```bash
pip install -r ~/.gemini/config/skills/prompt_optimize_skill/assets/requirements.txt
```

### 方式二：Claude Code

```bash
git clone https://github.com/<your-username>/prompt_optimize_skill.git \
  ./.agents/skills/prompt_optimize_skill
pip install -r ./.agents/skills/prompt_optimize_skill/assets/requirements.txt
```

### 方式三：手动使用

如果你不使用 AI Agent，也可以直接调用 CLI 工具：

```bash
git clone https://github.com/<your-username>/prompt_optimize_skill.git
pip install -r prompt_optimize_skill/assets/requirements.txt
python prompt_optimize_skill/scripts/run_prompt_optimizer.py --help
```

---

## ⚙️ 配置

### API Key 设置

复制模板并填入你的 API Key：

```bash
cp assets/.env.example .env
```

根据你使用的模型服务商，在 `.env` 中取消注释对应行：

```env
# DeepSeek
DEEPSEEK_API_KEY=sk-your-key-here

# OpenAI
# OPENAI_API_KEY=sk-your-key-here

# Anthropic
# ANTHROPIC_API_KEY=sk-ant-your-key-here

# 通义千问 (DashScope)
# DASHSCOPE_API_KEY=sk-your-key-here
```

---

## 🚀 快速开始

### 你需要准备

1. **待优化的 Prompt**——纯文本或 Markdown 文件
2. **人工标注的评测集**——JSONL / CSV / Excel 格式，每行包含输入和对应的正确答案
3. **业务目标**——用自然语言描述（例如"提升分类的召回率"）
4. **目标模型**——你实际部署使用的模型（如 `deepseek-chat`、`gpt-4o`）

### 评测集格式示例

**JSONL 格式**：

```jsonl
{"id": "1", "input": "客户要求退款", "gold": "after_sales"}
{"id": "2", "input": "查询物流状态", "gold": "logistics"}
{"id": "3", "input": "产品使用教程", "gold": "product_guide"}
```

**CSV 格式**：

```csv
id,input,gold
1,客户要求退款,after_sales
2,查询物流状态,logistics
3,产品使用教程,product_guide
```

### 与 Agent 交互

安装完成后，直接在 Agent 对话中描述你的需求即可：

```
帮我优化这个分类 Prompt。评测数据在 ./data/eval.jsonl，
目标模型是 deepseek-chat，业务目标是优先提升召回率。
```

Agent 会自动识别并调用此 Skill，执行完整的优化流程。

---

## 🛠️ CLI 参考

所有子命令均输出 JSON，可从任意目录通过绝对路径调用：

```bash
SKILL=/path/to/prompt_optimize_skill
python $SKILL/scripts/run_prompt_optimizer.py <subcommand> [args]
```

| 子命令 | 作用 | 关键参数 |
|--------|------|----------|
| `validate-inputs` | 校验数据集格式、确认每行都有 gold answer | `--dataset` `[--input-column]` `[--gold-column]` |
| `load-dataset` | 加载并规范化数据集；`--strip-gold` 时仅输出 `{id, input}` | `--dataset` `[--strip-gold]` |
| `run-inference` | 调用目标模型 API 生成预测 | `--prompt` `--dataset` `--model` `[--api-key]` `[--concurrency]` |
| `score-predictions` | 按任务类型规则判定每条预测的 `is_correct` | `--task-type` `--predictions` `[--dataset]` |
| `compute-metrics` | 计算任务类型对应的全部指标 | `--task-type` `--results` |
| `check-guardrails` | 基于护栏规则判定候选 Prompt 是否合格 | `--payload` |

---

## 📊 支持的任务类型与指标

| 任务类型 | 评判规则 | 可用指标 |
|----------|----------|----------|
| `classification` | 精确字符串匹配 | accuracy, macro_precision, macro_recall, macro_f1 |
| `quality_judgement` | 精确字符串匹配 | accuracy, macro_precision, macro_recall, macro_f1 |
| `tagging` | 集合相等 | micro_precision, micro_recall, micro_f1, exact_match, label_regression_rate |
| `structured_extraction` | 字段集 + 字段值匹配 | field_accuracy, field_f1, missing_field_rate, extra_field_rate |
| `rubric_scoring` | 数值相等 | exact_match, within_1_accuracy, mae |

---

## 🔄 优化流程概览

```
┌─────────────────────────────────────────────────────────┐
│                    Step 0: Validate                      │
│   校验数据集 → 确认模型 & API Key                         │
└─────────────────┬───────────────────────────────────────┘
                  ▼
┌─────────────────────────────────────────────────────────┐
│                 Step 1: Eval Plan                        │
│   确定 task_type、primary_metric、guardrails             │
└─────────────────┬───────────────────────────────────────┘
                  ▼
┌─────────────────────────────────────────────────────────┐
│                Step 2: Baseline                          │
│   用当前 Prompt 跑推理 → 评分 → 计算基线指标              │
└─────────────────┬───────────────────────────────────────┘
                  ▼
┌─────────────────────────────────────────────────────────┐
│            Iteration (最多 N 轮)                         │
│                                                         │
│   1. 归因分析：定位错误根因                               │
│   2. 改写建议：针对每个根因生成最小编辑                    │
│   3. 策略分组：将建议聚合为 ≤4 个策略方向                 │
│   4. 候选生成：为每个策略生成候选 Prompt                  │
│   5. 评分：每个候选跑推理 + 评分                         │
│   6. 护栏校验：检查回归、次要指标下降等                    │
│   7. 选择：挑选最优候选作为下一轮 Prompt                  │
│                                                         │
└─────────────────┬───────────────────────────────────────┘
                  ▼
┌─────────────────────────────────────────────────────────┐
│                    Final Report                          │
│   输出 final_prompt.md + best_round_report.md            │
└─────────────────────────────────────────────────────────┘
```

---

## 📋 运行时依赖

| 依赖 | 版本要求 | 用途 |
|------|----------|------|
| [litellm](https://github.com/BerriAI/litellm) | ≥ 1.40 | 统一多模型 API 调用 |
| [openpyxl](https://openpyxl.readthedocs.io/) | ≥ 3.1 | 读取 Excel 格式评测集 |
| [python-dotenv](https://github.com/theskumar/python-dotenv) | ≥ 1.0 | 从 .env 文件加载环境变量 |

**Python 版本要求**: 3.10+

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

## 📄 License

MIT License
