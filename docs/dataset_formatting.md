# Формат и документация датасетов MERA

Содержание страницы:
- [Формат датасета](#формат-датасета)
    - [Структура файлов](#структура-файлов)
    - [Структура вопросов](#структура-вопросов)
- [Документация датасета](#документация-датасета)
    - [Текстовые описания](#текстовые-описания-raw_readme_rumdи-raw_readme_enmd)
    - [Метаданные датасета](#метаданные-датасета-raw_dataset_metajson)
    - [Правила оформления промптов](#как-оформлять-промпты)
- [Автосборка документации](#автосборка-документации)
- [➡️ Следующий шаг: загрузка датасета на 🤗HF](#следующий-шаг)


Разработанный датасет нужно оформить по описанной ниже структуре. Можно идти по описанию сверху вниз как по инструкции, пользуясь кросс-ссылками на разделы, поясняющие детали каждого этапа.


## Формат датасета

### Структура файлов

Датасет хранится в виде папки со структурированными файлами.

❗️ В репозиторий выкладываются только некоторые файлы, в структуре ниже они отмечены восклицательным знаком. Это файлы, документирующие датасет и его параметры. Остальные файлы, хранящие сам датасет, будут загружены только на Hugging Face Hub (это [следующий этап](#следующий-шаг)).

❗️ Часть файлов автособираемые, то есть их не нужно создавать вручную, они создаются скриптом на основе файлов с префиксом `raw_` (которые создаются вручную). В структуре ниже автособираемые файлы помечены символом доллара.

Структура папки:

```
dataset_name (название датасета)/
├── !$README.md$!  # автособираемая документация (англ)
├── !$README_ru.md$!  # автособираемая документация (рус)
├── !$dataset_meta.json$!  # автособираемая метаинформация
├── !raw_readme_en.json!  # документация вручную (англ)
├── !raw_readme_ru.json!  # документация вручную (рус)
├── !raw_dataset_meta.json!  # метаинформация вручную
├── test.json  # основные вопросы теста (не класть в репо, только на HF)
├── shots.json  # вопросы для few-shot примеров (не класть в репо, только на HF)
└── samples  # файлы с мультимодальными элементами для вопросов (не класть в репо, только на HF)
    └── image0001.png
    └── image0002.png
    └── audio0001.wav
    └── audio0002.wav
    ........................................
    └── image1234.png
    └── audio1234.wav
    ........................................
    └── image9999.png
    └── audio9999.wav
```

>### Ниже приведены подробные описания формата файлов, для примера можно ориентироваться на [готовые датасеты](../datasets).


Все вопросы датасета хранятся в двух файлах:
- `test.json` — основные вопросы теста
- `shots.json` — вопросы-примеры для few-shot режима (❗️не должны повторять ни вопросы, ни медиа из основного теста❗️)

Структура файлов:

```json
test.json
{
    "access": "", //"private" у приватного датасета, "public" у публичного
    "data": [
        // вопросы датасета, их структура описана ниже (см. Структура вопросов)
    ]
}
```

```json
shots.json
{
    "data": [
        // вопросы датасета, их структура описана ниже (см. Структура вопросов)
    ]
}
```

### Структура вопросов

В этом разделе рассмотрена структура вопросов, хранящихся в поле `data` файлов `test.json` и `shots.json`. Описание каждого поля приведено ниже.

```json
{
    "instruction": int, // int в исходном файле датасета, str при загрузке на HF Hub
    "inputs": {
        "image": str, // путь до файла в `samples/`
        "question": str,
        "option_a": str,
        "option_b": str,
        "option_c": str,
        "option_d": str
    },
    "outputs": str,
    "meta": {
        "id": int,
        "categories": dict,
        "image": {
            "synt_source": List[str],
            "source": str,
            "type": str,
            "content": str,
            "context": str
        }
    }
}
```


#### `instruction` — промпт-инструкция

Индекс одного из заданных для датасета промптов с плейсхолдерами. Промпты хранятся в метаданных датасета в поле `prompts`. Подробно о подготовке промптов написано [здесь](#как-оформлять-промпты).

На следующем этапе, когда датасет будет выгружаться в HF Hub, по этому индексу промпт будет вставлен автоматически из списка в метаданных датасета в поле `prompts`. Таким образом, если в дальнейшем потребуется внести изменения в формулировки промптов, менять их нужно только в метаданных, а затем заново выгружать датасет.

❗️Промпты должны быть равномерно распределены по вопросам датасета.


#### `inputs` — входные данные для вопроса

Словарь с полями, набор которых должен соответствовать набору плейсхолдеров в соответствующем промпте.


##### `image` — мультимодальный элемент

В случае использования аудио, поле будет называться `audio`, видео — `video`. Если в вопросе несколько мультимодальных элементов одного типа, то они нумеруются через "_i" с единицы, например "image_1", "image_2".


##### `question` — текстовый вопрос


##### `option_*` — варианты ответа

Варианты ответа в вопросе не обязательны, используются только если это предполагает формат задачи.

Нумеруются строчными буквами латинского алфавита ("option_a", "option_b", ...). Допускается разное количество вариантов ответа в вопросах датасета, при условии что это количество сбалансировано по датасету в целом — например, половина вопросов бинарные и половина с выбором из 4 вариантов.


#### `meta` — метаинформация об элементах вопроса

В это поле включается вся сопровождающая информация к вопросу и используемым в нем мультимодальным элементам. Эта информация ни в коем случае не используется в самом вопросе и промпте, а нужна для анализа статистик датасета и детальной оценки ответов моделей. Если какое-то поле может давать потенциальный ключ к ответу (например, в метаинформации есть поле с hints для CoT), это нужно указать в описании поля в метаинформации.


##### `id` — номер вопроса

Вопросы нумеруются с 1, нумерация сквозная для всех вопросов, **включая вопросы для shots**.


##### `image` — метаинформация об изображении

Словарь с разметкой, см. [типологию метаинформации об изображении](./image_file_meta.md). Такая разметка обязательна только для приватных задач, для публичных — на усмотрение разработчиков датасета.

В словарь также добавляется поле `synt_source` со списком оригинальных названий моделей, использованных для генерации соответствующей картинки.

Если какое-то поле внутри `image` неприменимо ни для одного вопроса в датасете, использовать это поле не нужно. Если не использовано ни одно поле, `image` остается пустым словарем (`{}`).


##### `categories` — классифицирующие поля

Словарь, в который входят все размеченные домены, поддомены, типы, классы вопросов и т. д. Отдельное поле на каждую категорию, со значением для конкретного сэмпла.


##### Другие поля

Внутри  `meta` можно добавлять любые другие поля.


## Документация датасета

Написание документации для датасетов MERA автоматизировано, так что автор датасета заполняет только информацию, которая уникальна для этого датасета, а все универсальные описания автоматически подтягиваются из шаблона.

Документация состоит из двух частей:

* **`raw_readme_ru.json`** и **`raw_readme_en.json`** – текстовые описания (описание задачи и методология создания датасета) на русском и английском в отдельных файлах;
* **`raw_dataset_meta.json`** – JSON с формальными характеристиками датасета.

Файлы должны соответствовать требованиям к формату json, таким как отсутствие trailing commas и trailing whitespaces. Отступ (indent) – 4 символа, ensure\_ascii=False (файл должен быть человекочитаемым).

>### Ниже приведены подробные описания формата файлов, для примера можно ориентироваться на [готовые датасеты](../datasets).


### Текстовые описания (`raw_readme_ru.md`и `raw_readme_en.md`)

Эти файлы содержат подробную документацию задачи (на русском и на английском соответственно): постановку, мотивацию создания датасета, детальную методологию создания датасета.

Каждый из двух файлов должен содержать словарь с заголовками и текстами следующих разделов документации:

| Поле ru | Поле en | Описание поля |
| :---- | :---- | :---- |
| Описание задачи | Task description | Описание датасета с кратким саммари по мотивации и методологии создания. Самая основная информация, сопроводительный текст к датасету. |
| Мотивация | Motivation | Этот раздел должен разносторонне отвечать на вопрос "зачем эта задача?". Вопросы для ориентира: \- На оценку каких моделей ориентирован этот датасет? Для каких моделей он НЕ подходит (limitations)? \- На каких пользователей ориентированы результаты оценки на данной задаче? Как эти пользователи будут интерпретировать результаты, полученные при оценке на этом датасете? \- Какие способности моделей оценивает задача? Что подразумевается под этими способностями? Не просто language understanding, а что именно и в какой постановке. Зачем оценивать именно эти способности? \- Как устроены вопросы в датасете, и почему они устроены именно так? Как понять, что именно такой дизайн задачи оценивает те способности моделей, которые нужно оценить? Валидность эксперимента. \- Как выбраны метрики (особенно когда метрика не просто доля правильных ответов) и почему они именно такие? Как такой выбор метрик помогает пользователю интерпретировать результаты? |
| Создание датасета | Dataset creation | Подробное описание методологии сбора датасета, источников данных, отбора данных, процесса валидации вопросов. |
| Human baseline | Human baseline | Подробное описание методологии проведения human baseline ([инструкция](human_baseline.md)). |
| Авторы | Contributors | Список строк-имен разработчиков датасета в формате "Имя Фамилия". |

Формат файла:  
```json
raw_readme_ru.json
{
    "Заголовок раздела 1": "текст раздела\nсо\nвсеми\nпереносами",
    "Заголовок раздела 2": "текст с\n\n### вложенными\nзаголовками"
}
```

**Добавление верхнеуровневых полей помимо перечисленных в таблице не предусмотрено**. Внутри разделов можно использовать любую структуру в формате MD с подзаголовками уровня "\#\#\# Заголовок" и ниже. Формат: ровно 1 пустая строка (`\n\n`) между заголовком и телом раздела, не более одной пустой строки внутри раздела.


### Метаданные датасета (`raw_dataset_meta.json`)

Этот файл содержит словарь с формальными признаками датасета, используемыми для автособираемой документации (карточки датасета). Словарь содержит рассчитываемые поля (как, например, `"len(DATA)"`), которые будут пересчитаны на основе файлов датасета.

❗️ Этот файл — "истина в последней инстанции" по датасету. В нем должны отражаться все вносимые изменения. Например, название датасета во всех случаях берется именно из файла с метаданными.


| Название поля | Описание поля |
| ---- | ---- |
| `dataset_name` | Название датасета. Пишется в CamelCase или CAPITALS, за исключением случаев адаптации других датасетов для русского языка – в этом случае называются в виде ruCamelCase / ruCAPITALS. |
| `description` | Короткое описание задачи (на английском языке), общая идея (максимум 200 символов). |
| `license` | Для публичного датасета: лицензия на материалы из оригинального датасета. Даже если из него переиспользована только часть (например, только картинки или только вопросы), лицензия все равно должна сохраняться. Формат: строка с md-форматированием, то есть если у лицензии помимо названия есть страница с описанием, пишем в виде \[license name\](url). Для приватного датасета: "MERA\_private". |
| `dataset_size` | Рассчитываемое поле. Оставить `"len(DATA)"`, будет рассчитано и вставлено при автосборке карточки. |
| `modalities` | Список модальностей из \["text", "image", "audio", "video"\]. Текстовая модальность включается всегда, если в случае конкретного датасета не оговорено иное. |
| `skills` | Список [навыков](skills_tax.md), на которые ориентирован тест. Список может быть скорректирован в процессе ревью датасета. |
| `domains` | Домены датасета в случае разделения на домены-папки, иначе пустой список. |
| `universal_domains` | Список универсальных доменов, которые покрывает датасет. Заполняется по согласованию с админами MERA. По умолчанию оставить пустым списком. |
| `synt_source_models` | Список моделей, которые были использованы при генерации текстовых и мультимодальных материалов для вопросов бенчмарка. Поле рассчитываемое, заполняется при автосборке карточки на основе вопросов датасета. Оставить `"list(set(sample["meta"]["domain"] for sample in DATA))"`. |
| `data_example` | Пример данных из оформленного по [инструкции](#структура-вопросов) датасета с точным соблюдением формата и набора полей. Пример не должен дублировать никакие вопросы из основного теста. Мультимодальные данные для примера нельзя брать из теста. |
| `data_field_descriptions` | Описания полей `data_example`. Тип данных поля в описании указывать не нужно, он автоматически берется из значений в `data_example`. Описание для каждого поля на русском и английском в формате `{"field_name": {"en": "Description.", "ru": "Описание."}}`.<br>Если подходящее датасету описание такого поля есть в term\_dictionary.json, то пишется `"en": "default"` и `"ru": "default"` соответственно, иное значение описания только если дефолтное не подходит. |
| `prompts` | Промпты с плейсхолдерами. Набор плейсхолдеров в каждом промпте должен соответствовать набору полей в `data_example["inputs"]`. В общем случае промптов должно быть не менее 10. В случае датасетов с доменами — не менее 10 промптов на каждый домен. [Как оформлять промпты?](#как-оформлять-промпты) |
| `metrics` | Словарь из названий и описаний метрик, используемых для оценки. Есть словарь с дефолтными описаниями метрик [`term_dictionary.json`](./templates/term_dictionary.json) – в случае, когда в этом словаре есть нужная метрика с подходящим описанием, описание дублировать не нужно, достаточно вместо него написать "default" (`{"metric_name": {"en": "default",  "ru": "default"}}`). Соответственно, иное значение описания нужно писать только в случае если дефолтное не подходит. |
| `human_benchmark` | Значения метрик [human baseline](human_baseline.md) (float). Название метрик обязательно из списка в поле `metrics`. Human baseline проводится только после первого ревь датасета, поэтому на первом этапе значения метрик можно указать как 0.0. |


## Как оформлять промпты

Формат:

Промпт задается в виде строки с плейсхолдерами. Набор плейсхолдеров должен в точности соответствовать набору поле в поле `inputs` вопросов датасета. Текстовые плейсхолдеры (то есть куда будут вставляться текстовые элементы) пишутся в фигурных скобках {} (`{question}`), картинки/аудио/видео – в угловых <> (`<image>`). Если мультимодальных элементов несколько, соответствующие плейсхолдеры нумеруются через "_i", нумерация в каждом вопросе с 1 внутри каждой модальности (`<image_1>`, `<image_2>`).

Пример валидного промпта:

```
"Внимательно посмотрите на картинку: <image>. Прочитайте вопрос и варианты ответа и выберите правильный ответ, указав только буквы правильного варианта без дополнительных пояснений. Вопрос:\n{question}\nA. {option_a}\nB. {option_b}\nC. {option_c}\nD. {option_d}\nОтвет:"
```

Требования к промптам:
- Промпты-инструкции должны быть достаточно **разнообразны** и **однозначны**. Разнообразие может достигаться разными способами: например, можно чередовать в промптах обращения на ВЫ и на ТЫ или менять порядок мест вопроса / картинки / описания формата.
- Инструкция должна точно **описывать формат вывода ответа**. Его в дальнейшем нам необходимо парсить, и инструктивная модель должна знать в каком виде писать ответ. Например, "напиши ответ только буквой без дополнительных пояснений". Не должно быть такого, что в коде парсятся ответы да/нет, но при этом в инструкции нет фразы: "отвечай только да или нет".
- В инструкция не должно быть опечаток, странных форм слов, отсутствия согласования и тд. Советуем проверять проверкой орфографии.
- Если промпт предполагает какое-то прямое продолжение, например, заканчивается на "Ответ:" или "4+2=", то убедитесь, что там нет пробелов, переносов строк и других **лишних символов**, которые могут помешать модели продолжать промпт.


## Автосборка документации

Для автосборки документации нужно подготовить 3 файла:
- [`raw_dataset_meta.json`](#метаданные-датасета-raw_dataset_metajson)
- [`raw_readme_en.json`](#текстовые-описания-raw_readme_rumdи-raw_readme_enmd)
- [`raw_readme_ru.json`](#текстовые-описания-raw_readme_rumdи-raw_readme_enmd)

Далее сложить их в папку `datasets/` в репо и запустить скрипт, который провалидирует формат полей и, если все корректно, соберет 3 чистовых файла:

- `dataset_meta.json`
- `README.md`
- `README_ru.md`

Запускать скрипт из корня репо, подставить название папки с датасетом вместо YOUR_DATASET:

```
python scripts/autocollect_docs.py "YOUR_DATASET"
```
В результате будут созданы следующие файлы:
- `datasets/YOUR_DATASET/dataset_meta.json`
- `datasets/YOUR_DATASET/README.md`
- `datasets/YOUR_DATASET/README_ru.md`

=> Проверьте, что автособранные ридми и метаинформация корректные, и загрузите их в папку датасета в репозиторий.


## Следующий шаг

Оформленный датасет нужно выложить на 🤗 Hugging Face Hub и отправить организаторам MERA, этот процесс описан в отдельной [инструкции](./dataset_hf.md).