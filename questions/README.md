# 80 道 AI 候选题

这些题目是用 `cninfo_summary_crawler.py` 抓取的 50 份 2025 年年报摘要生成的**候选**题集，不是正式题集。

| 文件 | 说明 |
|---|---|
| `questions_s_80_candidates.jsonl` | 80 行，A20 / B20 / C15 / D15 / E10 |
| `questions_s_80_candidates.md` | 人工审阅表 |
| `questions_s_80_candidates.validation.md` | 机械校验结果 |

每行字段：

```json
{
  "id": "q301",
  "category": "A",
  "question": "...",
  "answer": "...",
  "evidence": [{"file": "603801_志邦家居/2025_年度报告摘要.md", "locator": "第 2 页，利润分配预案"}],
  "probe": "...",
  "notes": "...",
  "status": "ai_candidate",
  "official": false
}
```

## 重要

按主项目的 `questions/HUMAN_AUTHORING.md`，正式 80 题必须由人写/确认。这里所有题都标了：

```json
"status": "ai_candidate",
"official": false
```

正式使用前需要：

1. 逐题对照原文，核对答案和页码；
2. 改写题面；
3. 把 `status` 改为 `official`，或另存为 `official.jsonl`。

D 类的 `answer` 是“必须覆盖的要点”，不是最终长文答案。
