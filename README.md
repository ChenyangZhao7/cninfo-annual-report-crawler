# CNINFO Annual Report Crawler

**English** | [中文](README.zh-CN.md)

Crawls A-share **annual report summaries** from CNINFO (cninfo.com.cn) and converts each PDF's text layer into Markdown page by page. Ships with an investor Q&A miner for the Shenzhen and Shanghai exchanges. Output:

- one merged Markdown file (good for reading and full-text search);
- one Markdown file per summary;
- `manifest.jsonl`: source URL, fetch time, sha256, page count, character count;
- `REPORT.md`: crawl and sampling report;
- `raw/`: the original PDFs (kept by default, `--no-raw` deletes them).

**Format conversion only: no LLM, no summarising, no rewriting, no translation.**

## Environment

- Python 3.9+
- Dependency: `pypdf`

```bash
pip install -r requirements.txt
```

## Quick start

Fetch 50 summaries for 2025:

```bash
python cninfo_summary_crawler.py --year 2025 --target 50
```

Output goes to `cninfo_output/`:

```
cninfo_output/
  50_summaries.md          # merged
  summaries/               # 50 single files
  manifest.jsonl
  REPORT.md
  raw/                     # original PDFs
```

List candidates without downloading:

```bash
python cninfo_summary_crawler.py --year 2025 --target 50 --dry-run
```

Sample 50 at random (the default is largest-first):

```bash
python cninfo_summary_crawler.py --year 2025 --target 50 --random
```

Widen the candidate pool to make sure 50 are found:

```bash
python cninfo_summary_crawler.py --year 2025 --target 50 --pool 120 --max-pages 15
```

Delete the original PDFs after conversion:

```bash
python cninfo_summary_crawler.py --year 2025 --target 50 --no-raw
```

## Options

| Option | Default | Meaning |
|---|---:|---|
| `--year` | last year | Reporting period, e.g. `2025` |
| `--target` | 50 | How many summaries to deliver |
| `--pool` | 90 | Candidate companies; must be ≥ target |
| `--max-pages` | 12 | Maximum metadata pages to page through |
| `--out` | `cninfo_output` | Output directory |
| `--random` | off | Pick `target` at random; the default prefers larger files |
| `--include-st` | off | Include ST / delisting-risk companies |
| `--late` | off | Extend the window to December of the following year, including re-disclosures and corrections |
| `--min-chars` | 2000 | Minimum characters per summary |
| `--min-chars-per-page` | 100 | Minimum characters per page; filters out scans |
| `--interval` | 1.0 | Seconds between PDF requests |
| `--timeout` | 120 | Download timeout in seconds |
| `--retries` | 3 | Request retries |
| `--no-raw` | off | Delete the original PDF after conversion |
| `--dry-run` | off | List candidates only, do not download |

## Selection logic

1. Query annual report summaries with `category_ndbg_szsh` + `searchkey=年度报告摘要`.
2. The default window runs from December of the reporting year to 30 April of the next; `--late` extends it to December.
3. Keep only announcements whose period matches and whose title contains "年度报告摘要".
4. Exclude: half-year reports, English versions, cancelled filings, "关于…更正公告" notices themselves, and ST / delisting-risk companies.
5. One version per company per period; "更正后 / 更新后" summaries are kept.
6. By default take the `target` largest by Markdown bytes, so 50 summaries land near 1.5 MB.
7. `--random` samples with a fixed seed.

## Output format

The merged file opens with a table of contents; each summary looks like:

```markdown
<!-- document: 000001 平安银行 | 2025 | 年度报告摘要 | https://... | sha256: ... -->
# 000001 平安银行 — 2025 年年度报告摘要

- 来源：https://...
- 抓取时间：2026-11-02T18:00:00+08:00
- 页数：14；字符：12,561；大小：25.5 KB

<!-- page: 1 -->
...原文...
```

- UTF-8 without BOM, LF line endings.
- Each page keeps `<!-- page: N -->`.
- Each file keeps `source_url`, `retrieved_at` and `sha256`.
- Body text comes from the PDF text layer; tables are kept as text lines, not yet structured Markdown tables.

## Investor Q&A miner (optional)

`cninfo_question_miner.py` crawls investor questions and reports what people ask most:

- Shenzhen / Beijing: 深交所互动易
- Shanghai: 上证e互动
- Outputs each company's questions, answers, dates and source links, plus the most frequent topics in the sample.

```powershell
# Use the manifest produced by the crawler (recommended)
python cninfo_question_miner.py --manifest cninfo_output/manifest.jsonl

# Or supply your own CSV
# company_code,company_name
# 000001,平安银行
# 600519,贵州茅台
python cninfo_question_miner.py --companies companies.csv
```

Output:

```text
questions_output/
  questions.md        # frequent topics + per-company questions
  questions.jsonl     # every question, answer, date and link
  REPORT.md
```

Common options:

| Option | Default | Meaning |
|---|---:|---|
| `--limit-per-company` | 30 | Maximum questions per company; 0 = unlimited |
| `--max-pages` | 3 | Maximum pages per company |
| `--top-per-company` | 10 | How many questions to show per company in Markdown |
| `--top-keywords` | 20 | How many frequent topics to show |
| `--interval` | 1.0 | Seconds between requests |

Notes:

- "Frequent topics" is fixed-vocabulary string counting with **no LLM**, so it is reproducible.
- Questions are sorted by likes then date; 互动易 likes are usually 0, so date dominates.
- 上证e互动 questions come from company page HTML and answers are not fetched yet; 互动易 includes answers.
- Only public Q&A is crawled; no login or CAPTCHA is bypassed.

## Known limitations

- Scanned PDFs have no text layer and are skipped; no OCR.
- Tables may be split across lines by the PDF text layer and read worse than the original PDF.
- CNINFO endpoints, fields and anti-bot behaviour can change; long-term availability is not guaranteed.
- Only public filings; no login, CAPTCHA or paywall is bypassed.

## Compliance

- Intended for organising and researching public material.
- Single-threaded by default with a one-second interval; backs off on 403/429.
- Confirm CNINFO's terms of service and local law before uploading or publishing results.
- Any redistribution of unrelated data is the user's responsibility.

## Example question set

`questions/` holds **80 candidate AI questions** generated from 50 report summaries:

- `questions/questions_s_80_candidates.jsonl`
- `questions/questions_s_80_candidates.md`
- `questions/questions_s_80_candidates.validation.md`

These are candidates, not a finished benchmark; answers, page numbers and wording need human review before use.

## License

MIT
