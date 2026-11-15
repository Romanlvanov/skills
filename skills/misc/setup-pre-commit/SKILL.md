---
name: setup-pre-commit
description: Настраивает в текущем репозитории pre-commit хуки Husky с lint-staged (Prettier), проверкой типов и тестами. Используй, когда пользователь хочет добавить pre-commit хуки (add pre-commit hooks), настроить Husky (set up Husky), сконфигурировать lint-staged (configure lint-staged) или добавить форматирование, проверку типов или тесты при коммите (commit-time formatting/typechecking/testing).
---

# Настройка pre-commit хуков

## Что настраивается

- pre-commit хук **Husky**
- **lint-staged**, запускающий Prettier на всех застейдженных файлах
- конфиг **Prettier** (если его нет)
- скрипты **typecheck** и **test** в pre-commit хуке

## Шаги

### 1. Определи менеджер пакетов

Проверь наличие `package-lock.json` (npm), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), `bun.lockb` (bun). Используй тот, что есть. Если неясно, по умолчанию npm.

### 2. Установи зависимости

Установи как devDependencies:

```
husky lint-staged prettier
```

### 3. Инициализируй Husky

```bash
npx husky init
```

Это создаёт каталог `.husky/` и добавляет `prepare: "husky"` в package.json.

### 4. Создай `.husky/pre-commit`

Запиши этот файл (для Husky v9+ shebang не нужен):

```
npx lint-staged
npm run typecheck
npm run test
```

**Адаптируй**: замени `npm` на обнаруженный менеджер пакетов. Если в package.json репозитория нет скрипта `typecheck` или `test`, убери эти строки и сообщи пользователю.

### 5. Создай `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### 6. Создай `.prettierrc` (если его нет)

Создавай, только если конфига Prettier ещё нет. Используй эти значения по умолчанию:

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

### 7. Проверь

- [ ] `.husky/pre-commit` существует и исполняемый
- [ ] `.lintstagedrc` существует
- [ ] скрипт `prepare` в package.json равен `"husky"`
- [ ] конфиг `prettier` существует
- [ ] Запусти `npx lint-staged`, чтобы убедиться, что всё работает

### 8. Закоммить

Добавь в индекс все изменённые и созданные файлы и закоммить с сообщением: `Add pre-commit hooks (husky + lint-staged + prettier)`

Коммит пройдёт через новые pre-commit хуки: хороший дымовой тест того, что всё работает.

## Примечания

- Husky v9+ не требует shebang в файлах хуков
- `prettier --ignore-unknown` пропускает файлы, которые Prettier не умеет разбирать (изображения и т.п.)
- pre-commit сначала запускает lint-staged (быстро, только застейдженные файлы), затем полную проверку типов и тесты
