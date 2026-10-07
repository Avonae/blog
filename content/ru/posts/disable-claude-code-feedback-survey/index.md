---
title: Как отключить опросы в Claude Code
date: '2026-10-05'
lastmod: '2026-10-05'
tags:
- Claude
- Claude Code
- инструкция
categories:
- ИИ
translationKey: disable-claude-code-feedback-survey-en
aliases:
- /2026-10-05-Disable-Claude-Code-feedback-survey/
---

Меня напрягают маркетинговые опросы и другой мусор, когда мне пытаются втюхать ненужную херню. У клода этого довольно много. Но оказалось, что при использовании клода в курсоре опросы «Как вам сегодня работа?» можно отключить. В приложении Claude Code [отключить это нельзя](https://github.com/anthropics/claude-code/issues/94710).

![Опрос в Claude Code](claude-code-feedback-survey.png)

Я приложил весь свой конфиг `.claude/settings.json` с описанием параметров ниже, можете взять только нужное:

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

Параметры тут:

- `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` и `DISABLE_FEEDBACK_COMMAND` — отключение опросов.
- `cleanupPeriodDays` — срок хранения сессий. Я поставил бесконечный, т.к. не вижу смысла их архивировать.
- `DISABLE_ERROR_REPORTING` — отключить отчеты об ошибках.
- `attribution` с пустыми значениями — запрет добавления `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>` к коммитам и PR на гитхабе.

Вот и все. Делитесь своими находками в комментариях, с интересом почитаю.
