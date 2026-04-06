# VLM 手写体识别项目规划

> 针对新中国成立后手写体历史资料的结构化识别项目
> 版本：v2.0
> 更新日期：2026-04-06
> 适用平台：macOS / Windows

---

## 一、项目背景与目标

### 1.1 背景
研究团队拥有一批新中国成立后的手写体历史资料（如信用社存款凭证、档案等），需要将其数字化为结构化数据。计划自建 VLM（Vision-Language Model）识别系统。

### 1.2 目标
```
输入：历史手写体文档图片（如 1980 年代信用社凭证）
输出：结构化 JSON 数据，包含：
{
  "document_type": "信用社存款凭证",
  "date": "1981-02-01",
  "location": "地点",
  "amount": 1500.00,
  "transaction_item": "交易项目"
}
```

### 1.3 数据规模与优先级
| 项目 | 说明 |
|------|------|
| 总规模 | ~10,000 页 |
| 优先字段 | **地点**、**金额**、**交易项目** |
| 隐私要求 | 尽量避免云端，优先本地部署 |

---

## 二、团队角色与协作模式

### 2.1 团队成员配置

| 角色 | 职责 | 技能要求 |
|------|------|----------|
| **项目负责人** | 定义数据需求、质量标准 | 经济学/历史学背景 |
| **数据标注员** | 提供标注样本、验证结果 | 细心，了解史料 |
| **技术执行** | 环境部署、脚本运行 | Python 基础 |
| **AI 助手** | 代码编写、问题诊断 | Claude Code |

### 2.2 协作工具
- **版本控制**: GitHub
- **沟通**: 微信/Slack + GitHub Issues
- **文档**: 本规划文档 + README
- **数据共享**: 团队网盘/Git LFS（小文件）

---

## 三、硬件配置要求

### 3.1 macOS 系统

#### 检查配置命令
```bash
# 查看芯片型号
uname -m  # arm64 = M 系列，x86_64 = Intel

# 查看内存
memory_pressure | grep "MemSize"

# 查看可用存储
df -h /
```

#### 配置对照表
| 配置 | 推荐方案 | 预期性能 |
|------|----------|----------|
| M1/M2, 8GB | 云端 API 或 2B 量化模型 | 中等 |
| M1/M2, 16GB+ | 本地 7B 量化模型 | 良好 |
| M3/M4, 16GB+ | 本地部署最佳体验 | 优秀 |
| Intel Mac | 建议云端方案 | 较慢 |

### 3.2 Windows 系统

#### 检查配置方法

**方法 1：系统信息**
```powershell
# 按 Win+R，输入 cmd，运行：
systeminfo | findstr /C:"OS" /C:"Total Physical"
```

**方法 2：设置查看**
- 设置 → 系统 → 关于 → 查看设备规格

**方法 3：命令检查**
```powershell
wmic os get Caption
wmic computersystem get totalphysicalmemory
```

#### 配置对照表
| 配置 | 推荐方案 | 预期性能 |
|------|----------|----------|
| CPU 8 核以下，8GB | 云端 API | 中等 |
| CPU 8 核+，16GB | 本地量化模型 | 良好 |
| RTX 3060+，16GB+ | 本地 GPU 加速 | 优秀 |
| RTX 4090，32GB+ | 本地 7B 全精度 | 最佳 |

#### Windows 必备环境
```powershell
# 1. 安装 Python 3.10+ （如未安装）
# 下载地址：https://www.python.org/downloads/
# 安装时勾选 "Add Python to PATH"

# 2. 安装 Git（如未安装）
# 下载地址：https://git-scm.com/download/win

# 3. 验证安装
python --version
git --version
```

---

## 四、技术路线选择

### 4.1 三种技术方案对比

| 方案 | 难度 | 成本 | 效果 | 推荐场景 |
|------|------|------|------|----------|
| **方案 1：云端 API** | ★☆☆ | ¥0.002-0.005/张 | ★★★★ | 快速验证，<500 张 |
| **方案 2：本地轻量模型** | ★★☆ | 免费 | ★★★☆ | 批量处理，隐私需求 |
| **方案 3：微调专用模型** | ★★★ | 时间成本 | ★★★★★ | 大量同类文档 |

### 4.2 本项目推荐方案

考虑到 **10,000 页规模** 和 **隐私优先** 原则：

```
混合方案：
├── 阶段 1：云端 API 测试 100 张 → 验证可行性
├── 阶段 2：本地部署处理 9,900 张 → 批量处理
└── 阶段 3：疑难样本云端复判 → 质量保障
```

### 4.3 10,000 页成本估算

| 方案 | 时间 | 成本 |
|------|------|------|
| 纯云端 | 1-2 小时 | ¥300-500 |
| 本地部署 | 8-10 小时 | ¥5-10（电费） |
| **混合方案** | **10-12 小时** | **¥20-30** |

---

## 五、软件环境部署

### 5.1 macOS 部署方案

#### 方案 A：Ollama（最简单，推荐入门）
```bash
# 1. 安装 Ollama
brew install ollama

# 2. 启动服务
ollama serve

# 3. 下载模型（选择其一）
ollama pull qwen2-vl:2b      # 2B 参数，6-8GB 内存
ollama pull minicpm-v:8b     # 8B 参数，12-16GB 内存
```

#### 方案 B：MLX（Apple Silicon 优化最佳）
```bash
# 1. 安装
pip3 install mlx-lm mlx-vlm

# 2. 运行示例
python3 -m mlx_vlm --model Qwen/Qwen2-VL-2B-Instruct
```

#### 方案 C：llama.cpp（量化支持好）
```bash
# 1. 克隆
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# 2. 编译
make -j

# 3. 运行
./llama-cli -m path/to/model.gguf -p "识别图片文字"
```

### 5.2 Windows 部署方案

#### 方案 A：Ollama for Windows（推荐）
```powershell
# 1. 下载 Ollama Windows 版
# https://ollama.ai/download

# 2. 安装后在 PowerShell 运行
ollama pull qwen2-vl:2b

# 3. 启动服务
ollama serve
```

#### 方案 B：Python + 直接推理
```powershell
# 1. 创建虚拟环境
python -m venv vlm_env
.\vlm_env\Scripts\Activate

# 2. 安装依赖
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
pip install transformers qwen-vl-utils

# 3. 运行测试
python scripts/01_test_model.py
```

#### 方案 C：WSL2（Linux 子系统，适合高级用户）
```powershell
# 1. 启用 WSL2
wsl --install

# 2. 安装 Ubuntu
wsl --install -d Ubuntu-22.04

# 3. 在 WSL 中按照 Linux 方式部署
```

### 5.3 云端 API（备选方案）

| 服务商 | 文档 | 特点 |
|--------|------|------|
| 智谱 AI | https://open.bigmodel.cn/dev/api | 手写体效果好 |
| 月之暗面 | https://platform.moonshot.cn/docs | 支持长文本 |
| 阿里百炼 | https://help.aliyun.com/zh/model-studio | 性价比高 |

---

## 六、推荐模型清单

### 6.1 轻量级模型（本地部署）

| 模型 | 参数量 | 内存需求 | 手写体能力 | 推荐平台 |
|------|--------|----------|------------|----------|
| **Qwen2-VL-2B** | 2B | 6-8GB | ★★★☆ | macOS/Win |
| **MiniCPM-V 2.6** | 8B | 12-16GB | ★★★★ | macOS/Win |
| **LLaVA-Phi-3** | 3B | 8-10GB | ★★★☆ | macOS/Win |

### 6.2 模型选择建议

```
内存 8GB：Qwen2-VL-2B（4-bit 量化）
内存 16GB：MiniCPM-V 2.6 或 LLaVA-Phi-3
内存 24GB+：Qwen2-VL-7B（4-bit）
有独立显卡（Windows）：优先使用 GPU 加速
```

---

## 七、项目目录结构

```
Historical Study of Population Migration Patterns/
├── vlm_ocr/
│   ├── configs/               # 配置文件
│   │   └── model_config.yaml
│   ├── samples/               # 样本图片（.gitignore）
│   │   ├── train/             # 训练样本
│   │   └── test/              # 测试样本
│   ├── scripts/               # Python 脚本
│   │   ├── 01_api_test.py     # API 测试
│   │   ├── 02_recognize.py    # 单张识别
│   │   ├── 03_batch_process.py # 批量处理
│   │   └── 04_export_stata.py # 导出 Stata
│   ├── output/                # 识别结果
│   │   ├── json/              # 原始 JSON
│   │   ├── excel/             # Excel 汇总
│   │   └── stata/             # Stata 格式
│   └── logs/                  # 运行日志
├── docs/
│   ├── VLM 手写体识别项目规划.md
│   └── 识别质量检验报告.md
├── .gitignore
└── README.md
```

### .gitignore 建议
```gitignore
# 忽略大文件
vlm_ocr/samples/*.png
vlm_ocr/samples/*.jpg
vlm_ocr/output/
models/

# 忽略敏感信息
.env
*.key
credentials.json

# 保留配置
!vlm_ocr/configs/
!vlm_ocr/scripts/
```

---

## 八、详细实施步骤

### 阶段 1：环境准备与可行性验证（Week 1）

**目标**：确认 VLM 能否有效识别你的文档类型

| 任务 | 负责人 | 产出 |
|------|--------|------|
| 1. 安装环境 | 技术执行 | 可运行环境 |
| 2. 准备 10 张样本 | 数据标注员 | 测试图片集 |
| 3. 测试 2-3 个 API | 技术执行 | 对比报告 |
| 4. 确定字段模板 | 项目负责人 | JSON 模板 |

### 阶段 2：本地部署与对比测试（Week 2-3）

**目标**：确定最终技术方案

| 任务 | 负责人 | 产出 |
|------|--------|------|
| 1. 部署本地模型 | 技术执行 | 本地环境 |
| 2. 同一样本对比 | 全员 | 效果对比表 |
| 3. 确定方案 | 项目负责人 | 技术决策文档 |

### 阶段 3：批量处理与质量检验（Week 4-8）

**目标**：完成 10,000 页识别

| 任务 | 负责人 | 产出 |
|------|--------|------|
| 1. 构建批处理脚本 | 技术执行 | 自动化脚本 |
| 2. 分批处理 | 技术执行 | 原始 JSON |
| 3. 质量抽查（10%） | 数据标注员 | 质检报告 |
| 4. 导出 Stata 格式 | 技术执行 | .dta 文件 |

### 阶段 4：微调优化（可选，Week 9+）

**目标**：进一步提升特殊字段识别率

---

## 九、团队 GitHub 协作规范

### 9.1 分支管理

```
main          # 稳定版本
├── dev       # 开发分支
│   ├── feature/api-test    # 功能分支
│   ├── feature/batch-process
│   └── fix/recognition-bug # 修复分支
```

### 9.2 Commit 规范

```bash
git commit -m "[feat] 添加批量处理脚本"
git commit -m "[fix] 修复金额字段解析错误"
git commit -m "[docs] 更新部署文档"
git commit -m "[test] 添加 API 测试用例"
```

### 9.3 Issue 模板

```markdown
## 问题类型
- [ ] 环境配置
- [ ] 识别错误
- [ ] 功能需求
- [ ] 其他

## 问题描述

## 复现步骤

## 截图/日志
```

---

## 十、预算与时间估算

### 10.1 团队时间投入

| 角色 | 阶段 1 | 阶段 2 | 阶段 3 | 阶段 4 | 合计 |
|------|--------|--------|--------|--------|------|
| 负责人 | 4h | 2h | 4h | 4h | 14h |
| 技术执行 | 8h | 8h | 20h | 10h | 46h |
| 标注员 | 2h | 4h | 30h | 10h | 46h |

### 10.2 金钱成本

| 项目 | 费用 | 备注 |
|------|------|------|
| 云端 API 测试 | ¥50 | 100 张 |
| 云端 API 复判 | ¥20 | 500 张疑难 |
| 云存储（可选） | ¥100/年 | 数据备份 |
| **合计** | **¥170** | 本地部署为主 |

---

## 十一、风险与应对

| 风险 | 可能性 | 影响 | 应对方案 |
|------|--------|------|----------|
| 本地模型效果差 | 中 | 高 | 回退到云端 API |
| 处理速度太慢 | 高 | 中 | 夜间批量 + 多台设备 |
| 特殊字无法识别 | 高 | 中 | 人工标注 + 微调 |
| 环境配置失败 | 中 | 高 | Docker/云端备选 |
| 数据隐私泄露 | 低 | 高 | 本地优先 + 脱敏处理 |

---

## 十二、快速启动清单

### 今天可以做的事（1-2 小时）

```bash
# macOS
python3 --version
uname -m
mkdir -p vlm_ocr/{samples,scripts,output}

# Windows
python --version
mkdir vlm_ocr\samples vlm_ocr\scripts vlm_ocr\output
```

### 第一周 Checkpoint

- [ ] 环境配置完成
- [ ] 10 张样本准备就绪
- [ ] 至少测试 2 个 API 服务
- [ ] 确定字段提取模板
- [ ] GitHub 仓库创建完成

---

## 附录 A：常见问题解答

### Q1: 本地模型识别率不如云端怎么办？
**A**: 优先使用云端验证可行性，确认后：
1. 尝试更大的本地模型（7B vs 2B）
2. 优化 prompt 提示词
3. 疑难样本单独用云端处理

### Q2: Windows 上 Ollama 无法启动？
**A**:
1. 检查是否有杀毒软件拦截
2. 以管理员身份运行
3. 尝试 WSL2 方案

### Q3: 处理速度太慢如何优化？
**A**:
1. 使用量化模型（4-bit）
2. 降低图片分辨率（300 DPI → 150 DPI）
3. 多设备并行处理

### Q4: 如何保证识别质量？
**A**:
1. 建立金标准样本集（50 张人工标注）
2. 每批处理前用金标准校验
3. 定期抽检（10% 比例）

---

## 附录 B：推荐学习资源

| 类型 | 链接 |
|------|------|
| Ollama 文档 | https://ollama.ai/docs |
| Hugging Face 课程 | https://huggingface.co/learn |
| Qwen-VL 使用指南 | https://github.com/QwenLM/Qwen-VL |
| GitHub 入门 | https://docs.github.com/en/get-started |

---

## 附录 C：项目日志模板

```markdown
## 2026-04-06
- 完成：项目规划文档 v2.0
- 问题：无
- 解决：-
- 下一步：环境配置、样本准备
- 参与人：XXX
```

---

## 下一步行动

**团队需要确认**：
1. [ ] 各成员 Mac/Windows 配置
2. [ ] 确定技术负责人
3. [ ] 创建 GitHub 仓库
4. [ ] 准备 10 张测试样本
5. [ ] 确定字段提取模板（地点、金额、交易项目）

然后开始第一周的环境配置与可行性测试。
