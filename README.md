Проект YaMDb в Docker-контейнерах
----------------------------------------------------------------

### Для первого запуска проекта 
из папки 
 ``` infra_sp2 / infra ```, содержащей файл ```docker-compose```
необходимо выполнить команду: 

``` 
docker-compose up -d --build 
``` 

для создания образа **_Docker_** основного приложения **_web_**, подгрузки образов 
**_postgres_** и **_nginx_** с сайта _**DockerHub**_ и компоновки их в 
**_docker-compose_**.

После запуска необходимо выполнить команды для выполнения миграций в базе данных, 
сбора статики и создания суперпользователя:
```angular2html
docker-compose exec web python manage.py migrate

docker-compose exec web python manage.py collectstatic --no-input 

docker-compose exec web python manage.py createsuperuser
```
После этого будет доступна админская часть приложения по адресу:

http://localhost/admin/,
в которую можно войти как суперпользователь.

Также будет доступна документация приложения в формате _**yaml**_ по адресу:

http://localhost/redoc/.

## Проект YaMDb собирает отзывы (Reviews) пользователей на произведения (Titles).
Сами произведения в YaMDb не хранятся, здесь нельзя посмотреть фильм или 
послушать музыку. В каждой категории есть произведения: книги, фильмы или музыка.
Произведения делятся на категории: «Книги», «Фильмы», «Музыка». 
Каждый зарегистрированный пользователь может оставить отзыв на понравившееся 
произведение и(или) комментарий под отзывами других пользователей.

### Заполнение базы данных приложения можно выполнить через админку или с помощью веб-запросов. 


### Незарегистрированным пользователям доступны только GET-запросы:
* http://localhost/api/v1/titles/ - список всех произведений
* http://localhost/api/v1/titles/{titles_id}/ - получение произведения по ID
* http://localhost/api/v1/titles/{title_id}/reviews/ - получение списка всех 
отзывов на конкретное произведение
* http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - 
получение отзыва по ID
* http://localhost/api/v1/categories/ - список всех категорий
* http://localhost/api/v1/genres/ - список всех жанров
* http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/ - 
получение списка всех комментариев к отзыву
* http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/{comment_id}/ 
-получение комментария к отзыву

### Алгоритм регистрации пользователей:

1. Пользователь отправляет **POST**-запрос на добавление нового пользователя с 
параметрами **_email_** и _**username**_ на эндпоинт http://localhost/api/v1/auth/signup/.
2. YaMDB отправляет письмо с кодом подтверждения (_**confirmation_code**_) 
на адрес _**email**_.
3. Пользователь отправляет POST-запрос с параметрами **_username_** и 
_**confirmation_code**_ на эндпоинт http://localhost/api/v1/auth/token/, в ответе на
запрос ему приходит token (JWT-токен).
4. При желании пользователь отправляет **PATCH**-запрос на эндпоинт 
http://localhost/api/v1/users/me/ и заполняет поля в своём профайле (описание 
полей — в документации).

### Зарегистрированным пользователям доступны запросы:
Те же, что и незарегистрированным, а также:
* http://localhost/api/v1/users/me/ - **(GET-запрос)** получение данных своей 
учетной записи
* http://localhost/api/v1/users/me/ - **(PATCH-запрос)** изменение данных своей 
учетной записи
* http://localhost/api/v1/titles/{title_id}/reviews/ - **(POST-запрос)** добавление 
отзыва
* http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - **(PATCH-запрос)** 
частичное обновление своего отзыва
* http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/ - 
**(POST-запрос)** добавление комментариев к отзыву 

### Пользовательские роли

1. Аноним — может просматривать описания произведений, читать отзывы и комментарии.
2. Аутентифицированный пользователь (user) — может, как и Аноним, читать всё, 
дополнительно он может публиковать отзывы и ставить оценку произведениям 
(фильмам/книгам/песенкам), может комментировать чужие отзывы; может редактировать 
и удалять свои отзывы и комментарии. Эта роль присваивается по умолчанию каждому 
новому пользователю.
3. Модератор (moderator) — те же права, что и у Аутентифицированного пользователя 
плюс право удалять любые отзывы и комментарии.
   * __ __ -
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/{comment_id}/ - 
   **(PATCH-запрос)** частичное обновление комментария к отзыву
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - 
   **(PATCH-запрос)** частичное обновление отзыва по id
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - 
   **(DELETE-запрос)** Удаление отзыва по id
4. Администратор (admin) — полные права на управление всем контентом проекта. 
Может создавать и удалять произведения, категории и жанры. Может назначать роли
пользователям.
Суперюзер Django — обладает правами администратора (admin)
   * http://localhost/api/v1/users/ - **(GET-запрос)** получение списка всех 
   пользователей
   * http://localhost/api/v1/users/ - **(POST-запрос)** добавление нового 
   пользователя
   * http://localhost/api/v1/users/{username}/ - **(GET-запрос)** получение 
   пользователя по имени
   * http://localhost/api/v1/users/{username}/ - **(PATCH-запрос)** изменение 
   данных пользователя по имени
   * http://localhost/api/v1/users/{username}/- **(DELETE-запрос)** удаление 
   пользователя
   * http://localhost/api/v1/titles/ - **(POST-запрос)** Добавление произведения
   * http://localhost/api/v1/titles/{titles_id}/ - **(PATCH-запрос)** Частичное 
   обновление информации о произведении
   * http://localhost/api/v1/titles/{titles_id}/ - **(DELETE-запрос)** удаление
   произведения
   * http://localhost/api/v1/genres/ - **(POST-запрос)** Добавление жанра
   * http://localhost/api/v1/genres/{slug}/ - **(DELETE-запрос)** удаление жанра
   * http://localhost/api/v1/categories/ - **(POST-запрос)** Добавление категории
   * http://localhost/api/v1/categories/{slug}/ - **(DELETE-запрос)** удаление 
   категории
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/{comment_id}/ -
   **(PATCH-запрос)** частичное обновление комментария к отзыву
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - 
   **(PATCH-запрос)** частичное обновление отзыва по id
   * http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/ - 
   **(DELETE-запрос)** Удаление отзыва по id
