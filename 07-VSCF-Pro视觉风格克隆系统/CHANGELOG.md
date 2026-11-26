# VSCF-Pro 变更记录

## 2026-11-26｜通用JSON模板文件中文命名
- 将 `style-profile.template.json` 重命名为 `01_风格档案创建模板_STYLE_PROFILE.json`。
- 将 `theme-input.template.json` 重命名为 `02_主题迁移输入模板_THEME_INPUT.json`。
- 保留两个模板原有JSON字段和功能区别，不将通用模板绑定到单一STYLE档案。
- 更新模块README目录树，并补充两个模板的职责说明。

## 2026-11-26｜风格档案目录中文命名
- 将 `STYLE_001_Luminous_Painterly_Fantasy/` 重命名为 `STYLE_001_明亮通透的幻想环境绘画/`。
- 将 `STYLE_002_Conceptual_Red_Editorial/` 重命名为 `STYLE_002_高对比红色概念摄影海报/`。
- 保留STYLE_001、STYLE_002永久编号及原有JSON档案内容，不修改风格DNA。
- 同步更新模块README目录树和风格档案库命名规则。

## 1.0｜审核版
- 建立VSCF-Pro独立视觉风格克隆模块。
- 建立启动入口、超级系统提示词、架构文档、JSON Schema、跨平台指南和标准口令。
- 建立风格提取、主题迁移、主题扩展、风格校验四个工作流。
- 建立风格档案模板与主题输入模板。
- 纳入STYLE_001和STYLE_002两个单图初步提取示例。
- 两个示例档案均标记为provisional，尚未通过跨主题稳定性验证。
