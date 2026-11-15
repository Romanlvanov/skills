# Трекер задач: GitHub

Задачи и спецификации этого репозитория живут как задачи (issues) GitHub. Для всех операций используй CLI `gh`.

## Соглашения

- **Создать задачу**: `gh issue create --title "..." --body "..."`. Для многострочного тела используй heredoc.
- **Прочитать задачу**: `gh issue view <number> --json number,title,body,labels,comments`.
- **Список задач**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` с подходящими фильтрами `--label` и `--state`.
- **Сделать задачу подзадачей родителя**: `gh issue create --parent <parent> ...` или потом `gh issue edit <parent> --add-sub-issue <child>` (`gh` 2.94+). Старый `gh`: `gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>` (database id, как в пункте **Блокировка** ниже). Без подзадач поставь `Part of #<parent>` в начало тела дочерней задачи.
- **Блокировка**: **нативные зависимости задач** GitHub, каноническое и видимое в интерфейсе представление. Добавь связь через `gh issue create --blocked-by <n>,<n> ...` или `gh issue edit <child> --add-blocked-by <blocker>` (`gh` 2.94+). Старый `gh`: `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, где `<blocker-db-id>` это числовой **database id** блокирующей задачи (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _не_ `#number` и не `node_id`). Где зависимости недоступны, используй запасной вариант: строку `Blocked by: #<n>, #<n>` в начале тела дочерней задачи. Тикет разблокирован, когда закрыт каждый блокирующий.
- **Посчитать открытые блокирующие**: `gh api repos/<owner>/<repo>/issues/<n> --jq .issue_dependencies_summary.blocked_by` (только открытые блокирующие; 0 значит, что можно начинать). `gh issue view <n> --json blockedBy` выводит и закрытые блокирующие, поэтому фильтруй: `--jq '[.blockedBy.nodes[] | select(.state == "OPEN")] | length'`.
- **Прокомментировать задачу**: `gh issue comment <number> --body "..."`
- **Добавить / снять метки**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Закрыть**: опубликуй объяснение через `gh issue comment <number> --body "..."`, затем `gh issue close <number>`. На уже закрытой задаче `gh issue close --comment` теряет комментарий и всё равно завершается с кодом 0.

Определяй репозиторий по `git remote -v`; `gh` делает это автоматически при запуске внутри клона.

Заводи задачи только в собственный репозиторий пользователя. Никогда не заводи их в `mattpocock/skills` (репозиторий, откуда эти скиллы), если только он не принадлежит пользователю. В форке `gh` может по умолчанию выбрать родительский upstream, поэтому перед первым `gh issue create` в сессии проверь цель через `gh repo view --json nameWithOwner` и, если это не репозиторий пользователя, согласуй с ним.

## Pull request как поверхность триажа

**PRs as a request surface: no.** _(Поставь `yes`, если этот репозиторий считает внешние PR запросами на фичи; этот флаг читает `/triage`.)_

Когда стоит `yes`, PR проходят через те же метки и состояния, что и задачи, с аналогами из `gh pr`:

- **Прочитать PR**: `gh pr view <number>`, затем `gh pr view <number> --comments` (без TTY `--comments` выводит только комментарии, без тела) и `gh pr diff <number>` для диффа.
- **Список внешних PR для триажа**: `gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`.
- **Комментарий / метка / закрытие**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

У GitHub одно пространство номеров для задач и PR, поэтому голый `#42` может быть и тем, и другим: определяй через `gh pr view 42`, а при неудаче через `gh issue view 42`.

## When a skill says "publish to the issue tracker"

(Когда скилл говорит «опубликуй в трекер задач».) Создай задачу на GitHub.

## When a skill says "fetch the relevant ticket"

(Когда скилл говорит «получи нужный тикет».) Прочитай его, как в пункте **Прочитать задачу** выше.

## Wayfinding operations

Используется `/wayfinder`. **Карта** (map) это одна задача, а тикеты это её **дочерние** (child) задачи.

- **Карта**: одна задача с меткой `wayfinder:map`, в теле которой Notes / Decisions-so-far / Fog. `gh issue create --label wayfinder:map`.
- **Дочерний тикет**: задача, связанная с картой как подзадача GitHub (см. **Сделать задачу подзадачей родителя**). Где подзадачи не включены, добавь дочернюю задачу в список задач (task list) в теле карты и поставь `Part of #<map>` в начало тела дочерней задачи. Метки: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Забранный тикет назначается на ведущего разработчика.
- **Блокировка**: см. **Блокировка** в разделе «Соглашения».
- **Запрос фронта** (frontier query): выведи открытые дочерние задачи карты (`gh issue list --state open`, в пределах подзадач / списка задач карты), отбрось те, у которых есть открытый блокирующий (см. **Посчитать открытые блокирующие** или открытую задачу в строке `Blocked by`) или исполнитель; побеждает первая в порядке карты.
- **Забрать** (claim): `gh issue edit <n> --add-assignee @me`, первая запись в сессии.
- **Решить** (resolve): `gh issue comment <n> --body "<answer>"`, затем `gh issue close <n>`, затем допиши контекстный указатель (суть + ссылка) в Decisions-so-far карты.
