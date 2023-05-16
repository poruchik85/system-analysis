### ES

Модель:
https://miro.com/app/board/uXjVMJQgG2E=/?share_link_id=348484834524

В ES я выделил следующие контексты:
- Заказ услуги. Всё, что непосредственно связано с выполнением заказа: его создание(копирование), подбор воркера, определение стоимости, получение воркером расходников, собственно выполнение. Все эти операции имеют строгий порядок выполнения (или даже прямо вызывают следующую), и необходимы для достижения основной бизнес-цели (выполнение заказа);
- Склад - приём печенек и сборка пакетов расходников. Я вынес это в отдельный контекст, потому что к основному процессу это относится косвенно (выдачу расходников можно вообще отдать на аутсорс - для воркера разницы нет, как интерфейс получения расходников будет реализован). Впрочем, то же самое касается и всех остальных контекстов;
- Матчинг - по описанию можно понять так, что команда, реализующая матчинг, пилит только отдельные шаги, и предоставляет сервис для получения этих шагов. Таким образом, "Алгоритм матчинга" - это внешний сервис, предоставляющий некий перечень элементов алгоритма, из  которых уже менеджер составляет сам алгоритм ([US-070]). Если это не так (уточнить у бизнеса), и "Алгоритм матчинга" - полностью самодостаточная вещь, то он полностью станет ЧЯ, будет лишь принимать клиента и услугу, и, опираясь на список воркеров, возвращать нужного.
- Аналитика - сервис аналитики вынесен отдельно по шаблону работы с данными - во первых, данные им не нужны онлайн, а во вторых - наверняка в ближайшем будущем бизнес попросит отчеты-агрегации, так что для этого контекста скорее всего лучше подойдёт OLAP; 
- HR - процесс найма воркеров ортогонален основному процессу, поэтому отделён от него;
- Ставки - это незначащая система (в смысле, от неё никакая другая не зависит, в том числе основной процесс);
- Биллинг - ему придётся работать с непредсказуемыми внешними сервисами, бОльшая часть его работы - асинхронна;
- Нотификации - вынес это в отдельный контекст, потому что он требуется везде. Кроме того, сейчас по требованиям нужно отправлять только почту, только вот так не бывает. На первом же демо представитель бизнеса с виноватой улыбкой скажет: "Всё отлично, только вы забыли про sms/push/почтовых голубей...". В этом контексте нужные каналы и будут добавляться. В общем - это отдельный сервис со своей БД, связанный с другими частями только функционально на вход.

Также, поскольку клиентами будут выступать существующие сотрудники компании, нет смысла на первом этапе добавлять процесс регистрации для них. Можно вопользоваться существующим auth service - или вновь созданным, но он в любом случае будет частью другой системы.

### Модель данных
[pdf](Модель%20данных.pdf)  
[diagrams.net](https://drive.google.com/file/d/1lSLIith1bIXC-tx-gP0iHxahlD1Ndxfw/view?usp=sharing)

### Общая модель
[pdf](Итоговая%20модель.pdf)  
[diagrams.net](https://drive.google.com/file/d/1iRLvDC_k0psoFXp6V5SIJfz8gbq2W6-u/view?usp=sharing)

### Реализация проекта
1. Основная часть проекта ("Заказ услуги", "Склад", "HR", "Биллинг", "Ставки") будет выполнена в виде модульного монолита. Такой подход позволит с одной стороны внедрить основной процесс максимально быстро (про минимальный TTM с микросервисами в уроке - не согласен. Разработка микросервисов сама по себе не проще аналогичного по функциям монолита, а сверху ещё накладывается инфраструктурный ад. То есть монолит - это объективно гораздо проще и дешевле, тем более никаких предпосылок для хайлоада в проекте пока не видно), а с другой - в случае необходимости можно будет отщипнуть отдельные сервисы с переносом данных в их собственные БД.
2. Матчинг - видимо отдельный сервис. Раз там ребята такие секретные, да ещё и язык свой придумали - пусть сами и корячатся. С ними нужно только плотно проработать глоссарий и интерфейсы.
3. Аналитика - отдельная система со своей аналитической БД. Данные туда будут синкаться асинхронно из основной базы.
4. Нотификации - отдельный сервис со своей БД. Взаимодействие асинхронное (ввиду использования внешних сервисов). Можно поинтересоваться у материнской компании - может у них это уже есть.
5. Про возможность ddos - эта история скорее не про разработку, а про девопс, но при регистрации можно минимально добавить какую-нибудь рекапчу.

### Спорные места:
- работа матчинга. Нужно прояснить, что должны делать менеджеры в основном контексте. Возможно, что ничего;
- проверка тестов кандидатов в воркеры. Выглядит так, что тесты проверяются автоматически, но возможно это не так - тогда в HR добавится соответствующая операция;
- не уверен, следует ли включать процессы управления менеджерами (добавление/удаление/мутации).