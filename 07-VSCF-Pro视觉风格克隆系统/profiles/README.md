# STYLE Library｜风格档案库

本目录保存经过VSCF-Pro提取的独立STYLE DNA。

## 命名规则

```text
STYLE_001_English_Style_Name/
├── README.md
├── style-profile.json
├── prompt-cn.json
├── prompt-en.json
├── prompt-simple-cn.md
├── prompt-natural-language.md
└── validation/
```

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
