# Как писать брифы для агента

Бриф для агента представляет собой структурированный комментарий, который публикуется на задаче (issue) GitHub или PR, когда она переходит в `ready-for-agent`. Он служит авторитетной спецификацией, по которой будет работать AFK-агент (агент, работающий сам). Исходное тело и обсуждение служат контекстом, а бриф для агента является контрактом.

Бриф описывает, **что должен сделать агент**, и это распространяется на оба случая: для задачи нужно построить изменение с нуля; для PR нужно сделать то, что осталось *с существующим диффом*: доделать его, закрыть пробелы, учесть замечания ревью. Принципы одни и те же; пример с PR ниже показывает разницу.

## Принципы

### Долговечность важнее точности

Задача может пролежать в `ready-for-agent` дни или недели. За это время кодовая база изменится. Пиши бриф так, чтобы он оставался полезным, даже когда файлы переименуют, переместят или отрефакторят.

- **Описывай** интерфейсы, типы и поведенческие контракты
- **Называй** конкретные типы, сигнатуры функций или формы конфигурации, которые агент должен найти или изменить
- **Не** ссылайся на пути к файлам: они устаревают
- **Не** ссылайся на номера строк
- **Не** предполагай, что текущая структура реализации останется прежней

### Поведение, а не процедура

Описывай, **что** должна делать система, а не **как** это реализовать. Агент заново исследует кодовую базу и сам примет решения о реализации.

- **Хорошо:** «Тип `SkillConfig` должен принимать необязательное поле `schedule` типа `CronExpression`»
- **Плохо:** «Открой src/types/skill.ts и добавь поле schedule в строке 42»
- **Хорошо:** «Когда пользователь запускает `/triage` без аргументов, он должен видеть сводку задач, требующих внимания»
- **Плохо:** «Добавь оператор switch в главную функцию-обработчик»

### Полные критерии приёмки

Агенту нужно знать, когда он закончил. У каждого брифа для агента должны быть конкретные проверяемые критерии приёмки. Каждый критерий должен проверяться независимо.

- **Хорошо:** «Запуск `gh issue list --label needs-triage` возвращает задачи, прошедшие начальную классификацию»
- **Плохо:** «Триаж должен работать правильно»

### Явные границы охвата

Укажи, что вне охвата. Это не даёт агенту заниматься лишней полировкой или строить предположения о соседних фичах.

## Шаблон

```markdown
## Agent Brief

**Category:** bug / enhancement
**Summary:** one-line description of what needs to happen

**Current behavior:**
Describe what happens now. For bugs, this is the broken behavior.
For enhancements, this is the status quo the feature builds on.

**Desired behavior:**
Describe what should happen after the agent's work is complete.
Be specific about edge cases and error conditions.

**Key interfaces:**
- `TypeName`: what needs to change and why
- `functionName()` return type: what it currently returns vs what it should return
- Config shape: any new configuration options needed

**Acceptance criteria:**
- [ ] Specific, testable criterion 1
- [ ] Specific, testable criterion 2
- [ ] Specific, testable criterion 3

**Out of scope:**
- Thing that should NOT be changed or addressed in this issue
- Adjacent feature that might seem related but is separate
```

## Примеры

### Хороший бриф для агента (баг)

```markdown
## Agent Brief

**Category:** bug
**Summary:** Skill description truncation drops mid-word, producing broken output

**Current behavior:**
When a skill description exceeds 1024 characters, it is truncated at exactly
1024 characters regardless of word boundaries. This produces descriptions
that end mid-word (e.g. "Use when the user wants to confi").

**Desired behavior:**
Truncation should break at the last word boundary before 1024 characters
and append "..." to indicate truncation.

**Key interfaces:**
- The `SkillMetadata` type's `description` field: no type change needed,
  but the validation/processing logic that populates it needs to respect
  word boundaries
- Any function that reads SKILL.md frontmatter and extracts the description

**Acceptance criteria:**
- [ ] Descriptions under 1024 chars are unchanged
- [ ] Descriptions over 1024 chars are truncated at the last word boundary
      before 1024 chars
- [ ] Truncated descriptions end with "..."
- [ ] The total length including "..." does not exceed 1024 chars

**Out of scope:**
- Changing the 1024 char limit itself
- Multi-line description support
```

### Хороший бриф для агента (улучшение)

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** Add `.out-of-scope/` directory support for tracking rejected feature requests

**Current behavior:**
When a feature request is rejected, the issue is closed with a `wontfix` label
and a comment. There is no persistent record of the decision or reasoning.
Future similar requests require the maintainer to recall or search for the
prior discussion.

**Desired behavior:**
Rejected feature requests should be documented in `.out-of-scope/<concept>.md`
files that capture the decision, reasoning, and links to all issues that
requested the feature. When triaging new issues, these files should be
checked for matches.

**Key interfaces:**
- Markdown file format in `.out-of-scope/`: each file should have a
  `# Concept Name` heading, a `**Decision:**` line, a `**Reason:**` line,
  and a `**Prior requests:**` list with issue links
- The triage workflow should read all `.out-of-scope/*.md` files early
  and match incoming issues against them by concept similarity

**Acceptance criteria:**
- [ ] Closing a feature as wontfix creates/updates a file in `.out-of-scope/`
- [ ] The file includes the decision, reasoning, and link to the closed issue
- [ ] If a matching `.out-of-scope/` file already exists, the new issue is
      appended to its "Prior requests" list rather than creating a duplicate
- [ ] During triage, existing `.out-of-scope/` files are checked and surfaced
      when a new issue matches a prior rejection

**Out of scope:**
- Automated matching (human confirms the match)
- Reopening previously rejected features
- Bug reports (only enhancement rejections go to `.out-of-scope/`)
```

### Хороший бриф для агента (PR)

Для PR раздел «Current behavior» описывает состояние диффа, а бриф просит агента доделать или исправить его, а не строить с нуля.

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** Finish the contributor's `--json` output flag for `triage list`

**Current behavior:**
The PR adds a `--json` flag that serializes the issue list to JSON. The happy
path works and the diff matches the project's command structure. Two gaps
remain: errors are still printed as human text (not JSON), and the new flag has
no test coverage.

**Desired behavior:**
With `--json`, all output (including errors) is well-formed JSON on stdout,
and the command's exit codes are unchanged. The existing human-readable output
is untouched when the flag is absent.

**Key interfaces:**
- The command's error path should emit `{ "error": string }` under `--json`
  instead of the plain-text error
- Reuse the existing serializer the PR already added; don't introduce a second

**Acceptance criteria:**
- [ ] `triage list --json` emits valid JSON for both success and error cases
- [ ] Exit codes match the non-JSON command
- [ ] A test covers the `--json` success output and one error case
- [ ] Default (non-JSON) output is byte-for-byte unchanged

**Out of scope:**
- Adding `--json` to any other command
- Changing the JSON shape of the success payload the PR already defined
```

### Плохой бриф для агента

```markdown
## Agent Brief

**Summary:** Fix the triage bug

**What to do:**
The triage thing is broken. Look at the main file and fix it.
The function around line 150 has the issue.

**Files to change:**
- src/triage/handler.ts (line 150)
- src/types.ts (line 42)
```

Он плох, потому что:
- Нет категории
- Расплывчатое описание («the triage thing is broken»)
- Ссылается на пути к файлам и номера строк, которые устареют
- Нет критериев приёмки
- Нет границ охвата
- Нет описания текущего и желаемого поведения
