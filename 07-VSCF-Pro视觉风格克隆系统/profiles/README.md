# STYLE Library｜风格档案库

本目录保存经过VSCF-Pro提取的独立STYLE DNA。

## 命名规则

风格档案文件夹统一使用「永久编号_中文风格名称」格式。浏览目录时，可直接通过中文名称识别风格；STYLE编号作为唯一身份ID，不因文件夹改名而改变。

```text
STYLE_001_中文风格名称/
├── style-profile.json
├── prompt-cn.json
├── prompt-en.json
├── prompt-simple-cn.md
├── prompt-natural-language.md
└── validation/
```

例如：

- `STYLE_001_明亮通透的幻想环境绘画/`
- `STYLE_002_高对比红色概念摄影海报/`

## 档案状态

- provisional：初步提取，尚待用户审核；
- reviewed：关键字段已由用户审核；
- tested：已完成真实生成和跨主题测试；
- stable：多样本、多主题验证稳定。

## 管理规则

1. 每种风格独立编号。
2. 不同风格不得合并成一个档案。
3. 主题变化不创建新风格ID。
4. 核心DNA变化时升级档案版本。
5. 原始参考图默认不入库；保存前需确认。
6. 文件夹名称采用「STYLE编号_中文风格名称」；修改中文名称不得改变STYLE编号或风格DNA。
