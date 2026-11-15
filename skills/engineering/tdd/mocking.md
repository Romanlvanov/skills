# Когда мокать

Мокай **только на границах системы**:

- Внешние API (платежи, почта и т.п.)
- Базы данных (иногда; лучше тестовая БД)
- Время/случайность
- Файловая система (иногда)

Не мокай:

- Свои собственные классы/модули
- Внутренних соавторов
- Всё, что ты контролируешь

## Проектирование под мокирование

На границах системы проектируй интерфейсы, которые легко мокать:

**1. Используй внедрение зависимостей**

Передавай внешние зависимости снаружи, а не создавай их внутри:

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. Предпочитай интерфейсы в стиле SDK универсальным fetcher-функциям**

Создавай отдельную функцию для каждой внешней операции вместо одной универсальной функции с условной логикой:

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

Подход SDK означает:
- Каждый мок возвращает одну конкретную форму
- Нет условной логики в подготовке теста
- Проще увидеть, какие эндпоинты использует тест
- Типобезопасность для каждого эндпоинта
