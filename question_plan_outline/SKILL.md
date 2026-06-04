---
name: question-plan-outline
description: >-
  将 question-outline 生成的解题大纲 JSON 转为互动解题教案 JSON（题目 + 步骤一/二…，含问题/答案/错误提示/历史记录/动效描述）。
  当用户提供题目解题步骤 JSON、说题目教案、解题互动步骤、question plan、question-plan-outline 时使用。
---

# 解题大纲 JSON → 互动解题教案 JSON

把 **question-outline** 产出的解题大纲 JSON，转成可用于互动课堂的分步解题教案 JSON。

## 输入 / 输出

| | 说明 |
|---|---|
| 输入 | `id` + `outline[]` + `keyKnowledge[]` 的解题大纲 JSON |
| 输出 | 一个互动解题教案 JSON |
| 命名 | 默认 `<id>_plan.json`，如 `Q002_part3_plan.json` |

## 参考结构（必守）

细则见 [references/output-format.md](references/output-format.md)，样例见 [references/example-Q002_part3_input.json](references/example-Q002_part3_input.json)、[references/example-Q002_part3_plan.json](references/example-Q002_part3_plan.json)。

```json
{
  "题目": "本题要解决的核心问题（一句话）",
  "步骤一": {
    "问题": "向学生提出的互动问题（可含填空 ______）",
    "答案": "标准答案；多空用分号分隔",
    "错误提示": "第一次错误：……；第二次错误：……；第三次错误：直接显示正确答案",
    "历史记录": "到本步为止已建立的解题结论",
    "动效描述": "本步正确后屏幕/图形动画说明；无则写「无」"
  },
  "步骤二": { "...": "..." }
}
```

- 步骤键名必须是：`步骤一`、`步骤二`、`步骤三`……（中文数字）。
- 输出中不要保留输入字段：`id`、`outline`、`keyKnowledge`、`step`、`title`、`content`。
- 每个步骤对象必须包含且只包含：`问题`、`答案`、`错误提示`、`历史记录`、`动效描述`。

## 转换规则

### 题目

- 从 `outline[0].content` 提炼本题核心任务。
- 保留已知条件和求解目标，避免写成长段题干。

### 步骤

- 默认一个 `outline[]` 步骤转换成一个互动步骤，顺序一致。
- `title` 只作为设计问题的依据，不直接输出。
- `content` 改写成学生可回答的问题：填空、计算、判断、补全过程、说明理由。
- 标准答案要短，可判分；多空用分号分隔。
- 错误提示分三层：第一次给方向，第二次给关键关系或式子，第三次显示正确答案。
- 历史记录采用累积式：记录本步之前和本步已经确定的关键结论。
- 动效描述要服务解题过程：点线高亮、辅助线出现、公式变形、最值线段重合等；没有图形时写 `无`。

### 知识点

- `keyKnowledge[]` 不单独输出为字段。
- 需要把关键知识点自然融入对应步骤的问题、答案、错误提示或历史记录中。

## 禁止

- 不要输出 `outline` 格式。
- 不要输出 `keyKnowledge`、`relatedSteps`。
- 不要输出互动概念课以外的额外说明。
- 不要逐字照搬解题过程，要改写成可交互提问。
- 不要省略最终答案或取等号条件。

## 工作流

1. 接收输入：用户粘贴 JSON、@ 文件，或指定 `output/json/xxx_outline.json`。
2. 读取并理解 `outline[]` 的解题链条。
3. 转换为 `题目 + 步骤一/二/...` 的互动解题 JSON。
4. 用户要求保存时：
   - 若指定目录，写入该目录。
   - 若输入文件在 `json/` 目录，默认写回同目录：`<id>_plan.json`。
   - 未指定且找不到输入目录时，写入 `%USERPROFILE%\Documents\question-plan-output\<id>_plan.json`。

## 自检

- [ ] JSON 可解析，无注释、无尾逗号
- [ ] 顶层只有 `题目` 和连续的 `步骤一/二/...`
- [ ] 每步只有 `问题`、`答案`、`错误提示`、`历史记录`、`动效描述`
- [ ] 不含 `id`、`outline`、`keyKnowledge`、`relatedSteps`
- [ ] 最后一步包含最终答案
- [ ] 若是最值题，写清取等号条件
