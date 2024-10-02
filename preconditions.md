# Предусловия

Перед настройкой авторизации убедитесь, что выполнены следующие действия:

1. В личном кабинете VK ID для бизнеса [создано приложение](https://id.vk.com/business/go/docs/vkid/1.60.0/create-application)
1. Установлен и настроен VK ID SDK.

## Установка VK ID SDK

Для установки VK ID SDK воспользуйтесь одним из двух способов:

- NMP:

    ```bash
    npm install @vkontakte/superappkit
    ```

- YARN:

    ```bash
    yarn add @vkontakte/superappkit
    ```

Данные команды скачают библиотеку `superappkit`, которая позволяет интегрировать веб-приложение в экосистему ВКонтакте.

> Примечание:
>
> Для установки через менеджер пакетов требуется наличие Node.JS.

## Настройка VK ID SDK

Для настройки VK ID SDK выполните в терминале следующую команду:

```typescript
import { Connect } from '@vkontakte/superappkit';
```

Данная команда импортирует из библиотеки `superappkit` модуль `Connect`, необходимый для корректной работы VK ID.

После установки и настройки VK ID SDK [настройте авторизацию](authMethods.md).
