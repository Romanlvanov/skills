---
name: scaffold-exercises
description: Создаёт структуру каталогов упражнений с разделами, заданиями, решениями и пояснениями, которая проходит линтинг. Используй, когда пользователь хочет сгенерировать каркас упражнений (scaffold exercises), создать заготовки упражнений (create exercise stubs) или завести новый раздел курса (set up a new course section).
---

# Каркас упражнений

Создай структуру каталогов упражнений, которая проходит `pnpm ai-hero-cli internal lint`, затем закоммить через `git commit`.

## Именование каталогов

- **Разделы**: `XX-section-name/` внутри `exercises/` (например, `01-retrieval-skill-building`)
- **Упражнения**: `XX.YY-exercise-name/` внутри раздела (например, `01.03-retrieval-with-bm25`)
- Номер раздела = `XX`, номер упражнения = `XX.YY`
- Имена в dash-case (строчные буквы, дефисы)

## Варианты упражнений

Каждому упражнению нужна хотя бы одна из этих подпапок:

- `problem/` - рабочее место студента с TODO
- `solution/` - эталонная реализация
- `explainer/` - концептуальный материал, без TODO

При создании заготовок по умолчанию используй `explainer/`, если в плане не указано иное.

## Обязательные файлы

В каждой подпапке (`problem/`, `solution/`, `explainer/`) нужен `readme.md`, который:

- **Не пустой** (должно быть реальное содержимое, хватит даже одной строки с заголовком)
- Не содержит битых ссылок

При создании заготовок делай минимальный readme с заголовком и описанием:

```md
# Exercise Title

Description here
```

Если в подпапке есть код, в ней также нужен `main.ts` (больше 1 строки). Но для заготовок упражнение только с readme допустимо.

## Порядок работы

1. **Разбери план** - извлеки названия разделов, названия упражнений и типы вариантов
2. **Создай каталоги** - `mkdir -p` для каждого пути
3. **Создай заготовки readme** - по одному `readme.md` с заголовком в каждой папке варианта
4. **Запусти линтер** - `pnpm ai-hero-cli internal lint` для проверки
5. **Исправь ошибки** - повторяй, пока линтер не пройдёт

## Сводка правил линтера

Линтер (`pnpm ai-hero-cli internal lint`) проверяет:

- У каждого упражнения есть подпапки (`problem/`, `solution/`, `explainer/`)
- Существует хотя бы одна из `problem/`, `explainer/` или `explainer.1/`
- В основной подпапке есть непустой `readme.md`
- Нет файлов `.gitkeep`
- Нет файлов `speaker-notes.md`
- Нет битых ссылок в readme
- Нет команд `pnpm run exercise` в readme
- `main.ts` обязателен в каждой подпапке, если упражнение не только с readme

## Перемещение и переименование упражнений

При перенумерации или перемещении упражнений:

1. Используй `git mv` (а не `mv`) для переименования каталогов - так сохраняется история git
2. Обнови числовой префикс, чтобы сохранить порядок
3. После перемещений снова запусти линтер

Пример:

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
```

## Пример: заготовки по плану

Для плана вида:

```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

Создай:

```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

Затем создай заготовки readme:

```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md -> "# Long-term Memory"
```
