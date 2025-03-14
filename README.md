# @bitrix/cli
@bitrix/cli&nbsp;— консольный инструмент Битрикс-разработчика,
основная цель&nbsp;— упростить и&nbsp;автоматизировать разработку фронтенда для
проектов на&nbsp;«Битрикс Управление Сайтом» и&nbsp;«Битрикс24».

[![npm version](https://badge.fury.io/js/%40bitrix%2Fcli.svg)](https://badge.fury.io/js/%40bitrix%2Fcli)


## Содержание
1. [Описание](#introduction)
2. [Установка](#install)
3. [Конфигурация](#config)
4. [Сборка](#build)
5. [Запуск тестов](#test)
6. [Создание экстеншна](#create)

<h2 id="introduction">Описание</h2>

@bitrix/cli&nbsp;— это набор консольных команд
1. `bitrix build` для сборки и&nbsp;транспиляции ES6+ кода
2. `bitrix test` для запуска Mocha тестов
3. `bitrix create` для быстрого создания «экстеншна»

> В&nbsp;первую очередь, `@bitrix/cli` предназначен для работы с&nbsp;«экстеншнами»,
шаблонами сайта и&nbsp;шаблонами компонентов.


<h2 id="install">Установка</h2>

NPM
```bash
$ npm install -g @bitrix/cli
```

YARN
```bash
$ yarn global add @bitrix/cli
```

<h2 id="config">Конфигурация</h2>

### Базовая конфигурация
```javascript
module.exports = {
	input: './app.js', 
	output: './dist/app.bundle.js',
};
```

### Все параметры
```javascript
module.exports = {
	// Файл для которого необходимо выполнить сборку. 
	// Необходимо указать относительный путь 
	input: string, 
	
	// Путь к бандлу, который будет создан в результате сборки 
	// Обычно это ./dist/<extension_name>.bundle.js
	// Необходимо указать относительный путь 
	output: string || {js: string, css: string},

    // Алиасы для внешних импортов
    // https://rollupjs.org/configuration-options/#output-globals
    globals: { [id: string]: string }
 
	// Неймспейс, в который будут добавлены все экспорты из файла указанного в input
	// Например 'BX.Main.Filter'
	namespace: string,
	
	// Списки файлов для принудительного объединения. 
	// Файлы будут объединены без проверок на дублирование кода. 
	// sourcemap's объединяются автоматически 
	// Необходимо указать относительные пути
	concat: {
		js: Array<string>,
		css: Array<string>,
	},
	
	// Разрешает или запрещает сборщику модифицировать config.php
	// По умолчанию true (разрешено)
	adjustConfigPhp: boolean,
	
	// Разрешает или запрещает сборщику удалять неиспользуемый код. 
	// По умолчанию true (включено).
	treeshake: boolean,
	
	// Разрешает или запрещает пересобирать бандлы 
	// если сборка запущена не в корне текущего экстеншна 
	// По умолчанию `false` (разрешено)
	'protected': boolean,
	
	plugins: {
		// Переопределяет параметры Babel.
		// Можно указать собственные параметры Babel
		// https://babeljs.io/docs/en/options
		// Если указать false, то код будет собран без транспиляции
		babel: boolean | Object,
		
		// Дополнительные плагины Rollup, 
		// которые будут выполняться при сборке бандлов 
		custom: Array<string | Function>,
	},
    // Определяет правила обработки путей к изображениям в CSS
    // Доступно с версии 3.0.0
    cssImages: {
        // Определяет правило по которому изображения должны быть обработаны
        // 'inline' — преобразует изображения в инлайн 
        // 'copy' — копирует изображения в директорию 'output'
        // По умолчанию 'inline'.
        type: 'inline' | 'copy', 

        // Путь к директории в которую должны быть скопированы используемые изображения 
	    output: string,

        // Максимальный размер изображений в кб, которые могут быть преобразованы в инлайн
        // По умолчанию 14кб
        maxSize: number,

        // Использовать ли svgo для оптимизации svg 
        // По умолчанию true 
        svgo: boolean, 
    },
    resolveFilesImport: {
        // Путь к директории в которую должны быть скопированы импортированные изображения 
        output: string,
        
        // Определяет разрешенные для импорта типы файлов
        // По умолчанию ['**/*.svg', '**/*.png', '**/*.jpg', '**/*.gif']
        // https://github.com/isaacs/minimatch
        include: Array<string>,

        // По умолчанию []
        exclude: Array<string>,
    },
    
    // Определяет правила Browserslist 
    // false — не использовать (по умолчанию)
    // true — использовать файл .browserslist / .browserslistrc
    browserslist: boolean | string | Array<string>,
  
    // Включает или отключает минификацию 
    // По умолчанию отключено 
    // Может принимать объект настроек Terser
    // false — не минифицировать (по умолчанию)
    // true — минифицировать с настройками по умолчанию 
    // object — минифицировать с указанными настройками 
    minification: boolean | object,
  
    // Включает или отключает преобразование нативных JS классов 
    // По умолчанию значенение параметра выставляется автоматически на основании browserslist
    transformClasses: boolean,
  
    // Включает или отключает создание Source Maps файлов 
    sourceMaps: boolean,
    
    // Настройки тестов 
    tests: {
        // Настройки локализации 
        localization: {
            // Код языка локализации. По умолчаниию 'en'
            languageId: string,
            // Включает или выключает автозагрузку фраз в тестах. По умолчанию включено 
            autoLoad: boolean,
        },
    },
};
```

<h2 id="build">Сборка</h2>

Для запуска сборки выполните команду
```bash
$ bitrix build
```
> Сборщик рекурсивно найдет все файлы `bundle.config.js` и выполнит
для каждого конфига сборку и транспиляцию.

### Дополнительные параметры

#### --watch [\<fileExtension\>[, ...]], -w=[\<fileExtension\>[, ...]]
Режим отслеживания изменений. Пересобирает бандлы после изменения исходных файлов.
В качестве значения можно указать список расширений файлов, в которых нужно отслеживать изменения.
```bash
$ bitrix build --watch
```
Сокращенный вариант 
```bash
$ bitrix build -w
```
Вариант с отслеживанием изменений в указанных типах файлов 
```bash
$ bitrix build -w=defaults,json,mjs,svg
```
> `defaults` — набор расширений файлов которые отслеживаются по умолчанию.
Он равен `js,jsx,vue,css,scss`.

#### --test, -t
Режим непрерывного тестирования. Тесты запускаются после каждой сборки.
Обратите внимание, сборка с&nbsp;параметром `--test` выводит в&nbsp;отчете только статус прохождения
тестов&nbsp;— прошли или не&nbsp;прошли, полный отчет выводит только команда `bitrix test`.
```bash
$ bitrix build --test
```

#### --modules \<moduleName\>[, ...], -m=\<moduleName\>[, ...]
Сборка только указанных модулей. Параметр поддерживается только в&nbsp;корневой с&nbsp;модулями `local/js` и `bitrix/modules`.
В&nbsp;значении укажите имена модулей через запятую, например:
```bash
$ bitrix build --modules main,ui,landing
```

#### --path \<path\>, -p=\<path\>
Запуск сборки для указанной директории. В&nbsp;значении укажите относительный путь к&nbsp;директории,
например:
```bash
$ bitrix build --path ./main/install/js/main/loader
```
Сокращенный вариант
```bash
$ bitrix build -p=./main/install/js/main/loader
```

#### --extensions \<extensionName\>[, ...], -e=\<extensionName\>[, ...]
Запускает сборку указанных экстеншнов. В качестве значения нужно указать имя экстеншна, 
либо список имен через запятую. Команду можно запускать из любой директории проекта.
```bash
$ bitrix build -e=main.core,ui.buttons,landing.main
```


<h2 id="tests">Запуск тестов</h2>
```bash
$ bitrix test
```   
Команда запускает Mocha тесты и&nbsp;выводит подробный отчет о&nbsp;прохождении тестов.
> Тестами считаются&nbsp;JS файлы, расположенные в&nbsp;директории `./test`,
относительно файла `bundle.config.js`. В&nbsp;момент запуска тестов исходный код и&nbsp;код тестов,
налету обрабатывается сборщиком и&nbsp;после чего выполняется. Поэтому тесты можно писать на&nbsp;ES6+

### Дополнительные параметры

#### --watch [\<fileExtension\>[, ...]], -w=[\<fileExtension\>[, ...]]
Режим отслеживания изменений. Запускает тесты после изменения исходных файлов и&nbsp;кода тестов.
В качестве значения можно указать список расширений файлов, в которых нужно отслеживать изменения.
```bash
$ bitrix test --watch
```
Сокращенный вариант
```bash
$ bitrix test -w
```
Вариант с отслеживанием изменений в указанных типах файлов
```bash
$ bitrix test -w=defaults,json,mjs,svg
```
> `defaults` — набор расширений файлов которые отслеживаются по умолчанию.
Он равен `js,jsx,vue,css,scss`.

#### --modules \<moduleName\>[, ...], -m=\<moduleName\>[, ...]
Тестирование только указанных модулей. Параметр поддерживается только в
корневой директории репозитория. В&nbsp;значении укажите имена модулей через запятую,
например:
```bash
$ bitrix test --modules main,ui,landing
```

#### --path \<path\>, -p=\<path\>
Запуск тестов для указанной директории. В&nbsp;значении укажите относительный путь к&nbsp;директории,
например:
```bash
$ bitrix test --path ./main/install/js/main/loader
```
Сокращенный вариант 
```bash
$ bitrix test -p=./main/install/js/main/loader
```

#### --extensions \<extensionName\>[, ...], -e=\<extensionName\>[, ...]
Запускает тесты в указанных экстеншнах. В качестве значения нужно указать имя экстеншна,
либо список имен через запятую. Команду можно запускать из любой директории проекта.
```bash
$ bitrix test -e=main.core,ui.buttons,landing.main
```

<h2 id="create">Создание «экстеншна»</h2>

Для создания «экстеншна»
1. Перейдите в директорию `local/js/{module}`
2. Выполните команду `bitrix create`
3. Ответьте на вопросы мастера
