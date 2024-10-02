# Настройка авторизации через всплывающее окно One Tap

Для настройки авторизации через всплывающее окно One Tap используется метод `Connect.floatingOneTapAuth`. Метод отображает окно быстрой авторизации в правом верхнем углу экрана с использованием свойства `position: fixed`.

Также можно использовать метод `Connect.oneTapAuth` с параметрами `AuthButtonType = floating` и `AuthButtonParams = FloatingOneTapAuthParams`.

## Сигнатура метода

```typescript
Connect.floatingOneTapAuth(params: FloatingOneTapAuthParams): VKOneTapAuthResult | null;

Connect.oneTapAuth(oneTapAuthType: AuthButtonType, params: AuthButtonParams): VKOneTapAuthResult | null;
```

## Описание параметров

Значения параметров метода `Connect.floatingOneTapAuth` представлены в виде объекта `FloatingOneTapAuthParams`.

Объявление типа `FloatingOneTapAuthParams`:

```typescript
type FloatingOneTapAuthParams = {
  callback: (result: VKAuthButtonCallbackResult) => void;
};
```

Здесь функция `callback` реализована типом `VKAuthButtonCallbackResult`, которых хранит в себе результат авторизации — ошибочный или успешный.

Объявление типа `VKAuthButtonCallbackResult`:

```typescript
type VKAuthButtonCallbackResult = VKAuthButtonResult | VKAuthButtonError;
```

Возможные значения:

- `VKAuthButtonResult` — объект, хранящий в себе результат успешной авторизации;
- `VKAuthButtonError` — объект, хранящий в себе результат авторизации, завершенной с ошибкой.

Объявление типа `VKAuthButtonResult`:

```typescript
type VKAuthButtonResult = {
  type: OneTapAuthEventsSDK | FloatingOneTapAuthEventsSDK |
  ButtonOneTapAuthEventsSDK | DataPolicyEventsSDK;
  provider?: 'vk';
  payload?: VKSilentAuthPayload | VKDataPolicyPayload | { uuid: string };
};
```

Здесь:

- `type` — события совершенной пользователем авторизации определенного типа (при использовании метода `Connect.oneTapAuth`, `Connect.floatingOneTapAuth`, `Connect.buttonOneTapAuth` или `Connect.userDataPolicy`);
- `provider` — название провайдера, через которого была совершена авторизация;
- `payload` — объект с дополнительными данными, которые могут быть использованы после авторизации.

Объявление типа `VKSilentAuthPayload`:

```typescript
type VKSilentAuthPayload = {
  auth: number;
  token: string;
  ttl: number;
  type: string;
  user: {
    id: number;
    first_name: string;
    last_name: string;
    avatar: string;
    phone: string;
};
  uuid: string;
  oauthProvider?: string;
  external_user?: ExternalUser;
};
```

Здесь:

- `auth` — признак активной сессии с ВКонтакте на устройстве пользователя. Возможные значения:

    `0` — сессии нет;
    `1` — сессия есть.

- `token` — токен для совершения действий от лица пользователя (Silent token, который необходимо обменять на Access token).
- `ttl` — срок жизни токена в секундах.
- `type` — тип токена.
- `user` — данные пользователя: ID, фамилия, имя, аватар и номер телефона.
- `uuid` — уникальный идентификатор, который выдается при обращении к VK ID SDK.
- `oauthProvider` — текстовый код oauth сервиса, через который авторизовался.
- `external_user` — объект пользователя, если реализована схема отображения сервисных данных вместо данных VK ID.

## Особенности работы метода

Метод содержит в себе функцию `callback`. Использование данной функции является обязательным для данного типа авторизации, так как она реализует функцию обработчика событий, приходящих из SDK (см. тип `ConnectEvents`).

## Возвращаемое значение

Метод возвращает объект типа `VKOneTapAuthResult` или `null`.

Объявление типа `VKOneTapAuthResult`:

```typescript
type VKOneTapAuthButtonResult = {
  destroy: () => void;
  getFrame: () => HTMLIFrameElement | null;
  authReadyPromise: Promise<OneTapAuthEventsSDK>;
};
```

Здесь:

- `destroy` — функция удаления всплывающего окна One Tap с веб-страницы;
- `getFrame` — функция получения фрейма всплывающего окна One Tap;
- `authReadyPromise` — объект типа `Promise`, обрабатывающий сообщения для всплывающего окна, приходящие из `OneTapAuthEventsSDK`.

## Пример реализации метода

Ниже представлен пример встраивания One Tap авторизации в виде всплывающего окна:

```typescript
var floatingOneTap = Connect.floatingOneTapAuth({  // Инициализация всплывающего окна One Tap
  callback: function(evt: any) {
  // Обработка событий, которые может вернуть всплывающее окно
  switch (type) {
    case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
      if (evt.payload.auth) {
        return onAuthUser(evt);
      } else {
        console.error(evt);
      }
      break;
    case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
      return Connect.redirectAuth({ screen: 'phone', url: 'https://...'})
    }
  },
});

// Добавление всплывающего окна в разметку веб-страницы (фактическое отображение всплывающего окна)
if (floatingOneTap) {
  document.body.appendChild(floatingOneTap.getFrame());
}

// Скрыть вслывающее окно можно с помощью метода VKOneTapAuthResult.destroy
floatingOneTap.destroy();
```

## События

В случае появления события `LOGIN_SUCCESS` VK ID SDK передаст ответ веб-приложению в формате `VKSilentAuthPayload`, а также [Silent token](https://id.vk.com/business/go/docs/vkid/1.60.0/tokens/silent-token) (параметр `token`).

В случае появления событий `PHONE_VALIDATION_NEEDED`пользователю нужно открыть полноценный VK ID, чтобы валидировать телефон.

Объявление типа `VKSilentAuthPayload`:

```typescript
type VKSilentAuthPayload = {
  auth: number;
  token: string;
  ttl: number;
  type: string;
  user: {
    id: number;
    first_name: string;
    last_name: string;
    avatar: string;
    phone: string;
  };
  uuid: string;
  oauthProvider?: string;
  external_user?: ExternalUser;
};
```

Здесь:

- `auth` — признак активной сессии с ВКонтакте на устройстве пользователя.

    Возможные значения:

    `0` — сессии нет;
    `1` — сессия есть.

- `token` — токен для совершения действий от лица пользователя (Silent token, который необходимо обменять на Access token).
- `ttl` — срок жизни токена в секундах.
- `type` — тип токена.
- `user` — данные пользователя: ID, фамилия, имя, аватар и номер телефона.
- `uuid` — уникальный идентификатор, который выдается при обращении к VK ID SDK.
- `oauthProvider` — текстовый код oauth сервиса, через который авторизовался.
- `external_user` — объект пользователя, если реализована схема отображения сервисных данных вместо данных VK ID.

После получения результата авторизации веб-приложение обменяет полученный Silent token (параметр `token`) на [Access token](https://id.vk.com/business/go/docs/vkid/1.60.0/tokens/access-token).

> Примечание:
>
> Обмен Silent token на Access token срабатывает также при использовании метода и `Connect.oneTapAuth`.
