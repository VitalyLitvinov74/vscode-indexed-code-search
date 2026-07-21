# Осуществимость родного сценария Search Everywhere

Дата среза: 2026-07-21.

Проверены исходные коды VS Code 1.89.0 — минимальной версии с обычной командой `Quick Search` — и текущей стабильной версии 1.127.0.

## Главный вывод

Первую версию можно построить без собственного окна результатов, индекса и поискового процесса. Родной `Quick Access` уже умеет:

- открыть файлы, символы рабочей области, текст или команды с начальной строкой;
- перенести введённую строку при переходе от одного родного поставщика результатов к другому;
- оставить VS Code просмотр результата, открытие, открытие рядом, отмену и переход в полную панель поиска.

Ближайший к PhpStorm путь выглядит так:

1. Двойной `Shift` открывает родной `Quick Open`.
2. Выделение или слово под курсором становится начальным запросом.
3. `Tab` и `Shift+Tab` переключают категории «Файлы → Символы → Текст → Команды».
4. Сложный текстовый запрос передаётся в `workbench.action.findInFiles`.

Ключевое ограничение: перенос строки между родными категориями и контекстные ключи конкретных категорий существуют в исходном коде VS Code, но не объявлены стабильным API расширений. Поэтому переключение категорий нужно считать слоем совместимости, а не бессрочным контрактом.

## Что проверено

| Утверждение | Доказательство | Уверенность |
|---|---|---|
| Двойной `Shift` допустим как сочетание | В заметках VS Code 1.54 приведён пример `"key": "shift shift"`; текущая служба сочетаний отдельно обрабатывает сочетания из одиночных модификаторов | высокая |
| `workbench.action.quickOpen` принимает начальную строку | Реализация команды передаёт строковый аргумент в `quickAccess.show(..., { preserveValue: true })` | высокая, но это встроенная команда рабочего интерфейса |
| Одна команда может открыть четыре родных режима | Начальные строки: `query`, `#query`, `%query`, `>query` | высокая |
| Выделение можно прочитать без обхода рабочей области | Стабильный API даёт `window.activeTextEditor`, `TextEditor.selection`, `TextDocument.getText` и `getWordRangeAtPosition` | высокая |
| При смене родного поставщика строка сохраняется | `QuickAccessController` отрезает старый префикс и добавляет новый | высокая для 1.89.0 и 1.127.0; сам механизм внутренний |
| Категорию можно определить в условии сочетания | В 1.89.0 и 1.127.0 присутствуют `inFilesPicker`, `inWorkspaceSymbolsPicker`, `inTextSearchPicker`, `inCommandsPicker` | средняя: ключи внутренние |
| Расширение может прочитать строку активного родного окна | Стабильный API такого доступа не даёт | высокая |
| Собственный смешанный список текстовых совпадений возможен на стабильном API | Нет: стабильный API не отдаёт результаты родного текстового поиска расширению | высокая |

## Родные режимы и команды

| Категория | Начальное открытие с запросом | Команда для переключения уже открытого окна | Контекстный ключ |
|---|---|---|---|
| Файлы | `workbench.action.quickOpen(query)` | `workbench.action.quickOpen` без аргумента | `inFilesPicker` |
| Символы | `workbench.action.quickOpen("#" + query)` | `workbench.action.showAllSymbols` | `inWorkspaceSymbolsPicker` |
| Текст | `workbench.action.quickOpen("%" + query)` | `workbench.action.quickTextSearch` | `inTextSearchPicker` |
| Команды | `workbench.action.quickOpen(">" + query)` | `workbench.action.showCommands` | `inCommandsPicker` |

Для первоначального открытия текста нужен именно `workbench.action.quickOpen("%" + query)`. Команда `workbench.action.quickTextSearch` не принимает произвольный запрос: она заполняет строку только выделением из редактора и только при разрешающей настройке `editor.find.seedSearchStringFromSelection`.

Когда фокус уже находится в родном окне, `workbench.action.quickTextSearch` не берёт выделение редактора. Это полезно для переключения категории: внутренний `QuickAccessController` переносит текущую строку и заменяет только префикс.

## Как может работать переключение категорий

Расширение может внести контекстные сочетания:

- из файлов вызвать `workbench.action.showAllSymbols`;
- из символов вызвать `workbench.action.quickTextSearch`;
- из текста вызвать `workbench.action.showCommands`;
- из команд вызвать `workbench.action.quickOpen`;
- для обратного пути использовать те же команды в обратном порядке.

В исходном коде VS Code 1.89.0 и 1.127.0 при смене поставщика выполняется следующая логика:

1. берётся значение видимого родного окна;
2. отбрасывается префикс текущего поставщика;
3. добавляется префикс следующего;
4. создаётся новое родное окно с той же строкой фильтра.

Это позволяет не читать запрос через API и не создавать собственный `QuickPick`.

### Почему `Tab` нельзя считать полностью безопасным

В VS Code `Tab` используется для навигации по элементам интерфейса, а `Shift+Tab` имеет отдельное значение для пользователей программ экранного доступа. Кроме того, расширение не получает стабильного события закрытия встроенного `Quick Access`, поэтому трудно надёжно ограничить переопределение `Tab` только окнами, открытыми этим расширением.

Предварительное решение для проверки на пользователях:

- режим «как в JetBrains» с `Tab` и `Shift+Tab` включается явным выбором;
- сочетания не действуют при `accessibilityModeEnabled`;
- всегда доступны отдельные команды категорий и запасные сочетания;
- установка не изменяет `keybindings.json` и настройки пользователя;
- при изменении внутренних контекстных ключей расширение теряет только переключение категорий, а не поиск.

До пользовательского опыта не следует объявлять `Tab` окончательным сочетанием по умолчанию.

## Двойной Shift

VS Code поддерживает сочетания только из модификаторов с версии 1.54 и в официальных заметках приводит пример открытия `workbench.action.quickOpen` двойным `Shift`.

Для расширения нужны:

- команда «Открыть Search Everywhere Native»;
- вклад `"key": "shift shift"` в `contributes.keybindings`;
- обычная перенастройка через редактор сочетаний VS Code;
- инструкция по `Developer: Toggle Keyboard Shortcuts Troubleshooting` для диагностики конфликтов;
- запасной вызов из палитры команд.

Сочетание срабатывает после отпускания модификатора. На Windows, Linux и macOS его нужно проверять отдельно вместе с раскладками, залипанием клавиш и расширениями раскладок JetBrains.

## Начальный запрос

Предлагаемый порядок:

1. Если есть непустое однострочное выделение, использовать его.
2. Иначе, если пользователь включил заполнение словом, взять диапазон `getWordRangeAtPosition`.
3. Иначе открыть пустой родной режим.
4. Не сохранять запрос в телеметрии или собственной базе.

Нужно отдельно проверить:

- многострочное и очень длинное выделение;
- пробелы по краям;
- запрос, начинающийся с `#`, `%`, `>` или `?`;
- отсутствие активного текстового редактора;
- недоступный документ в удалённой среде.

Для файлов начальный запрос с зарезервированного префикса может переключить поставщика. Это не повод разбирать файлы или строить свой поиск; безопаснее не заполнять такой запрос автоматически и оставить строку пустой.

## Минимальная версия

- Двойной модификатор доступен с VS Code 1.54.
- Обычная команда `Quick Search` выпущена в VS Code 1.89.
- Перенос строки и четыре контекстных ключа подтверждены в исходном коде 1.89.0.
- Те же зависимости подтверждены в текущей стабильной версии 1.127.0.

Предварительное значение `engines.vscode`: `^1.89.0`. Окончательно закреплять его следует после запуска проверок на реальных сборках 1.89.x и 1.127.x, а не только после чтения исходного кода.

## Что говорят сценарии сообщества

Публичные обсуждения повторяют три потребности:

1. Пользователи JetBrains ищут именно двойной `Shift` и единый вход, а стандартные ответы обычно отправляют их в отдельный поиск символов или панель текста.
2. Пользователям нужен текстовый поиск по мере ввода, похожий на `Quick Open`; после появления `Quick Search` эту часть уже предоставляет сам VS Code.
3. Часто ожидается смешанный список имён файлов и содержимого. Старые расширения решали это собственным индексом, но для этого проекта такой путь намеренно исключён.

Из этого следует позиционирование: продукт объединяет родные категории и переносит запрос, но не обещает смешанную выдачу из всех источников в одном списке.

## Граница первой версии

Первая версия должна содержать только:

- одну команду входа;
- чтение выделения или слова из активного редактора;
- вызов родных команд с префиксом;
- команды следующей и предыдущей категории;
- необязательный режим `Tab`;
- переход в `workbench.action.findInFiles`;
- небольшой слой проверки наличия команд и совместимости версии.

Не нужны:

- собственный `QuickPick` с поисковыми результатами;
- чтение файлов рабочей области;
- собственный `ripgrep`;
- база или индекс;
- получение внутреннего объекта родного окна;
- доступ к DOM;
- изменение `search.quickOpen.includeSymbols` или `search.quickAccess.preserveInput`.

## Проверки перед реализацией

1. На VS Code 1.89.x и 1.127.x пройти цикл вперёд и назад с запросом `UserService`.
2. Повторить с пробелами, Unicode, путём, символом PHP и текстовой строкой.
3. Проверить `Enter`, просмотр, открытие рядом, `Escape` и кнопку перехода в полную панель.
4. Проверить конфликты двойного `Shift` через журнал сочетаний.
5. Сравнить `Tab` и запасное сочетание минимум на пяти участниках.
6. Повторить на Windows, macOS или Linux и минимум в одной среде WSL, SSH или контейнера.
7. При запуске проверять наличие четырёх встроенных команд через `commands.getCommands`.
8. При несовместимости отключать только цикл категорий и показывать отдельные родные команды.

## Источники

### Официальные материалы

- [PhpStorm: Search Everywhere](https://www.jetbrains.com/help/phpstorm/searching-everywhere.html)
- [VS Code 1.54: сочетания только из модификаторов](https://code.visualstudio.com/updates/v1_54)
- [VS Code 1.89: Quick Search](https://code.visualstudio.com/updates/v1_89)
- [VS Code: настройка сочетаний](https://code.visualstudio.com/docs/configure/keybindings)
- [VS Code: доступность и навигация клавишей Tab](https://code.visualstudio.com/docs/configure/accessibility/accessibility)
- [VS Code API](https://code.visualstudio.com/api/references/vscode-api)
- [VS Code: точки расширения](https://code.visualstudio.com/api/references/contribution-points)
- [VS Code 1.127](https://code.visualstudio.com/updates/v1_127)

### Исходный код VS Code

- [Перенос строки между поставщиками в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/platform/quickinput/browser/quickAccess.ts)
- [Перенос строки между поставщиками в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vs/platform/quickinput/browser/quickAccess.ts)
- [Команда Quick Open в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/workbench/browser/actions/quickAccessActions.ts)
- [Регистрация файлов, символов и текста в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/workbench/contrib/search/browser/search.contribution.ts)
- [Регистрация палитры команд в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/workbench/contrib/quickaccess/browser/quickAccess.contribution.ts)
- [Команда Quick Search в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/workbench/contrib/search/browser/searchActionsTextQuickAccess.ts)
- [Команда поиска символов в 1.89.0](https://github.com/microsoft/vscode/blob/1.89.0/src/vs/workbench/contrib/search/browser/searchActionsSymbol.ts)
- [Стабильные API активного редактора и документа в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vscode-dts/vscode.d.ts)
- [Контекстный ключ файлов в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vs/workbench/browser/quickaccess.ts)
- [Регистрация символов в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vs/workbench/contrib/search/browser/searchQuickAccess.contribution.ts)
- [Регистрация текста в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vs/workbench/contrib/search/browser/search.contribution.ts)
- [Регистрация палитры команд в 1.127.0](https://github.com/microsoft/vscode/blob/1.127.0/src/vs/workbench/contrib/quickaccess/browser/quickAccess.contribution.ts)

### Обсуждения

- [Stack Overflow: аналог двойного Shift из JetBrains](https://stackoverflow.com/questions/76501923/jetbrains-ide-double-shift-to-search-everywhere-equivalent-in-vs-code)
- [Stack Overflow: быстрый поиск текста во всех файлах](https://stackoverflow.com/questions/45472947/vs-code-quick-search-in-all-files-all-code-instantaneously)
- [Reddit: переход с PyCharm, выделенный текст и общий вход](https://www.reddit.com/r/vscode/comments/1igatrc/switching_from_pycharm_global_search_with/)
- [Reddit: единый поиск по именам и содержимому файлов](https://www.reddit.com/r/vscode/comments/y3syt2/is_there_a_quick_search_featureextension_that/)
