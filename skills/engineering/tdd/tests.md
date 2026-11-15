# Хорошие и плохие тесты

## Хорошие тесты

**В интеграционном стиле**: тестируй через реальные интерфейсы, а не через моки внутренних частей.

```typescript
// GOOD: Tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Характеристики:

- Тестирует поведение, важное пользователям/вызывающим
- Использует только публичный API
- Переживает внутренний рефакторинг
- Описывает ЧТО, а не КАК
- Одна логическая проверка на тест

## Плохие тесты

**Тесты деталей реализации**: привязаны к внутренней структуре.

```typescript
// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

Тревожные признаки:

- Мокирование внутренних соавторов
- Тестирование приватных методов
- Проверки числа/порядка вызовов
- Тест ломается при рефакторинге без изменения поведения
- Имя теста описывает КАК, а не ЧТО
- Проверка внешними средствами вместо интерфейса

```typescript
// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**Тавтологические тесты**: ожидаемое значение повторяет реализацию, поэтому тест проходит по построению.

```typescript
// BAD: Expected value is recomputed the way the code computes it
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// GOOD: Expected value is an independent, known literal
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```
