---
title: How to disable surveys from Claude
date: '2026-10-05'
lastmod: '2026-10-05'
tags:
- Claude
- Claude Code
- guide
categories:
- AI
translationKey: disable-claude-code-feedback-survey-en
aliases:
- /2026-10-05-Disable-Claude-Code-feedback-survey-en/
---

Marketing surveys and other junk annoy me, especially when someone tries to push useless stuff on me. Claude has quite a lot of this. But it turns out that when you use it in Cursor, the "How is Claude doing this session?" surveys can be disabled. In the Claude Code app you [can't turn them off](https://github.com/anthropics/claude-code/issues/94710).

![Survey in Claude Code](claude-code-feedback-survey.png)

Here is my whole `.claude/settings.json` config, with the parameters described below. Take only what you need:

```json
{
  "cleanupPeriodDays": 99999,
  "env": {
    "CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY": "1",
    "DISABLE_FEEDBACK_COMMAND": "1",
    "DISABLE_ERROR_REPORTING": "1"
  },
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

The parameters:

- `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` and `DISABLE_FEEDBACK_COMMAND` — disable surveys.
- `cleanupPeriodDays` — how long sessions are kept. I set it to forever, since I see no point in archiving them.
- `DISABLE_ERROR_REPORTING` — disable error reports.
- `attribution` with empty values — prevent adding `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>` to commits and PRs on GitHub.

That's it. Share your own findings in comments, I'll be glad to read them.
