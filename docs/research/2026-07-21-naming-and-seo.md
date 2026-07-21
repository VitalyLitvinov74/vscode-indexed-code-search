# Название и поисковое позиционирование

Дата исследования: 21 июля 2026 года.

## Цель

Выбрать название, которое одновременно:

- совпадает с реальным запросом пользователей;
- точно описывает расширение без собственного индекса и поискового движка;
- отличается от расширений, которые строят собственную выдачу;
- подходит для GitHub и будущей карточки Visual Studio Marketplace;
- не использует PhpStorm или JetBrains как часть бренда.

## Что ищут пользователи

Официальное название сценария в PhpStorm — `Search Everywhere`. Он открывается двойным `Shift`, объединяет классы, файлы, символы, действия и текст, а `Tab` переключает категории. Это делает фразу `Search Everywhere` главным высокоинтентным запросом для пользователей, переходящих с продуктов JetBrains.

В вопросах сообщества повторяются формулировки:

- «эквивалент `Search Everywhere` в VS Code»;
- «двойной `Shift` для глобального поиска»;
- «поиск файлов, символов, команд и текста из одного места»;
- «поиск как в PhpStorm или PyCharm».

Официальная терминология VS Code — `Quick Open`, `Quick Search`, поиск символов рабочей области и палитра команд. Эти названия полезны в описании и ключевых словах, но сами по себе хуже передают пользовательский сценарий.

## Конкурентная выдача

На момент проверки в Visual Studio Marketplace уже присутствуют прямые конкуренты:

| Расширение | Установки на 21.07.2026 | Отличие от нашего направления |
| --- | ---: | --- |
| [Search everywhere](https://marketplace.visualstudio.com/items?itemName=kbysiec.vscode-search-everywhere) | 29 979 | собственное сканирование и индекс файлов и символов |
| [JetBrains Search Everywhere](https://marketplace.visualstudio.com/items?itemName=extruct-ai.search-everywhere) | 1 691 | собственная объединённая выдача и индексирование |
| [Search everywhere (faster)](https://marketplace.visualstudio.com/items?itemName=qpwo.vscode-search-everywhere-faster) | 981 | собственное индексирование рабочей области |
| [Search Everywhere Lite](https://marketplace.visualstudio.com/items?itemName=Maorlevinshtein.search-everywhere-lite) | 167 | собственная панель и запасное сканирование файлов |
| [Shift Shift - Search Everywhere](https://marketplace.visualstudio.com/items?itemName=Legartis.legartis-search-everywhere) | 78 | собственный объединённый список файлов и символов |

Числа установок — изменяемый снимок публичного Gallery API Marketplace на дату исследования, а не постоянные значения.

В GitHub имя `vscode-search-everywhere` уже занято основным проектом и множеством его ответвлений. Поэтому общее имя без отличительного слова создаёт лишний шум.

## Сравнение названий

| Вариант | Совпадение с запросом | Точность границы продукта | Отличимость | Вывод |
| --- | ---: | ---: | ---: | --- |
| `Search Everywhere Native` | 5/5 | 5/5 | 4/5 | лучший баланс; ключевая фраза стоит первой |
| `Native Search Everywhere` | 4/5 | 5/5 | 4/5 | слово `Native` в начале засоряет выдачу результатами про React Native |
| `Unified Native Search` | 2/5 | 4/5 | 5/5 | уникально, но теряет главный пользовательский запрос |
| `Shift Search for VS Code` | 4/5 | 3/5 | 2/5 | слишком привязано к сочетанию и пересекается с `Shift Shift` |
| `Search Everywhere UX` | 5/5 | 3/5 | 4/5 | `UX` не объясняет, что поиск выполняет сам VS Code |

Точная фраза `Search Everywhere Native` не найдена среди отображаемых названий Marketplace на дату проверки. Окончательная доступность идентификатора расширения проверяется при заведении издателя и перед первой публикацией.

## Решение

- Название продукта и будущей карточки: `Search Everywhere Native`.
- Репозиторий: `vscode-search-everywhere-native`.
- Предварительный идентификатор расширения: `search-everywhere-native`.
- Основная формулировка: «`Search Everywhere` для VS Code на родном поиске редактора».
- Краткое обещание: «Файлы, символы, текст и команды через один привычный вызов — без собственного индекса».
- Сравнение с PhpStorm допустимо в описании и документации, но не в бренде.
- Слова `indexed`, `indexing` и обещания собственного поискового движка исключаются из позиционирования.

## Ключевые слова для Marketplace

При подготовке `package.json` использовать не более 30 точных ключевых слов. Начальный набор:

- `search everywhere`;
- `vscode search`;
- `quick open`;
- `quick search`;
- `workspace symbols`;
- `command palette`;
- `double shift`;
- `code navigation`;
- `file search`;
- `text search`;
- `native search`;
- `keyboard productivity`;
- `phpstorm alternative`;
- `pycharm alternative`;
- `webstorm alternative`.

Упоминания продуктов JetBrains в ключевых словах должны сопровождаться явным отказом от аффилированности в `README.md` и карточке Marketplace.

## Заготовка карточки Marketplace

**Отображаемое название:** `Search Everywhere Native`

**Краткое описание:** Пользовательский путь `Search Everywhere` для VS Code поверх родных `Quick Open`, `Quick Search`, символов и команд — без собственного индекса.

Категория: `Other`. Метка предварительного выпуска: `preview: true` до подтверждения сценариев первой версии.

## Проверка эффективности после публикации

Отслеживать отдельно:

- показы карточки по запросам `search everywhere`, `double shift` и `vscode search`;
- переходы из GitHub и Marketplace;
- установки и удаления в первые семь дней;
- долю пользователей, включивших двойной `Shift`;
- обращения о смешанной выдаче, чтобы не обещать невозможное на стабильном API;
- отзывы о том, понятна ли разница между родным поиском VS Code и собственным индексом.

## Источники

- [PhpStorm: Search everywhere](https://www.jetbrains.com/help/phpstorm/searching-everywhere.html) — официальная документация, актуальность 2026 год; подтверждает название сценария, двойной `Shift`, категории и текстовый поиск.
- [VS Code: Extension Manifest](https://code.visualstudio.com/api/references/extension-manifest) — официальная документация, проверено 21.07.2026; подтверждает роль `displayName`, `description`, `keywords` и ограничение до 30 ключевых слов.
- [VS Code: User interface](https://code.visualstudio.com/docs/editing/userinterface) — официальная документация, проверено 21.07.2026; подтверждает родные режимы `Quick Open`, символы и палитру команд.
- [VS Code: Basic editing](https://code.visualstudio.com/docs/editing/codebasics) — официальная документация, проверено 21.07.2026; подтверждает `Quick Search` и поиск текста по рабочей области.
- [Stack Overflow: JetBrains IDE double Shift equivalent in VS Code](https://stackoverflow.com/questions/76501923/jetbrains-ide-double-shift-to-search-everywhere-equivalent-in-vs-code) — сигнал сообщества; подтверждает формулировку пользовательского запроса.
- [Reddit: Switching from PyCharm — global search](https://www.reddit.com/r/vscode/comments/1igatrc/switching_from_pycharm_global_search_with/) — сигнал сообщества; подтверждает спрос на выделенный текст, двойной `Shift`, файлы, действия и символы.
- [John Jago: Double shift for search in VS Code](https://johnjago.com/shift-shift-vs-code/) — практический вторичный источник, 28.12.2024; подтверждает простой сценарий поверх `workbench.action.quickOpen`.
- [GitHub: kbysiec/vscode-search-everywhere](https://github.com/kbysiec/vscode-search-everywhere) — первичный репозиторий конкурента; подтверждает насыщенность общего имени.
