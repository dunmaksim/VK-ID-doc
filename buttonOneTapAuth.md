# Настройка авторизации через кнопку One Tap

Для настройки авторизации через кнопку One Tap используется метод `Connect.buttonOneTapAuth`. Кнопку One Tap можно разместить в любом месте веб-страницы.

Также можно использовать метод `Connect.oneTapAuth` с параметрами `AuthButtonType = button` и `AuthButtonParams = ButtonOneTapAuthParams`.

## Сигнатура метода

```typescript
Connect.buttonOneTapAuth(params: ButtonOneTapAuthParams): VKOneTapAuthButtonResult | null;

Connect.oneTapAuth(oneTapAuthType: AuthButtonType, params: AuthButtonParams): VKOneTapAuthResult | null;
```

## Описание параметров

Объект с параметрами метода `Connect.buttonOneTapAuth` должен реализовывать интерфейс `ButtonOneTapAuthParams`.

Объявление интерфейса `ButtonOneTapAuthParams`:

```typescript
interface ButtonOneTapAuthParams extends FloatingOneTapAuthParams {
  container?: HTMLElement | null;
  options?: ButtonOneTapAuthOptions;
}
```

Здесь:

- `container` — HTML-элемент веб-страницы приложения, в который будет добавлено окно с кнопкой;
- `options` — объект, который управляет отображением элементов внутри окна с кнопкой.

Параметр `options` имеет тип `ButtonOneTapAuthOptions`.

Объявление типа `ButtonOneTapAuthOptions`:

```typescript
type ButtonOneTapAuthOptions = {
  showAgreements?: boolean | number;
  showAlternativeLogin?: boolean | number;
  showAgreementsDialog?: boolean | number;
  displayMode?: ButtonOneTapAuthDisplayModes;
  buttonSkin?: ButtonOneTapSkin;
  langId?: number;
};
```

Здесь:

- `showAgreements` — отображение ссылки на политику конфиденциальности и передаваемые данные под кнопкой авторизации (см. [Настройка модального окна с политикой конфиденциальности и передаваемыми данными](userDataPolicy.md)):

   Возможные значения:

    - `true` — включено;
    - `false` — выключено.

- `showAlternativeLogin` — отображение кнопки «Войти другим способом».

- `showAgreementsDialog` — отображение диалогового окна со ссылкой на политику конфиденциальности и передаваемые данные.

    Это диалоговое окно отображается только при условии, что значение параметра `showAgreements` равно `false`.

    **Примечание**

    Параметр `showAgreementsDialog` считается устаревшим начиная с версии 1.56.0.

- `displayMode` — отображение данных пользователя.

    Возможные значения:

    - `default` — стандартное сообщение «Продолжить как ...»;
    - `name_phone` — имя и телефон;
    - `phone_name` — телефон и имя.

- `buttonSkin` — стиль отображения кнопки.

    Тип: `ButtonOneTapSkin`.

    Возможные значения:

    - `primary` — основной стиль;
    - `flat` — плоский стиль.

- `langId` — идентификатор языка.

    Возможные значения:

    | `langId` | Язык        |
    |:--------:|:-----------:|
    | 0        | Русский     |
    | 1        | Украинский  |
    | 3        | Английский  |
    | 4        | Испанский   |
    | 6        | Немецкий    |
    | 15       | Польский    |
    | 16       | Французский |
    | 82       | Турецкий    |

## Особенности работы метода

Интерфейс `ButtonOneTapAuthParams` дополняет тип `FloatingOneTapAuthParams`. Это необходимо для того, чтобы функция обратного вызова была доступна как для авторизации через кнопку One Tap, так и через всплывающее окно One Tap. Использование функции обратного вызова является обязательным для данных типов авторизации, так как она реализует функцию обработчика событий, приходящих из SDK (см. тип `ConnectEvents`).

Представление типа `FloatingOneTapAuthParams` и его описание доступны в разделе [Настройка авторизации через всплывающее окно One Tap, пункт «Описание параметров»](floatingOneTapAuth.md#описание-параметров).

## Возвращаемое значение

Метод возвращает объект типа `VKOneTapAuthButtonResult` или `null`.

Объявление типа `VKOneTapAuthButtonResult`:

```typescript
type VKOneTapAuthButtonResult = {
  destroy: () => void;
  getFrame: () => HTMLIFrameElement | null;
  authReadyPromise: Promise<OneTapAuthEventsSDK>;
};
```

Здесь:

- `destroy` — функция удаления кнопки One Tap с веб-страницы;
- `getFrame` — функция получения фрейма кнопки One Tap;
- `authReadyPromise` — объект типа `Promise`, обрабатывающий сообщения для кнопки, приходящие из `OneTapAuthEventsSDK`.

## Пример реализации метода

Ниже представлен пример встраивания One Tap авторизации в виде кнопки:

```typescript
var buttonOneTap = Connect.buttonOneTapAuth({  // Инициализация кнопки One Tap
  callback: function (evt) {
    const type = evt.type;
    if (!type) {
      return;
    }

    // Обработка событий, которые может вернуть кнопка
    switch (type) {
      case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
        return onAuthUser(evt);
      case ConnectEvents.OneTapAuthEventsSDK.FULL_AUTH_NEEDED:
      case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
      case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN:
        return Connect.redirectAuth({ url: 'https://...', state: 'dj29fnsadjsd82...'});
      case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN_OPTIONS:
        return Connect.redirectAuth({ screen: 'phone', source: type }).then(onAuthUser, () => alert('Ошибка!'));
    }
    return;
  },

  // Добавления параметров для конфигурации кнопки
  options: {
    showAlternativeLogin: true,
    showAgreements: false,
    showAgreementsDialog: true,
    displayMode: 'default',
  },
});

// Добавление кнопки в разметку веб-страницы (фактическое отображение кнопки)
if (buttonOneTap) {
  document.getElementById('someHtmlElementId').appendChild(buttonOneTap.getFrame());
}

// Убрать кнопку можно с помощью метода VKOneTapAuthButtonResult.destroy
buttonOneTap.destroy();
  ```

## События

В случае появления события `LOGIN_SUCCESS` VK ID SDK передаст ответ веб-приложению в формате `VKSilentAuthPayload`, а также [Silent token](https://id.vk.com/business/go/docs/vkid/1.60.0/tokens/silent-token) (параметр `token`).

В случае появления событий `PHONE_VALIDATION_NEEDED` и `FULL_AUTH_NEEDED` пользователю нужно открыть полноценный VK ID, чтобы завершить регистрацию (осуществить полную авторизацию), или валидировать телефон.

Другие события, которые могут прийти из VK ID SDK при использовании метода:

- `SHOW_LOGIN` — возникает при клике на кнопку входа для не авторизированного пользователя;
- `SHOW_LOGIN_OPTIONS` — возникает при клике на дополнительные возможности входа пользователя.

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

    - `0` — сессии нет;
    - `1` — сессия есть.

- `token` — токен для совершения действий от лица пользователя (Silent token, который необходимо обменять на Access token).
- `ttl` — срок жизни токена в секундах.
- `type` — тип токена.
- `user` — данные пользователя: ID, фамилия, имя, аватар и номер телефона;
- `uuid` — уникальный идентификатор, который выдается при обращении к VK ID SDK.
- `oauthProvider` — текстовый код oauth сервиса, через который авторизовался пользователь.
- `external_user` — объект пользователя, если реализована схема отображения сервисных данных вместо данных VK ID.

После получения результата авторизации веб-приложение обменяет полученный Silent token (параметр `token`) на [Access token](https://id.vk.com/business/go/docs/vkid/1.60.0/tokens/access-token).

> Примечание:
>
> Обмен Silent token на Access token срабатывает также при использовании метода `Connect.oneTapAuth`.
