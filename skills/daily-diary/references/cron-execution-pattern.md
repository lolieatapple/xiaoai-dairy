# Daily diary cron execution pattern

Reusable pattern validated in the 2026-05-23 diary run. Use this as a checklist/helper for autonomous diary cron jobs.

## One Python probe for context gathering

After confirming the date with `date +%Y-%m-%d` and reading the latest old diary, it is efficient to run one Python script that:

1. Fetches all RSS sources with `urllib.request` + `xml.etree.ElementTree`.
2. Strips HTML from titles/summaries and prints a compact JSON candidate list.
3. Loads `~/.hermes/.env` into `os.environ.setdefault(...)` before judging whether `WEREAD_API_KEY` exists.
4. Calls WeRead gateway with flat top-level params and `skill_version`.
5. Prints a compact summary of:
   - recent shelf entries from `/shelf/sync`
   - notebook overview from `/user/notebooks`
   - monthly reading clues from `/readdata/detail`
   - 2–3 active `/store/search` candidate directions for subjective book choice

This keeps the raw API noise out of the final diary while preserving enough evidence for a subjective selection.

## Mail/send/GitHub backup verification pattern

Use a single Python send script that:

1. Reads the diary with `Path(abs_path).read_text(encoding='utf-8')`, never from Hermes `read_file()` output.
2. Prints `repr(content[:100])` before sending.
3. Rejects empty content or any `(^|\n)\s*\d+\|` line-number prefix.
4. Sends to each recipient one at a time with `subprocess.run(..., stdout=PIPE, stderr=STDOUT)`.
5. Prints each recipient and return code, then calls `check_returncode()`.
6. Only after all sends succeed, appends the send-log line.
7. Copies the same `YYYY-MM-DD.md` into `~/path/to/xiaoai-dairy/diaries/`, commits `diaries/YYYY-MM-DD.md`, and pushes to `origin main` using the deploy-key SSH remote `git@github-xiaoai-dairy:lolieatapple/xiaoai-dairy.git`.

Finish with a separate verification script checking: absolute diary file exists, non-empty, no line-number prefix, send-log contains today's exact line, and `origin/main` contains `diaries/YYYY-MM-DD.md`.