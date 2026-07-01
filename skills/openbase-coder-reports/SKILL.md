---
name: openbase-coder-reports
description: >-
  Use this skill when listing, filtering, reading, tagging, or querying
  Openbase Coder report artifacts. Prefer the reports CLI/API over ad hoc
  filesystem searches.
version: 0.1.0
---

# Openbase Coder Reports

Use the first-class reports command surface whenever a user or agent needs to
find, inspect, or query report artifacts.

## When To Use

Use this skill when:

- the user asks to list, find, open, read, summarize, or filter reports
- the task mentions `.reports` folders across projects
- the task needs report metadata, ids, tags, projects, repos, or date grouping
- an agent is about to use `find`, `rg --files`, or raw filesystem scanning to
  discover reports
- the task touches the console Reports page or `/api/projects/reports/`
  endpoints

## Core Rule

Prefer:

```bash
openbase-coder reports ...
```

Do not use ad hoc filesystem searches as the default report discovery path. The
CLI and console share the same Python reports discovery/query layer, so they
agree on known report sources, ids, paths, metadata, tags, and modified-time
filter semantics.

## Main Commands

List reports across known project `.reports` folders:

```bash
openbase-coder reports list
openbase-coder reports list --limit 20
openbase-coder reports list --json
```

Filter by project, repo, tag, or modified-time range:

```bash
openbase-coder reports list --project openbase-coder-workspace
openbase-coder reports list --repo cli
openbase-coder reports list --tag Needs Review
openbase-coder reports list --when today
openbase-coder reports list --when yesterday
openbase-coder reports list --when 7d
openbase-coder reports list --since 2026-06-01 --until 2026-06-30
```

Show metadata for one report:

```bash
openbase-coder reports show REPORT_ID
openbase-coder reports show /absolute/project/.reports/summary.md
openbase-coder reports show REPORT_ID --json
```

Read or summarize markdown/text reports:

```bash
openbase-coder reports read REPORT_ID
openbase-coder reports read REPORT_ID --summary
openbase-coder reports read REPORT_ID --json
```

Relative report paths can be ambiguous across projects. If a relative path
matches more than one report, rerun with the full report id or absolute path.

## Date Semantics

Date filters use filesystem modified time. Filename dates such as
`2026-06-30-summary.md` are exposed as `filename_date` metadata only; they do
not control `--when`, `--since`, or `--until` filtering.

This is intentional because generated reports may be renamed, copied, or edited
after creation. If product behavior changes, update the shared reports query
layer before changing console or CLI behavior separately.

## Console And API

The console page is:

```text
/dashboard/reports
```

The relevant local API endpoints are:

```text
GET    /api/projects/reports/
GET    /api/projects/reports/all/
GET    /api/projects/reports/global/
GET    /api/projects/reports/file/
GET    /api/projects/reports/tags/
PATCH  /api/projects/reports/tags/
GET    /api/projects/reports/download/
POST   /api/projects/reports/action/
```

The console and CLI should delegate to
`cli/openbase_coder_cli/openbase_coder_cli_app/reports.py` instead of maintaining
separate scanners or metadata models.

## Source Locations

In the `openbase-coder-workspace` multi workspace, the main files are:

```text
cli/openbase_coder_cli/openbase_coder_cli_app/reports.py
cli/openbase_coder_cli/cli/reports.py
cli/tests/test_reports_api.py
cli/tests/test_reports_cli.py
coder-react/src/pages/Reports.tsx
coder-react/src/pages/ProjectDetail.tsx
coder-react/src/lib/reportGroups.ts
coder-react/src/lib/useReportFileActions.ts
```

## Validation

After changing reports behavior, run focused checks from the relevant subrepos:

```bash
cd cli
uv run pytest tests/test_reports_api.py tests/test_reports_cli.py
uv run ruff check openbase_coder_cli/openbase_coder_cli_app/reports.py openbase_coder_cli/cli/reports.py tests/test_reports_api.py tests/test_reports_cli.py

cd ../console
pnpm exec tsc -p tsconfig.json --noEmit
```
