**Выполнил: Радионов Никита Дмитриевич БПИ236**

Были реализованы следующие эндпоинты, которые доступны пользователю:

- `/api/files/upload` - загрузить файл в систему
- `/api/files/{fileId}` - получить файл по его id
- `/api/analysis/{fileId}` - выполнить анализ файла по его id
- `/api/analysis/wordcloud/{fileId}` - получить картинку облака слов по id файла

Таким образом проект реализует четыре схемы, которые были описаны в задании.

Также можно посмотреть swagger по эндпониту: `/swagger`. Через него можно будет вручную протестировать вышеперечисленные эндпоинты.

В рамках задания рассматривается работа только с .txt файлами, поэтому подразумевается, что, если пользователь отправляет файлы, то отправляет только их.

### 1. Архитектура решения

Были реализованы три микросервиса, каждый из которых выполняет определенную роль. Все микросервисы взаимодействуют напрямую между собой по протоколу HTTP.
Каждый микросервис реализует свой REST API. Ни один из микросервисов (кроме API Gateway) не доступен напрямую пользователю — все они скрыты за Gateway. Все микросервисы обрабатывают ошибки на уровне бизнес-логики и возвращают корректные HTTP-статусы (400, 404, 500 и т.д.).

В рамках каждого микросервиса реализована базовая трёхслойная архитектура: слой контроллеров (транспортный уровень), слой бизнес-логики и слой хранения данных (взаимодействие с базой данных и файловой системой). Благодаря этому обеспечивается разделение ответственности и упрощается сопровождение кода.

При недоступности любого внутреннего сервиса API Gateway перехватывает исключение HttpRequestException и возвращает пользователю 503 Service Unavailable с поясняющим сообщением (например «FileStoringService is unavailable» и т.д.).

**ApiGateway**

Отвечает за маршрутизацию входящих HTTP-запросов от пользователя к соответствующим внутренним сервисам. Пользователь взаимодействует исключительно с этим компонентом Остальные сервисы не доступны извне.

**FileStoringService**

Отвечает за прием, хранение и выдачу загруженных текстовых файлов. Каждый файл сохраняется на диске и сопровождается записью в базе данных, содержащей метаданные, такие как имя, хэш и путь к файлу. При загрузке файла в FileStoringService вычисляется SHA-256 хэш содержимого.

**FileAnalisysService**

Выполняет анализ загруженных файлов, включая подсчет количества абзацев, слов и символов. Также генерирует облако слов с использованием внешнего API и сохраняет результаты анализа для последующего доступа.


### 2. Про Docker

Каждый микросервис разворачивается в отдельном Docker-контейнере. 
Для FileStoringService и FileAnalisysService поднимаются собственные экземпляры PostgreSQL. Это обеспечивает изоляцию данных и соответствие принципам микросервисной архитектуры.

### 3. Запуск

Чтобы запустить проект, нужно перейти в терминале в директорию с решением (в ней находится файл docker-compose.yml) и выполнить команду

```
docker-compose up --build
```

Появятся нужные образы и на основе них запустятся контейнеры.

После чего, можно будет обращаться к

```
localhost:5000
```

по соответствующему эндпоинту для использования функционала.
