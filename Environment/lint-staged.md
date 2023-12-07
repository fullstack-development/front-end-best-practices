# Lint-staged

[Lint-staged](https://github.com/lint-staged/lint-staged) - это небольшая утилита, которая позволяют запускать набор команд для файлов, которые были проиндексированы Git (добавлены с помощью `git add`). Нам это полезно тем, что мы можем настроить Git hooks и перед каждым коммитом запускать линтеры только для проиндексированных файлов.

## Prerequisites

Прежде чем переходить к установке, убедитесь, что вы установили [husky](./husky.md).

## Установка

1. Установите `lint-staged` в качестве devDependencies используя ваш пакетный менеджер, например:

    `npm install --save-dev lint-staged`

2. Создайте в корне проекта файл `.lintstagedrc` со следующим содержимым:

    ```json
    {
      "*.{css}": "stylelint --fix",
      "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
      "*.{md,json}": "prettier --write"
    }
    ```
    Файл конфигурации может зависеть от особенностей вашего проекта. Подробнее [здесь](https://github.com/lint-staged/lint-staged#configuration).

3. Добавьте скрипт в ваш `package.json`:

    ```json
    {
      "scripts": {
        "lint-staged": "lint-staged"
      }
    }
    ```
4. Создайте файл `.husky/pre-commit` со следующим содержимым: 

    ```shell
    #!/usr/bin/env sh
    . "$(dirname -- "$0")/_/husky.sh"

    npm run lint-staged 
    # pnpm run lint-staged <- if you're using pnpm
    # yarn lint-staged <- if you're using yarn
    ```

## Использование

Просто делайте коммиты как обычно. Вышеупомянутый конфиг будет запускать скрипты линтеров с автоматическим исправлением для индексированных файлов. Если какой-то скрипт завершится с ошибкой, создание коммита будет отменено. 