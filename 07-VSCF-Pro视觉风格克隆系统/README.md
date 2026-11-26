# VSCF-Pro｜视觉风格克隆系统

**版本：** 1.0（审核版）  
**英文名称：** Visual Style Cloning Framework — Professional  
**模块定位：** Xinxi 知识库中的独立视觉风格提取、结构化建模、主题迁移与提示词复用工具。

## 1. 模块目标

VSCF-Pro 将参考图中的可观察视觉规律转化为可复用的风格档案（Style DNA），并将风格规则与主题内容分离。系统支持：

- 单图或多图风格分析；
- 13个视觉控制系统与叙事身份分析；
- 风格恒定变量、主题变量、应用变量分层；
- 中文JSON、英文JSON、中文简易提示词、自然语言提示词四种输出；
- 风格迁移、主题扩展、主题变更；
- 风格一致性校验与偏差诊断；
- 风格档案编号、版本管理和复用。

> 核心原则：迁移视觉生成规则，不机械复制参考图中的具体内容。

## 2. 快速入口

第一次使用：阅读 [START_HERE.md](START_HERE.md)。

执行引擎：[MASTER_SYSTEM_PROMPT.md](MASTER_SYSTEM_PROMPT.md)。

系统方法：[SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md)。

跨平台配置：[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)。

标准调用口令：[QUICK_COMMANDS.md](QUICK_COMMANDS.md)。

## 3. 目录结构

```text
07-VSCF-Pro视觉风格克隆系统/
├── README.md
├── START_HERE.md
├── MASTER_SYSTEM_PROMPT.md
├── SYSTEM_ARCHITECTURE.md
├── JSON_SCHEMA.json
├── INTEGRATION_GUIDE.md
├── QUICK_COMMANDS.md
├── REPO_INTEGRATION_PATCH.md
├── CHANGELOG.md
├── workflows/
│   ├── 01-风格提取.md
│   ├── 02-主题迁移.md
│   ├── 03-主题扩展.md
│   └── 04-风格校验.md
├── templates/
│   ├── 01_风格档案创建模板_STYLE_PROFILE.json
│   └── 02_主题迁移输入模板_THEME_INPUT.json
└── profiles/
    ├── README.md
    ├── STYLE_001_明亮通透的幻想环境绘画/
    │   └── style-profile.json
    └── STYLE_002_高对比红色概念摄影海报/
        └── style-profile.json
```

## 4. 两种通用模板的用途

| 文件 | 用途 | 主要使用时机 |
|---|---|---|
| `01_风格档案创建模板_STYLE_PROFILE.json` | 定义风格档案的数据结构，保存STYLE DNA | 提取新风格、创建新的STYLE档案 |
| `02_主题迁移输入模板_THEME_INPUT.json` | 输入新主题及生成要求，并指定调用的STYLE ID | 使用已有风格迁移主题、扩展主题系列 |

这两个文件都是系统级通用模板，不分别对应某一个STYLE档案。所有风格档案共用风格档案创建模板；所有主题迁移任务共用主题迁移输入模板。

## 5. 证据与质量边界

- 单张参考图只能形成初步风格档案，不能证明跨作品稳定性。
- 观察事实、分析推断和待确认信息必须分开记录。
- 评分是辅助诊断，不是客观测量。
- 跨主题稳定性必须通过实际生成结果验证。
- 不同模型的图像生成能力、文字渲染和参数解释不同；统一JSON不代表像素级结果相同。
- 用户上传的参考图片默认不随档案入库；如需保存原图，应另行确认版权、来源和存储范围。

## 6. 版本原则

- 只更换主题，不修改风格档案版本。
- 修改核心风格DNA，必须升级风格档案版本并记录原因。
- 模型专属参数单独存放，不污染通用风格DNA。
- 四种提示词必须从同一份已确认的风格档案派生。
