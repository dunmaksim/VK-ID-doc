# Настройка авторизации через окно авторизации в браузере

Для настройки данного способа авторизации используется метод `Connect.redirectAuth`. Метод позволяет реализовать авторизацию с редиректом.

Алгоритм работы метода:

  1. VK ID открывает форму авторизации в текущей странице браузера.
  2. Пользователь производит авторизацию.
  3. В случае успешной авторизации, VK ID осуществит редирект на адрес, указанный в параметре `url`.

## Сигнатура метода

```typescript
Connect.redirectAuth({ params: RedirectAuthParams }): void;
```

## Описание параметров

В качестве параметров метода используется объект `RedirectAuthParams`:

  ```typescript
  type RedirectAuthParams = {
    url: string;
    state?: string;
    screen?: 'phone';
    action?: AuthAction;
    source?: string;
  };
  ````

Описание объекта:

  - `url` — адрес, на который будет произведён редирект после авторизации. **В адресе должен использоваться протокол https. При использовании http будет бесконечная загрузка**;
  - `state` — состояние веб-приложения, или любая произвольная строка, которая будет добавлена в query-string `url` после авторизации;
  - `screen` — при значении `'phone'` VK ID отобразит экран с вводом номера телефона;
  - `action` — объект для изменения поведения авторизации через VK ID;
  - `source` — параметр содержит данные о том, с какой страницы пользователь перешел на страницу авторизации.

Параметр `action` представлен в виде типа `AuthAction`:

  ```typescript
  type AuthAction = ExtendTokenParams | AuthWithUser;
  ```

Возможные значения:

  - `ExtendTokenParams` — объект события, который позволяет расширить partial-токен
  - `AuthWithUser` — событие, позволяющее авторизовать пользователя по SuperAppToken для сервисов, в которых доступна генерация таких токенов

Представление объекта `ExtendTokenParams`:

  ```typescript
  interface ExtendTokenParams { name: 'extend_token'; token: string;
  params: {
    extend_token_hash: string; };
  }
  ```

Описание объекта:

  - `name` — название события;
  - `token` — SuperAppToken v2;
  - `params` — хэш из ошибки `extend_token_hash`.

Представление события `AuthWithUser`:

  ```typescript
  interface AuthWithUser {
    name: 'login_with_user';
    token: string;
  }
  ```

Описание события:

  - `name` — название события;
  - `token` — SuperAppToken v2.

## Особенности работы метода

В query-string `url` передастся параметр payload с данными авторизации, формат данных в payload — `VKSilentAuthPayload`.

Если передать параметр `screen` со значением `phone`, то VK ID всегда будет показывать экран с вводом номера телефона, игнорируя активную сессию текущего пользователя.
