# Формат GLOSSARY.md

## Структура

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
{A one or two sentence description of the term}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## Правила

- **Имей мнение.** Если для одного понятия есть несколько слов, выбери лучшее, а остальные перечисли под `_Avoid_`.
- **Держи определения плотными.** Не больше одного-двух предложений. Определяй, что это ЕСТЬ, а не что оно делает.
- **Включай только термины, специфичные для контекста этого проекта.** Общие понятия программирования (таймауты, типы ошибок, служебные паттерны) сюда не относятся, даже если проект активно их использует. Прежде чем добавить термин, спроси себя: это понятие, уникальное для этого контекста, или общее понятие программирования? Место есть только первому.
- **Группируй термины под подзаголовками**, когда естественно складываются кластеры. Если все термины относятся к одной цельной области, плоский список подойдёт.

## Репозитории с одним и несколькими контекстами

**Один контекст (большинство репозиториев):** один `GLOSSARY.md` в корне репозитория.

**Несколько контекстов:** `GLOSSARY-MAP.md` в корне репозитория перечисляет контексты, где они живут и как связаны друг с другом:

```md
# Glossary Map

## Contexts

- [Ordering](./src/ordering/GLOSSARY.md): receives and tracks customer orders
- [Billing](./src/billing/GLOSSARY.md): generates invoices and processes payments
- [Fulfillment](./src/fulfillment/GLOSSARY.md): manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

Скилл сам определяет, какая структура применяется:

- Если `GLOSSARY-MAP.md` существует, прочитай его, чтобы найти контексты
- Если есть только корневой `GLOSSARY.md`, контекст один
- Если нет ни того, ни другого, лениво создай корневой `GLOSSARY.md`, когда разрешится первый термин

Когда контекстов несколько, определи, к какому из них относится текущая тема. Если неясно, спроси.
