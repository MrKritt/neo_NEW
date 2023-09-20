# ETL процесс ля интеграции данных из файлов в Неосинтез

Скрипт представляет собой ETL процесс для загрузки в неосинтез данных из эксель файлов.

Основные действия внутри скрипта реализованы как универсальные и не требуют изменения при добавлении/изменении новой конфигурации

*ELT - extract, transform, load (извлечение, преобразование, загрузка)

## Запуск

Запуск осуществляется через командную строку с обязательным указанием двух аргументов:
- режим
- суффикс файла конфигурации

Пример запуска режима appius с использованием файла конфигурации config_appius.json:

`python main.py appius appius`

Один и тот же режим может использовать разные конфигурации, например для интеграции данных на порталы ПИР СМР и ПЭ

## Режимы

Реализованы 4 режима:
- Заказы на доставку (delivery_order)
- Аппиус (appius)
- Уведомления (notification)
- Склады (storage)

Каждая конфигурация имеет два файла в папке config: эксель файл с сопоставлением колонок выгрузки и json файл с конфигурационными данными.

## Файлы конфигурации

Значение ключей файла конфигурации (файла config).
- url - полный адрес целевой системы с завершающим слэшем в конце (например https://construction.irkutskoil.ru/)
- logs_path - полный путь к папке, в которую складывать лог (например c://python/logs/)
- root_class_id - идентификатор класса папки-корня в неосинтез, внутри которой в рамках каждого титула будут размещаться интегрируемые сущности
- root_name - имя папки для создания новой или поиска существующей папки-корня
- group_class_id - идентификатор класса папки-группы в неосинтез. Не обязательный атрибут конфига. Папки такого класса будут созданы внутри папки-корня в случае настройки группировки
- group_by_column_name - имя колонки в файле-выгрузке, по значениям которой следует группировать интегрируемые сущности внутри папки-корня
- subobject_column_name - имя колонки в файле_выгрузки, содержащей имя подобъекта согласно справочнику НСИ
- item_class_id - идентификатор класса для самих интегрируемых сущностей в неосинтез
- object_attribute_id - идентификатор атрибута "Объект" в неосинтезе
- key_attribute_id - идентификатор атрибута класса сущности, содержащего уникальное значение для однозначной идентификации конкретной сущности
- key_column_name - имя колонки уникального идентификатора. Может быть как существующая колонка, так и созданная объединением нескольких колонок согласно атрибуту key_columns конфигурации
- key_columns - список наименований колонок в выгрузке по значениям, которых составляется уникальный идентификатор. Значения объединяются через "-" в порядке указания колонок. Пустые значения колонок игнорируются
- name_column_name - название колонки в выгрузке, значение в которой использовать как наименование сущности
- config_attribute_id - атрибут в сущности стройки, который содержит информацию для поиска нужно файла выгрузки (часть имени) или для фильтра значаний в выгрузке по некоторой колонке. Поведение зависит от конфигурации и реализуются явно в коде скрипта
- files_directory - директория с файлами выгрузок
- file_suffix - постоянная часть имени файла выгрузки определенного типа (например "_Д" для выгрузки заказов на доставку)
- attributes_file - относительный или полный путь к файлу эксель, содержащий правла сопоставления атрибутов неосинтез и колонок выгрузки (например configs/attributes_appius.xlsx)
- auth_data_file - файл содержащий строку запроса в api неосинтеза для получения токена. Содержит аутентификационные данные
- save_skipped - булево значение. Влияет на поведение скрипта в отношении сущностей по подобъектам, которые не заведены внутри стройки. Если true - интегрировать в общую папку. По умолчанию false.
- root_for_skipped_class_id - идентификатор класса неосинтеза для папки, в которую необходимо интегрировать сущности в случае save_skipped равном true. Сущность такого класса должна быть одна в рамках стройки
- one_root_mode - булево значение. По умолчанию false. Влияет на поведение скрипта в части распределения интегрируемых сущностей. По умолчанию раскладывается по титулам. Если true - складываются в общую папку с классом, указанным в root_for_skipped_class_id

## Файлы сопоставления атрибутов

Файлы служат для сопоставления колонок в файле выгрузке с конкретными атрибутами внутри неосинтеза, а так же определяет
какие колонки подлежат записи в атрибуты, так как не все значения из всех колонок извлекаются. Подход позовляет гибко управлять
какие именно данные подлежат интеграции в какие атрибуты.

Колонки в файле:
- id - идентификатор атрибута в неосинтез 
- type - тип атрибута в неосинтез (основные типы: 1 - число десятичное, 2 - строка, 3 - дата, 5 - дата и время, 6 - текст, 8 - ссылка)
- name - имя колонки в выгрузке
- folder - заполняется только для атрибутов типа 8 (справочный). Идентификатор папки-узла в неосинтезе, в котором содержатся допустимые сущности для данного справочного атрибута
- class - заполняется только для атрибутов типа 8 (справочный). Идентификатор класса справочных сущностей 
- regexp - заполняется, если необходимо извлечь из значения колонки выгрузки другое значени с использованием регулярного выражения
- regexp_name - обязательно, если указано регулярное выражение. Название, использующееся как ключ словаря (dict) при чтении данных из файла выгрузки

Имена колонок могут использоваться несколько раз. Это позволяет одно и то же значение использовать в разных атрибутах, например в одном случае значени как есть, а во втором с применением регулярного выражения