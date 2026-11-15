# Трекер задач: GitLab

Задачи и спецификации этого репозитория живут как задачи (issues) GitLab. Для всех операций используй CLI [`glab`](https://gitlab.com/gitlab-org/cli).

## Соглашения

- **Создать задачу**: `glab issue create --title "..." --description "..."`. Для многострочного описания используй heredoc. Передай `--description -`, чтобы открыть редактор.
- **Прочитать задачу**: `glab issue view <number> --comments`. Для машиночитаемого вывода используй `-F json`.
- **Список задач**: `glab issue list -O json` с подходящими фильтрами `--label`.
- **Прокомментировать задачу**: `glab issue note <number> --message "..."`. В GitLab комментарии называются «notes».
- **Добавить / снять метки**: `glab issue update <number> --label "..."` / `--unlabel "..."`. Несколько меток можно перечислить через запятую или повторив флаг.
- **Закрыть**: `glab issue close <number>`. `glab issue close` не принимает закрывающий комментарий, поэтому сначала опубликуй объяснение через `glab issue note <number> --message "..."`, затем закрой.
- **Merge request**: в GitLab PR называются «merge requests». Используй `glab mr create`, `glab mr view`, `glab mr note` и т.д., по той же схеме, что `gh pr ...`, но с `mr` вместо `pr` и `note`/`--message` вместо `comment`/`--body`.

Определяй репозиторий по `git remote -v`; `glab` делает это автоматически при запуске внутри клона.

## Merge request как поверхность триажа

**MRs as a request surface: no.** _(Поставь `yes`, если этот репозиторий считает внешние merge request запросами на фичи; этот флаг читает `/triage`.)_

Когда стоит `yes`, MR проходят через те же метки и состояния, что и задачи, с аналогами из `glab mr`:

- **Прочитать MR**: `glab mr view <number> --comments` и `glab mr diff <number>` для диффа.
- **Список внешних MR для триажа**: `glab mr list -O json`, затем оставь только MR, автор которых не участник/владелец проекта (MR контрибьютора, а не текущая работа мейнтейнера).
- **Комментарий / метка / закрытие**: `glab mr note`, `glab mr update --label`/`--unlabel`, `glab mr close`.

В отличие от GitHub, GitLab нумерует задачи и MR раздельно, поэтому `#42` однозначен, как только ты знаешь, какую поверхность имеет в виду мейнтейнер.

## When a skill says "publish to the issue tracker"

(Когда скилл говорит «опубликуй в трекер задач».) Создай задачу в GitLab.

## When a skill says "fetch the relevant ticket"

(Когда скилл говорит «получи нужный тикет».) Выполни `glab issue view <number> --comments`.

## Wayfinding operations

Используется `/wayfinder`. **Карта** (map) это одна задача, а тикеты это её **дочерние** (child) задачи.

- **Карта**: одна задача с меткой `wayfinder:map`, в теле которой Notes / Decisions-so-far / Fog. `glab issue create --label wayfinder:map`. (На тарифах GitLab с нативными эпиками карту может хранить эпик; задача с меткой работает везде.)
- **Дочерний тикет**: задача со строкой `Part of #<map>` в начале описания, связанная с картой (`glab issue create --linked-issues <map> --link-type relates_to`), с метками `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Забранный тикет назначается на ведущего разработчика.
- **Блокировка**: **нативная блокирующая связь** GitLab, каноническое и видимое в интерфейсе представление. Добавь её быстрым действием `/blocked_by #<n>`, опубликованным как note (`glab issue note <child> --message "/blocked_by #<blocker>"`). Нативные блокирующие связи есть только в Premium/Ultimate; на бесплатном тарифе (или где они недоступны) используй запасной вариант: строку `Blocked by: #<n>, #<n>` в начале описания. Тикет разблокирован, когда закрыт каждый блокирующий.
- **Запрос фронта** (frontier query): выведи дочерние задачи карты через `glab api projects/:id/issues/<map>/links`, оставив открытые задачи с меткой `wayfinder:<type>`; отбрось те, у которых есть открытый блокирующий (нативная связь `blocked_by` на открытую задачу (`glab api projects/:id/issues/<child-iid>/links`) или открытая задача в строке `Blocked by`) или исполнитель; побеждает первая в порядке карты.
- **Забрать** (claim): `glab issue update <n> --assignee @me`, первая запись в сессии.
- **Решить** (resolve): `glab issue note <n> --message "<answer>"`, затем `glab issue close <n>`, затем допиши контекстный указатель (суть + ссылка) в Decisions-so-far карты.
