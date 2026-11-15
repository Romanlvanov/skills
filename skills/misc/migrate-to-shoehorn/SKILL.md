---
name: migrate-to-shoehorn
description: Переводит тестовые файлы с утверждений типа `as` на @total-typescript/shoehorn. Используй, когда пользователь упоминает shoehorn, хочет заменить `as` в тестах (replace `as` in tests) или ему нужны частичные тестовые данные (partial test data).
---

# Миграция на Shoehorn

## Зачем shoehorn?

`shoehorn` позволяет передавать в тестах частичные данные, не ломая проверку типов TypeScript. Он заменяет утверждения `as` типобезопасными альтернативами.

**Только тестовый код.** Никогда не используй shoehorn в продакшен-коде.

Проблемы с `as` в тестах:

- Нас приучили его не использовать
- Целевой тип приходится указывать вручную
- Двойной as (`as unknown as Type`) для намеренно неверных данных

## Установка

```bash
npm i @total-typescript/shoehorn
```

## Шаблоны миграции

### Большие объекты, из которых нужны немногие свойства

До:

```ts
type Request = {
  body: { id: string };
  headers: Record<string, string>;
  cookies: Record<string, string>;
  // ...20 more properties
};

it("gets user by id", () => {
  // Only care about body.id but must fake entire Request
  getUser({
    body: { id: "123" },
    headers: {},
    cookies: {},
    // ...fake all 20 properties
  });
});
```

После:

```ts
import { fromPartial } from "@total-typescript/shoehorn";

it("gets user by id", () => {
  getUser(
    fromPartial({
      body: { id: "123" },
    }),
  );
});
```

### `as Type` → `fromPartial()`

До:

```ts
getUser({ body: { id: "123" } } as Request);
```

После:

```ts
import { fromPartial } from "@total-typescript/shoehorn";

getUser(fromPartial({ body: { id: "123" } }));
```

### `as unknown as Type` → `fromAny()`

До:

```ts
getUser({ body: { id: 123 } } as unknown as Request); // wrong type on purpose
```

После:

```ts
import { fromAny } from "@total-typescript/shoehorn";

getUser(fromAny({ body: { id: 123 } }));
```

## Когда что использовать

| Функция         | Сценарий                                                        |
| --------------- | --------------------------------------------------------------- |
| `fromPartial()` | Передать частичные данные, которые всё ещё проходят проверку типов |
| `fromAny()`     | Передать намеренно неверные данные (автодополнение сохраняется)  |
| `fromExact()`   | Требовать полный объект (позже можно заменить на fromPartial)    |

## Порядок работы

1. **Собери требования** - спроси пользователя:
   - В каких тестовых файлах утверждения `as` создают проблемы?
   - Работает ли он с большими объектами, где важны лишь некоторые свойства?
   - Нужно ли передавать намеренно неверные данные для тестирования ошибок?

2. **Установи и мигрируй**:
   - [ ] Установи: `npm i @total-typescript/shoehorn`
   - [ ] Найди тестовые файлы с утверждениями `as`: `grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   - [ ] Замени `as Type` на `fromPartial()`
   - [ ] Замени `as unknown as Type` на `fromAny()`
   - [ ] Добавь импорты из `@total-typescript/shoehorn`
   - [ ] Запусти проверку типов, чтобы убедиться
