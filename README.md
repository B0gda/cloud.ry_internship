# Ответы на тестовое задание на стажировку AppSecCloudCamp

## 1. Вопросы
1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались?
    ##### Ответ:
    На пути к освоению безопасной разработки я сталкивался с различными задачами и вызовами. Мой первый опыт в этой области был приобретен во время участия в соревнованиях Tinkoff CTF. Там мне приходилось искать уязвимости, связанные с неправильным роутингом, такими как SQL-инъекции, ошибками в логике приложения и нежелательным раскрытием конфиденциальной информации, которая должна была оставаться скрытой от пользователей. Этот опыт позволил мне глубже понять принципы безопасности в разработке и развить навыки по обнаружению и устранению потенциальных угроз.
   
2. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было?
    ##### Ответ:
     Да, я проводил ручной анализ кода для проектов, которые были выложены на хостинг. В процессе этого анализа мне удалось выявить уязвимости, связанные с XSS (межсайтовый скриптинг), которые были успешно исправлены до выпуска приложения.
   Знаю о существовании  статического и динамического анализаторов кода, но не применял на практике.
   
3. Если у вас был опыт поиска уязвимостей, расскажите, как это было?
    ##### Ответ:
     При работе с уязвимостью XSS, первоначально я создал специально сконструированные строки данных, содержащие HTML или JavaScript код, который мог бы быть внедрен в приложение через входные поля. После внедрения тестовых данных я проверил, как приложение обрабатывает эти данные. Далее я добавил дополнительные проверки информации на стороне сервера, чтобы при поступлении невалидных данных не происходило изменение базы данных.
   
4. Почему вы хотите участвовать в стажировке?
   ##### Ответ:
   Я хочу участвовать в стажировке, чтобы получить больше опыта в области безопасной разработки.
    Эта стажировка позволит расширить мои практические навыки, даст возможность работать в команде, а следовательно знакомства с новыми людьми и обмен знаниями и бесценным опытом! Я не могу сидеть на месте, каждый день изучаю что-то новое, используя книжный и интернет материалы, решаю практические задачи и применяю их на реальном коде. Я чувствую, что именно здесь я смогу восполнить пробелы в своих знаниях.

    Теория без практики невозможна, поэтому с удовольствием готов применить свои знания в стажировке.

    Я знаю, что в вашей компании мне удастся применить свои знания, повысить свой профессиональный уровень и влиться в дружный коллектив.

---
## 2. Security code review
### Часть 1. Security code review: GO
Уязвимость находится в строке:
` query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery) `

В этой строке используется конкатенация строк для создания SQL-запроса.
Если пользователь введет в поле поиска строку, содержащую SQL-код, он сможет выполнить произвольный запрос к базе данных, т.е. возможен доступ к конфиденциальным данным.

Также в некоторых местах кода ошибки обрабатываются с помощью ` log.Fatal `, что приводит к завершению работы программы.
Вместо этого следует использовать более безопасные методы обработки ошибок, например, возвращать код ошибки клиенту.


Для исправления уязвимости, связанную с sql-запросом, необходимо использовать параметризованные запросы, т.к. они предоставляют встроенную защиту от SQL-инъекций, предотвращая возможность внедрения вредоносного кода в запросы.
Пример исправления кода:

` query := "SELECT * FROM products WHERE name LIKE ?" `

` rows, err := db.Query(query, "%"+searchQuery+"%") `

В этом случае SQL-запрос подготавливается заранее, а значения параметров передаются отдельно.
Это предотвращает возможность SQL-инъекции, так как пользовательский ввод не интерпретируется как часть SQL-кода.

Также для решения проблемы можно было бы использовать библиотеки для объектно-реляционного отображения, такие как GORM, которые предоставляют безопасный способ работы с базой данных. Но тогда бы размер проекта увеличился, а параметризованные запросы не влияют на "вес" проекта.

### Часть 2: Security code review: Python
#### Пример №2.1:
Присутсвует уязвимость типа ***SSTI (Server-Side Template Injection)*** в строке ` output = Template('Hello ' + name + '! Your age is ' + age + '.').render() ` В этой строке используется конкатенация строк для создания шаблона Jinja2.
Если пользователь введет в поле ` name ` или ` age ` строку, содержащую код Jinja2, он сможет выполнить произвольный код на сервере и получить доступ к конфиденциальным данным.

Для исправления необходимо использовать безопасные методы рендеринга шаблонов, такие как метод render. Пример исправленного кода:

` output = Template('Hello {{ name }}! Your age is {{ age }}.').render(name=name, age=age) `

В этом случае значения переменных передаются в шаблон Jinja2 с помощью безопасного метода ` render `.
Это предотвращает возможность SSTI, так как пользовательский ввод не интерпретируется как код Jinja2.

#### Пример №2.2
Присутствует уязвимость типа ***Командная инъекция*** в строке ` cmd = 'nslookup ' + hostname
output = subprocess.check_output(cmd, shell=True, text=True) ` В этой строке используется конкатенация строк для создания команды, которая будет выполнена с помощью ` subprocess.check_output `.
Если пользователь введет в поле  ` hostname ` строку, содержащую вредоносную команду, она будет выполнена на сервере, что позволит получить доступ к скрытым от пользователя данным сервера.

Для исправления необходимо использовать безопасные методы выполнения команд, такие как команда `nslookup`:

`output = subprocess.check_output(["nslookup", hostname], text=True)`

В этом случае команда ` nslookup ` и ее аргументы передаются в ` subprocess.check_output ` в виде списка.
Это предотвращает возможность командной инъекции, так как пользовательский ввод не интерпретируется как часть команды.

---

## 3. Моделирование угроз

1. Потенциальные проблемы безопасности:
     * отсутствие разделения ролей и привилегий доступа к компонентам системы повышает риск несанкционированного доступа, т.е. неясно, проверяет ли внутреннее приложение права пользователей перед обработкой запросов от Microfront;
     * отсутствие защиты трафика (SSL/TLS) между клиентами и компонентами сервиса может позволить перехватить чувствительные данные (учетные данные, конфигурации и т.д.);
     * централизованное хранение учетных данных в БД может создать единую точку отказа и повысить риск компрометации большого количества аккаунтов;
     * отсутствие контроля входных данных может привести к уязвимостям, таким как внедрение кода, межсайтовый скриптинг и другие атаки.
     * на схеме не указано, шифруются ли данные при передаче или в состоянии покоя;
     * сервис взаимодействует с внешними платформами, такими как Telegram и Slack;
     * на диаграмме не показаны механизмы контроля доступа между компонентами;
2. Последствия эксплуатации уязвимостей:
     * злоумышленники могут использовать уязвимости, чтобы перегрузить сервис и сделать его недоступным для легитимных пользователей (финансовые и репутационные потери для компании в случае успешных атак);
     * уязвимости в сервисе могут быть использованы для компрометации подключенных аккаунтов Telegram и Slack.
     * несанкционированный доступ и изменение данных, а также нарушение работоспособности системы;
     * компрометация большого количества учетных записей при взломе централизованного хранилища;
     * перехват конфиденциальных данных (учетных данных, конфигураций) злоумышленниками;
3. Способы исправления и смягчения рисков:
     * использование защищенных протоколов (HTTPS) для передачи данных между клиентами и компонентами;
     * реализация контроля входных данных и входной валидации для предотвращения уязвимостей;
     * распределенное хранение учетных данных с применением криптографических методов защиты;
     * внедрение механизмов разделения ролей и привилегий доступа к компонентам системы;
     * регулярное обновление компонентов системы и устранение известных уязвимостей;
     * проведение регулярного аудита безопасности и тестирования на проникновение.
4. Уточняющие вопросы к разработчикам:
    * Шифруются ли данные при передаче и в состоянии покоя? Если да, то какие методы шифрования используются?
    * Какие методы проверки и обеззараживания данных используются (в том числе валидации входных данных)?
    * Как обрабатываются известные уязвимости в используемых компонентах и библиотеках?
    * Проводятся ли аудиты безопасности или процедуры тестирования на проникновение?
    * Как в сервисе обеспечивается контроль прав пользователей и доступа?
    * Как обрабатываются инциденты безопасности и как о них сообщается?
    * Как защищена интеграция с Telegram и Slack?

---
