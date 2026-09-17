# Trace as State (TaS) - Antigravity AI Agent Skill

[![Paper](https://img.shields.io/badge/arXiv-2609.02702-b31b1b.svg)](https://arxiv.org/abs/2609.02702)
[![Platform](https://img.shields.io/badge/Platform-Antigravity%202.0-blue.svg)](https://github.com)
[![License](https://img.shields.io/badge/License-CC%20BY%204.0-green.svg)](https://creativecommons.org/licenses/by/4.0/)

本项目将清华大学唐杰团队与智谱 AI 的最新前沿学术成果——《**Trace as State: Reasoning Traces as Conditional States for Long-Context Transformers**》（arXiv:2609.02702v1）——工程化实现为适用于 Google Antigravity / AI Agent 系统的实战型 **Skill**。

> 🔗 **原论文官方链接**：
> - 📄 **arXiv 摘要页**：[https://arxiv.org/abs/2609.02702](https://arxiv.org/abs/2609.02702)
> - 📥 **PDF 官方下载**：[https://arxiv.org/pdf/2609.02702](https://arxiv.org/pdf/2609.02702)
> - 🌐 **HTML 网页版在线阅读**：[https://arxiv.org/html/2609.02702v1](https://arxiv.org/html/2609.02702v1)


---

## 📌 背景与核心理论

在超长文本（Long Context）与复杂推理任务中，自回归 Transformer 模型存在天然的**单向因果时序缺陷**：后文生成的推导信息无法逆向影响前面已经建立的文本表征。

论文提出 **Trace as State (TaS)** 范式，通过数学证明表明：
- **条件在前 $[z, C]$**：最坏情况下持久工作内存仅需 $\lceil b \rceil$ bits；
- **条件在后 $[C, z]$**：最坏情况下内存需求暴增至 $\lceil b 2^b \rceil$ bits（呈**指数级爆炸**）。

本 Skill 将大模型第一轮探索生成的**思考草稿（Reasoning Trace）作为任务状态的显式文本代理**，在第二轮置顶于长文本之前，逆转因果表征劣势。在 GraphWalks Parents 等硬核长文本评测中，使 DeepSeek V4 Pro 的精确匹配率从 **29.2% 飙升至 81.8%**，GLM-5.2 达到 **100% 满分**！

---

## 🚀 核心工作流（Two-Pass SOP）

```text
[用户输入长文本/复杂代码任务] ──> [/trace-as-state 激活技能]
           │
           ▼
【阶段 1：派发先遣子代理独立摸排（Pass 1）】
- 后台启动只读探索子代理，在独立沙箱中通读原文；
- 详尽输出完整探索推导链、代码行号、排除假说与初步依据；
- 物理隔离初探噪音，主对话窗口保持绝对干净。
           │
           ▼
【阶段 2：状态序列化与防思维惯性包装】
- 执行 50,000 字符单条轨迹防爆截断；
- 显式剔除裸露答案，逼迫二轮独立复核；
- 注入强制防盲从警示语（将模型心态转为“挑刺审评官”）。
           │
           ▼
【阶段 3：主 Agent 状态前置深度复审（Pass 2）】
- 严格遵循排布铁律：[前置状态 T] + [长文主体 x] + [末尾具体问题与格式 q]；
- 主 Agent 亲自带着上帝视角进行定向重读核验，纠正初探错误；
- 直接在主会话交付确定性极高的答案，并无缝承接后续追问。
```

---

## 📂 仓库目录结构

```text
├── trace-as-state/
│   └── SKILL.md               # 核心 Skill 规范文件（SOP 作业流程与 Prompt 模板）
├── 2609.02702v1.pdf           # arXiv 原始学术论文 PDF
└── README.md                  # 项目介绍与使用指南
```

---

## 🛠️ 安装与使用方法

### 1. 安装到当前项目（Workspace 模式）
将 `trace-as-state/` 文件夹复制到你项目的 `.agents/skills/` 目录下：
```bash
# 在你的项目根目录下执行
mkdir -p .agents/skills
cp -r /path/to/trace-as-state .agents/skills/
```

### 2. 安装为全局技能（跨项目通用）
将 `trace-as-state/` 文件夹复制到 Antigravity 全局技能库目录：
```bash
# Windows
cp -r trace-as-state "C:\Users\<你的用户名>\.gemini\config\skills\"

# macOS / Linux
cp -r trace-as-state ~/.gemini/config/skills/
```

### 3. 在对话中调用
在与 Antigravity AI 对话时，只需输入：
```text
/trace-as-state 请帮我深入分析这个超长业务模块的调用链路，排查潜在的状态竞态问题
```
或直接要求 AI：“*请使用 Trace as State 技能深入重读这篇论文/文档*”。

---

## ⚠️ 避坑准则与工程防护

1. **必须物理隔离**：第一轮摸排必须由独立子代理在后台完成，严禁在主窗口自问自答，防止残留注意力矩阵造成思维惯性。
2. **防无关草稿（Random Trace 毒药）**：论文消融实验表明，塞入通用或无关的假草稿会导致准确率断崖式下跌至 14%（毒性极大）。草稿必须针对当前具体任务真实生成。
3. **单条轨迹 50,000 字符硬截断**：大模型思考可能无限延展，超过 5 万字符实施硬截断，防止超出上下文窗口上限并控制 API 成本。
4. **末尾问题锚定**：输入重组时，具体问题与输出格式指令必须始终保留在**全文本的最末端**，防止模型在读完全文后失焦。

---

## 📄 引用与鸣谢

本项目学术理论源自：
```bibtex
@article{zou2026trace,
  title={Trace as State: Reasoning Traces as Conditional States for Long-Context Transformers},
  author={Zou, Xu and Tang, Jie},
  journal={arXiv preprint arXiv:2609.02702},
  year={2026}
}
```
感谢清华大学唐杰老师团队与智谱 AI 团队在长上下文推理领域的基础性突破！
