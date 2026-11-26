# 跨平台使用与集成指南

## 1. 通用原则

GitHub仓库负责保存、版本化和分发系统文件；AI平台负责读取系统规则、分析图片和生成内容。**仅仅把文件上传到GitHub，不会自动让任何AI平台启用该系统。**

要使用VSCF-Pro，必须让目标AI会话实际读取 `MASTER_SYSTEM_PROMPT.md`，并在需要时读取对应STYLE档案。

## 2. ChatGPT

推荐方式：
1. 创建一个专门的ChatGPT Project（如果当前账号界面提供该功能）。
2. 将 `MASTER_SYSTEM_PROMPT.md` 的内容放入项目指令；如果指令长度受限，将完整文件作为项目资料，并在指令中要求每次执行前读取它。
3. 将 `SYSTEM_ARCHITECTURE.md`、`JSON_SCHEMA.json` 和常用STYLE档案作为项目资料。
4. 上传参考图并发送 `VSCF:CLONE`。
5. 后续新主题发送 `VSCF:TRANSFER` 并指定STYLE ID。

如果当前对话已连接GitHub并能读取该仓库，也可以要求AI读取仓库中的指定文件；必须确认AI确实读到了文件内容。不能访问仓库时，改为上传文件。

## 3. Gemini、豆包及其他支持系统指令/知识库的AI

1. 将MASTER_SYSTEM_PROMPT作为系统指令或智能体指令。
2. 将架构文档、JSON Schema和STYLE档案加入知识库（若平台支持）。
3. 上传参考图。
4. 使用START_HERE中的标准口令。
5. 如果平台不支持长期知识库，则在每次新会话中附上系统提示词和目标STYLE档案。

## 4. Lovart及以提示词为主的图像工具

如果工具不支持完整系统指令：
1. 在分析型AI中先运行VSCF-Pro，得到确认后的STYLE DNA。
2. 导出“中文简易提示词”或“自然语言提示词”。
3. 将提示词与新主题、参考图一起输入图像工具。
4. 对生成结果回到VSCF-Pro执行VALIDATE。

## 5. 跨平台一致性

通用STYLE DNA只存视觉规则。以下参数应按平台单独保存：
- 模型名称和版本；
- 画幅、分辨率；
- 参考图权重或图像引导参数；
- seed、stylize等模型专属参数；
- 负面提示词的专属格式。

不得假定一个平台的专属参数在另一个平台具有相同含义。

## 6. 维护建议

- 将确认后的STYLE档案保存到 `profiles/STYLE_xxx/`。
- 每个STYLE目录包含 `style-profile.json` 和必要的 `README.md`。
- 如需保存参考图，先确认来源、授权和是否适合公开仓库。
- 更新MASTER提示词或Schema时，升级模块版本并记录CHANGELOG。
