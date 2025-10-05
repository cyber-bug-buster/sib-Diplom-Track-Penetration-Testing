### Результаты тестирования
#### 1. **Уязвимости Denial of Service, Command Injection ([A05:2021-Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/), [A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))**
**Критичность:** <font color="red">**Высокая**</font>  
**Страница**: `http://92.51.39.106:8050/passcheck.php`  
**Описание**: На странице есть возможность проверить надежность пароля. При этом пароль проверяется по файлу паролей в операционной системе. Для чтения файла используется метод PHP `exec` с использованием пользовательского ввода в сыром ввиде.
```php
$pass = $_GET["password"];
exec("/bin/cat /usr/share/dict/words | grep " . $pass, $output, $status);
```

**Предложения по исправлению**:
- Не использовать опасные методы PHP по взаимодействию с ОС
- Использовать санитизацию и экранирование пользовательского ввода

Подробности реализации
- Заходим на исследуемую страницу и вводим в поле ввода `Password to check` любой пароль. После проверки пароля система отображает используемую shell-команду в интерфейсе.  
Используется следующий шаблон:  
`grep ^UserInput$ /etc/dictionaries-common/words`  
![password.png](pic/owasp/password.png)
- Попробуем повлиять на команду и введем один из спец. символов `&, &&, |, ||` для образования `pipeline` (конвейера) команд.
```sh
test | whoami
```
![response_denial.png](pic/owasp/response_denial.png)

- После отправки запроса на сервер сайт будет недоступен какое-то время, что является отказом в обслуживании. (Denial of Service)


#### 2. **Уязвимость [Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal) ([A01:2021-Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/))**
**Критичность:** <font color="red">**Высокая**</font>  
**Страница**: `http://92.51.39.106:8050/admin/index.php?page=login`  
**Описание**: Существует возможность обращения к файловой системе через параметр GET запроса, а так же запуска php-shell скрипта на сервере.
**Предложения по исправлению**:
- Валидация значений параметров запросов


Подробности реализации

1. Перейти на страницу `http://92.51.39.106:8050/admin/index.php?page=login`
2. В параметре `page` использовать следующий вектор атаки:
```
page=php://filter/read=convert.base64-encode/resource=../users/check_pass
```
В ответ получаем код запрошенной страницы в base64  
![base64.png](pic/owasp/base64.png)

3. Декодируем строку и получаем код страницы

![decode.png](pic/owasp/decode.png)


#### 3. **Уязвимость [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload). ([A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))**
**Критичность:** <font color="red">**Высокая**</font>  
**Страница**: `http://92.51.39.106:8050/pictures/upload.php`    
**Описание**:   
Уязвимость позволяет загрузить произвольный файл, отличный от картинки
Существует возможность выполнения следующих действий:
- Загрузить PHP-Shell файл

**Предложения по исправлению**:
- Добавить валидацию входного по типу содержимого. Например можно использовать сигнатурный анализ файла и сравнивать первые байты файлов с известными сигнатурами. Например, сигнатура для файлов формата JPEG будет выглядеть следующим образом: `FF D8 FF E0`.
- Запускать приложение под пользователем с минимальными правами. Пользователь не должен иметь прав на чтение системных файлов, тем более на их модификацию или удаление.

Подробности реализации

- Переходим на страницу загрузки файла и заполняем поля формы. Заполняем поле `File Name` произвольным именем файла с окончанием (расширением) `.php`. Далее выбираем специальный [php-shell.php](php-shell.php) файл.
![uploadfile.png](pic/owasp/uploadfile.png)

- После загрузки файла открывается страница просмотра загруженного файла. И в нашем случае картинка не отображается, т.к. был загружен файл с другим содержимым.
![failfail.png](pic/owasp/failfail.png)
- Используем уязвимость сервера, которая позволяет просматривать содержимое папки `upload` и определяем путь к нашему файлу. (http://92.51.39.106:8050/upload)  
![upload.png](pic/owasp/upload.png)
- Используем уязвимость `Path Traversal`. Переходим на страницу `http://92.51.39.106:8050/admin/index.php?page=../upload/shell/shell` и видим окно нашего shell-приложения.
![shellapp.png](pic/owasp/shellapp.png)
- Заходим в приложение и выполняем команду на сервере для отображения содержимого файла `/etc/passwd`. Сервер отправляет в ответ содержимое запрошенного файла.
![shellanswer.png](pic/owasp/shellanswer.png)


#### 4. **Уязвимость [SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) ([A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))**
**Критичность:** <font color="red">**Высокая**</font>  
**Страница**: `http://92.51.39.106:8050/users/login.php`  
**Описание**:
- Через форму авторизации пользователей есть возможность внедрить sql скрипт в поле логина.

Существует возможность выполнения следующих действий:
- Добавление, изменение данных в таблицах
- Удаление данных из таблиц
- Нарушение схемы БД

**Предложения по исправлению**:
- Добавить валидацию, санитизацию входных данных с формы


Подробности реализации

1. Перейти на страницу `http://92.51.39.106:8050/users/login.php` и в форме авторизации в поле логина использовать следующий вектор атаки:
```
' OR 1 -- -
```
![injection1.png](pic/owasp/injection1.png)

2. Запрос выполнился корректно.  
   Мы успешно авторизовываемся по пользователем `Sample User`

Проблема находится в данном участке кода
```php
 function check_login($username, $pass, $vuln = False)
   {
      if ($vuln)
      {
	 $query = sprintf("SELECT * from `users` where `login` like '%s' and `password` = SHA1( CONCAT('%s', `salt`)) limit 1;",
	                   $username,
	                   mysql_real_escape_string($pass));	 
      }
      else
      {
	 $query = sprintf("SELECT * from `users` where `login` like '%s' and `password` = SHA1( CONCAT('%s', `salt`)) limit 1;",
	                   mysql_real_escape_string($username),
	                   mysql_real_escape_string($pass));
      }
      $res = mysql_query($query);
```
Метод `mysql_real_escape_string` не вызывается для параметра `$username`.

#### 5. **Слабый пароль администратора ([A07:2021-Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/))**
**Критичность:** <font color="red">**Высокая**</font>  
**Страница**: `http://92.51.39.106:8050/admin/index.php?page=login`  
**Описание**:
- На странице авторизации администратора сайта используется слабый пароль `admin/admin`

Существует возможность получить несанкционированный доступ к административной консоли. Уязвимость со средней критичностью, т.к. текущая функциональность административной консоли небольшая, но в будущем может быть расширена.

**Предложения по исправлению**:
- Использовать сложный пароль
- Ввести ограничение на количество попыток авторизации


Подробности реализации

1. Перейти на страницу `http://92.51.39.106:8050/admin/index.php?page=login` и форме авторизации пользователя ввести логин/пароль:
   `admin/admin`
![auth.png](pic/owasp/auth.png)
![auths.png](pic/owasp/auths.png)   