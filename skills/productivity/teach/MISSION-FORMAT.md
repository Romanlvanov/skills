# Формат MISSION.md

`MISSION.md` лежит в корне рабочего пространства. Он фиксирует _причину_, по которой пользователь изучает тему. Каждое учебное решение (чему учить дальше, какие ресурсы показать, какие упражнения придумать) должно прослеживаться до этого документа.

## Шаблон

```md
# Mission: {Topic}

## Why
{1-3 sentences. The concrete real-world goal the user is chasing. What changes in their life or work when they have this skill? Avoid abstract framings like "to understand X"; push for the underlying outcome.}

## Success looks like
- {A specific, observable thing the user will be able to do}
- {Another specific thing}
- {…}

## Constraints
- {Time, budget, prior commitments, learning preferences, anything that bounds the approach}

## Out of scope
- {Adjacent topics the user explicitly does not want to chase right now, protecting the zone of proximal development}
```

## Правила

- **Одна миссия на рабочее пространство.** Если пользователь хочет изучить две несвязанные вещи, заведи два рабочих пространства.
- **Конкретное важнее абстрактного.** «Пробежать полумарафон к октябрю» лучше, чем «прийти в форму». «Выпустить CLI на Rust для моей команды» лучше, чем «выучить Rust».
- **Не принимай расплывчатость.** Если пользователь не может сформулировать зачем, опроси его, прежде чем что-то писать. Плохая миссия хуже, чем никакой.
- **Пересматривай, когда меняется реальность.** Миссии меняются. Когда цель пользователя сдвигается, обнови этот файл: не оставляй устаревшую миссию направлять будущие сессии.
- **Держи коротким.** Если `MISSION.md` вылезает за экран, он перестал быть компасом и стал планом.
