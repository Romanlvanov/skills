---
name: git-guardrails-claude-code
description: "Настраивает хуки Claude Code, которые блокируют опасные git-команды (push, reset --hard, clean, branch -D и т.п.) до их выполнения. Используй, когда пользователь хочет запретить разрушительные git-операции (prevent destructive git operations), добавить защитные git-хуки (add git safety hooks) или заблокировать git push/reset в Claude Code (block git push/reset in Claude Code)."
---

# Настройка защиты git

Настраивает хук PreToolUse, который перехватывает и блокирует опасные git-команды до того, как Claude их выполнит.

## Что блокируется

- `git push` (все варианты, включая `--force`)
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`

При блокировке Claude видит сообщение о том, что у него нет прав на эти команды.

## Шаги

### 1. Спроси область действия

Спроси пользователя: установить **только для этого проекта** (`.claude/settings.json`) или **для всех проектов** (`~/.claude/settings.json`)?

### 2. Скопируй скрипт хука

Скрипт в комплекте лежит здесь: [scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)

Скопируй его в нужное место в зависимости от области:

- **Проект**: `.claude/hooks/block-dangerous-git.sh`
- **Глобально**: `~/.claude/hooks/block-dangerous-git.sh`

Сделай его исполняемым через `chmod +x`.

### 3. Добавь хук в настройки

Добавь в соответствующий файл настроек:

**Проект** (`.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**Глобально** (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

Если файл настроек уже существует, влей хук в существующий массив `hooks.PreToolUse`. Не затирай другие настройки.

### 4. Спроси про доработку

Спроси, хочет ли пользователь добавить или убрать какие-то шаблоны из списка блокируемых. Отредактируй скопированный скрипт соответственно.

### 5. Проверь

Запусти быстрый тест:

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

Скрипт должен завершиться с кодом 2 и вывести сообщение BLOCKED в stderr.
