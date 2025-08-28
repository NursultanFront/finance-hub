# Monorepo Guidelines

Этот репозиторий использует **Nx** для организации кода и управления зависимостями.  
Здесь собраны приложения (`apps/`) и библиотеки (`libs/`), сгруппированные по бизнес-доменам.

---

## 📂 Структура папок

```
apps/                # конечные приложения (Angular, React, Node)
  hub-shell/         # основное PWA приложение
  finance/           # микрофронтенд для финансов

libs/
  feature/           # бизнес-фичи (контейнерные компоненты, страницы)
  ui/                # UI-библиотеки (только презентационные компоненты)
  data-access/       # доступ к данным (API, state management)
  util/              # утилиты и хелперы
  shared/            # общее для нескольких приложений
```

- **Feature** — реализуют бизнес-кейсы/страницы, могут использовать все остальные типы.
- **UI** — содержат только презентационные компоненты, без логики работы с данными.
- **Data-access** — содержат код доступа к API + state management.
- **Util** — переиспользуемые функции, константы, хелперы.

---

## 🛡️ Правила зависимостей

Мы используем правило ESLint `@nx/enforce-module-boundaries` для строгой архитектуры.  
Каждый проект имеет теги (`tags`) в `project.json`:

- `"tags": ["type:feature"]`
- `"tags": ["type:ui"]`  
- `"tags": ["type:data-access"]`
- `"tags": ["type:util"]`

### Матрица зависимостей

| From → To      | Feature | UI  | Data | Util |
|----------------|---------|-----|------|------|
| **Feature**    | ✅      | ✅  | ✅   | ✅   |
| **UI**         | ❌      | ✅  | ❌   | ✅   |
| **Data-access**| ❌      | ❌  | ✅   | ✅   |
| **Util**       | ❌      | ❌  | ❌   | ✅   |

- ✅ — разрешено  
- ❌ — запрещено

Примеры:
- `ui/button` **не может** импортировать `data-access/auth`.
- `data-access/orders` **не может** использовать `ui/table`.
- `feature/profile` **может** импортировать всё.

---

## ⚙️ Генераторы Nx

### Создание новых проектов:

```bash
# Feature библиотеки (финансовые фичи)
nx g @nx/angular:library --name=feature-finance --directory=libs/feature --tags=type:feature,domain:finance
nx g @nx/angular:library --name=feature-dashboard --directory=libs/feature --tags=type:feature,domain:finance

# UI библиотеки (отдельные компоненты)
nx g @nx/angular:library --name=ui-button --directory=libs/ui --tags=type:ui
nx g @nx/angular:library --name=ui-card --directory=libs/ui --tags=type:ui
nx g @nx/angular:library --name=ui-input --directory=libs/ui --tags=type:ui
nx g @nx/angular:library --name=ui-modal --directory=libs/ui --tags=type:ui

# Data-access библиотеки (для финансов)
nx g @nx/angular:library --name=data-access-finance --directory=libs/data-access --tags=type:data-access,domain:finance

# Utility библиотеки (общие утилиты)
nx g @nx/angular:library --name=util-common --directory=libs/util --tags=type:util
nx g @nx/angular:library --name=util-currency --directory=libs/util --tags=type:util
```

### Создание приложений:

```bash
# Основное приложение
nx g @nx/angular:app --name=hub-shell --routing=true --style=scss --standalone=true

# Микрофронтенд
nx g @nx/angular:app --name=finance --routing=true --style=scss --standalone=true
```

### Создание компонентов:

```bash
# В конкретной библиотеке
nx g @nx/angular:component --name=button --project=ui-components --standalone=true

# В feature библиотеке
nx g @nx/angular:component --name=dashboard --project=feature-finance --standalone=true
```

### Управление проектами:

```bash
# Перемещение проекта
nx g @nx/workspace:move --project=old-name --destination=new-location/new-name

# Удаление проекта  
nx g @nx/workspace:remove --project=project-name
```

---

## 📊 Визуализация зависимостей

- Построить граф проектов:
```bash
nx graph
```

- Построить граф задач (например, для build):
```bash
nx build my-app --graph
```

- Показать затронутые проекты:
```bash
nx affected:graph
```

---

## 🚀 Команды для разработки

### Запуск приложений:

```bash
# Основное приложение
nx serve hub-shell

# Микрофронтенд
nx serve finance

# Все приложения параллельно
nx run-many --target=serve --projects=hub-shell,finance --parallel
```

### Сборка:

```bash
# Одно приложение
nx build hub-shell

# Все приложения
nx run-many --target=build --projects=hub-shell,finance

# Только затронутые изменениями
nx affected:build
```

### Тестирование:

```bash
# Все тесты
nx run-many --target=test --all

# Только затронутые
nx affected:test

# Конкретная библиотека
nx test ui-components
```

### Линтинг:

```bash
# Все проекты
nx run-many --target=lint --all

# Только затронутые
nx affected:lint

# Конкретный проект
nx lint feature-finance
```

---

## 📦 Настройка Module Federation

Для микрофронтендов:

```bash
# Установка зависимостей
npm install @nx/webpack --save-dev

# Настройка host приложения
nx g @nx/angular:setup-mf hub-shell --mfType=host --routing=true

# Настройка remote приложения
nx g @nx/angular:setup-mf finance --mfType=remote --host=hub-shell --routing=true
```

---

## 🔧 PWA настройка

```bash
# Добавление PWA к приложению
nx g @angular/pwa:pwa --project=hub-shell
```

---

## 🌐 Internationalization

```bash
# Установка Transloco
npm install @jsverse/transloco @jsverse/transloco-messageformat --save

# Добавление к проекту
nx g @jsverse/transloco:ng-add --project=hub-shell
```

---

## ✅ Резюме

- Код делим по бизнес-доменам и слоям (`feature`, `ui`, `data-access`, `util`).
- Используем **теги** и правило `enforce-module-boundaries`.
- Генераторы Nx помогают создавать, перемещать и удалять проекты.
- Граф зависимостей (`nx graph`) всегда показывает актуальную архитектуру.
- Module Federation для микрофронтендов.
- PWA и i18n поддержка из коробки.