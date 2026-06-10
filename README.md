# 🚀 Prompt Optimize Skill

一个面向 AI Agent 的 **Prompt 自动优化 Skill**——给它一份评测集和业务目标，它就能自动迭代改进你的 Prompt，并用指标告诉你每一轮改了什么、好了多少。

## ✨ 它能做什么

- **指标驱动优化**：基于 accuracy / F1 / recall 等确定性指标量化每一轮改进，不靠主观感觉
- **全自动迭代**：Agent 自动执行「归因 → 改写 → 策略分组 → 候选生成 → 评分 → 护栏校验」闭环
- **护栏机制**：内置回归检测——新 Prompt 不会破坏已经做对的样本
- **多模型支持**：支持 DeepSeek / OpenAI / Anthropic / 通义千问等主流模型

### 支持的任务类型

| 任务类型 | 说明 | 示例 |
|----------|------|------|
| 分类 (classification) | 将输入归为一个类别 | 意图识别、情感分析 |
| 打标签 (tagging) | 给输入打多个标签 | 多标签分类 |
| 结构化抽取 (structured_extraction) | 从文本中提取结构化字段 | 信息抽取、NER |
| 评分 (rubric_scoring) | 按评分标准给出数值分数 | 文本质量评分 |

---

## 📦 安装

只需下载 `prompt_optimize` 文件夹，放到 Agent 的 skills 目录即可。

### Gemini CLI

**全局安装**（所有项目可用）：

```bash
# 下载 prompt_optimize 文件夹到 skills 目录
mkdir -p ~/.gemini/config/skills && \
curl -sL https://github.com/PengLiuXiao/prompt_optimatize/archive/refs/heads/main.tar.gz | \
tar xz --strip-components=1 --include='*/prompt_optimize/*' -C ~/.gemini/config/skills
```

**项目级安装**（仅当前项目可用）：

```bash
mkdir -p .agents/skills && \
curl -sL https://github.com/PengLiuXiao/prompt_optimatize/archive/refs/heads/main.tar.gz | \
tar xz --strip-components=1 --include='*/prompt_optimize/*' -C .agents/skills
```

### Claude Code

```bash
mkdir -p .agents/skills && \
curl -sL https://github.com/PengLiuXiao/prompt_optimatize/archive/refs/heads/main.tar.gz | \
tar xz --strip-components=1 --include='*/prompt_optimize/*' -C .agents/skills
```

安装后的目录结构：

```
.agents/skills/          # 或 ~/.gemini/config/skills/
└── prompt_optimize/     ← 这就是完整的 Skill
    ├── SKILL.md
    ├── assets/
    ├── scripts/
    └── references/
```

### 安装 Python 依赖

```bash
pip install -r <skill-path>/prompt_optimize/assets/requirements.txt
```

> **Python 版本要求**: 3.10+

---

## 🔧 配置 API Key

将模板复制到项目根目录并填入你的 Key：

```bash
cp <skill-path>/prompt_optimize/assets/.env.example .env
```

```env
# 根据你的模型服务商取消注释对应行：
DEEPSEEK_API_KEY=sk-your-key-here
# OPENAI_API_KEY=sk-your-key-here
# ANTHROPIC_API_KEY=sk-ant-your-key-here
# DASHSCOPE_API_KEY=sk-your-key-here
```

---

## 🚀 使用

### 你需要准备

1. **待优化的 Prompt**（文本或文件）
2. **人工标注的评测集**（JSONL / CSV / Excel），格式示例：
   ```jsonl
   {"id": "1", "input": "客户要求退款", "gold": "after_sales"}
   {"id": "2", "input": "查询物流状态", "gold": "logistics"}
   ```
3. **业务目标**（自然语言，如"优先提升召回率"）

### 开始优化

安装完成后，直接在 Agent 对话中说：

```
帮我优化这个 Prompt。评测数据在 ./data/eval.jsonl，
目标模型是 deepseek-chat，业务目标是优先提升召回率。
```

Agent 会自动调用此 Skill 完成全部优化流程，最终输出优化后的 Prompt 和改进报告。

---

## 📄 License

MIT License
