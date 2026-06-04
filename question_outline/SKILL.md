---
name: question-outline
description: >-
  将中学数学题目图片整理为解题大纲 JSON（id + outline 步骤数组 + keyKnowledge 知识点数组）。
  当用户发送数学题截图/照片，或提到 题目整理、解题大纲、question outline、题目 JSON 时使用。
  直接读图提取题干、条件、图形关系和解题过程，按给定样例结构输出，不依赖外部脚本。
---

# 题目图片 → 解题大纲 JSON

把用户发来的数学题图片整理成结构化「解题大纲 JSON」。目标不是生成互动教案，而是把题干、关键推理步骤、核心知识点压缩成可复用的解题 outline。

## 输入

- 用户在对话中发送题目图片、截图或扫描图。
- 图片可能包含：题干、图形、已给解答、批注、步骤推导。
- 若图片包含多个小问，优先处理用户指定的小问；未指定时按图片中最完整、最突出的小问处理。

## 输出格式（必守）

完整规范见 [references/output-format.md](references/output-format.md)。参考样例见 [references/example-Q002_part3_outline.json](references/example-Q002_part3_outline.json)。

```json
{
  "id": "Q002_part3",
  "outline": [
    {
      "step": 1,
      "title": "审题",
      "content": "Q 是 ⊙B（半径为 2）上的动点，B(3,0)，C(0,3)，A(-1,0)。求 4QC + 2QA 的最小值。"
    }
  ],
  "keyKnowledge": [
    {
      "name": "阿氏圆",
      "description": "圆上动点 Q 到两定点 B、A 距离之比 BQ/BA 为常数时，可构造辅助点 D 使 △BDQ ∽ △BQA，从而将系数转化为线段。"
    }
  ]
}
```

## 字段规则

### id

- 若用户提供编号，直接用。
- 若图片中能看出题号/小问，使用：`Q<题号>_part<小问号>`，如 `Q002_part3`。
- 若没有题号，用简短稳定 ID：`question_outline_001`。

### outline

- 数组，按解题逻辑顺序排列。
- 每步必须含：
  - `step`: 从 1 开始连续编号
  - `title`: 2~8 字概括动作，如「审题」「提取公因子」「构造辅助点 D」「证明相似」「线段替换」「求最小值」
  - `content`: 一句话写清本步的条件、计算或推理
- 步骤要**精炼但不断链**：保留必要计算、辅助构造、相似/全等证明、最值转化、最终结论。
- 不要照抄长篇解答；要把图片中的解法压缩成清晰链条。

### keyKnowledge

- 数组，提炼本题关键方法或知识点。
- 每项必须含：
  - `name`: 知识点名称，如「阿氏圆」「相似三角形（SAS）」「三点共线最值」
  - `description`: 该知识点在本题中的作用
- **不要**输出 `relatedSteps` 字段。

## 读图提取原则

1. **先还原题干**：坐标、函数、图形元素、动点、目标式、求值/证明要求。
2. **再整理解法**：优先采用图片中已经给出的解法；若图片只给题干，则自行补全标准解法。
3. **图形关系要文字化**：例如 `B(3,0)、C(0,3)、A(-1,0)`、`Q 是 ⊙B 半径为 2 上的动点`。
4. **公式保留关键等式**：如 `4QC + 2QA = 4(QC + ½QA)`、`QD = ½QA`、`CQ + QD ≥ CD`。
5. **最终步骤必须给出结论**：最小值、证明结果、答案。

## 禁止

- 不要输出 Markdown 代码块之外的解释，除非用户要求说明。
- 不要输出 `题目/步骤一/问题/答案` 这种互动教案结构（那是 lesson-plan-outline）。
- 不要把图片中的整段解答逐字 OCR 成一坨文本。
- 不要省略关键构造点、关键比例、相似依据、最终答案。
- 不要编造图片中不存在且解题不需要的条件。
- 不要输出 `keyKnowledge` 中的 `relatedSteps` 字段。

## 交付方式

- 默认在回复中直接给出完整 JSON。
- 用户要求保存时：
  - 若指定路径，写入该路径。
  - 未指定路径，则写入当前工作区 `output/json/<id>_outline.json`；没有该目录时写入 `%USERPROFILE%\Documents\question-outline-output\<id>_outline.json`。

## 自检

- [ ] JSON 可解析，无注释、无尾逗号
- [ ] `outline.step` 从 1 连续编号
- [ ] 每步 title 简短、content 完整
- [ ] `keyKnowledge` 每项只有 `name` 和 `description`
- [ ] 最后一步包含最终答案/结论
- [ ] 与参考样例结构一致
