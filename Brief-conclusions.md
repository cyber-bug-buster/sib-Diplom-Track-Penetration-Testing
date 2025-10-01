## Краткие выводы

По итогам тестирования были обнаружены следующие проблемы:

| № | Критичность | Название | Приложение |
|---|-------------|----------|------------|
| 1 | <font color="red">**Высокая**</font> | **Уязвимости Denial of Service, Command Injection ([A05:2021-Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/), [A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))** | Оба приложения |
| 2 | <font color="red">**Высокая**</font> | **Уязвимость [Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal) ([A01:2021-Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/))** | Оба приложения |
| 3 | <font color="red">**Высокая**</font> | **Уязвимость [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload). ([A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))** | Оба приложения |
| 4 | <font color="red">**Высокая**</font> | **Уязвимость [SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) ([A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/))** | Оба приложения |
| 5 | <font color="red">**Высокая**</font> | **Слабый пароль администратора ([A07:2021-Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/))** | NetologyVulnApp |
| 6 | <font color="orange">**Средняя**</font> | **Использование чужой сессии. ([A07:2021-Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/))** | NetologyVulnApp |
| 7 | <font color="orange">**Средняя**</font> | **Уязвимость к XSS атакам ([A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/), [Stored XSS](https://owasp.org/www-community/attacks/xss/#stored-xss-attacks))** | Оба приложения |
| 8 | <font color="orange">**Средняя**</font> | **Уязвимость к BruteForce атакам. ([A07:2021-Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/))** | Оба приложения |
| 9 | <font color="orange">**Средняя**</font> | **Отсутствие защиты от атак типа Сlickjacking, XSRF. ([A01:2021-Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/))** | Оба приложения |

На тестируемом сервисе `NetologyVulnApp` есть функционал добавления в корзину картинок и покупки. Средством монетизации сайта вероятно является именно посредничество при покупке и продаже, соответственно простой сайта на неопределенное время, использование чужой сессии для покупок, кража базы данных и добавление товара в чужую корзину является серьезным риском потери прибыли и репутации.  
На другом тестируемом сервисе `Beemer` есть функционал проверки доступности сервера по IP и я так понимаю блог про автомобили. Тут средством монетизации веротно будет реклама.  
В любом случае, помимо издержек от остановки сайта, компания может стать жертвой шпионажа или майнеров криптовалюты, которые будут использовать мощности сервера.

В первую очередь рекомендованы к устранению уязвимости с `Высокой` уровнем критичности.  
Для устранения уязвимостей и профилактики появления новых уязвимостей рекомендуется:
- Оценить уровень компании по OWASP SAMM
- Проверить процессы разработки ПО по `ГОСТ Р ИСО/МЭК 12207-2010`
- Внедрить практики безопасной разработки ПО, такие как статический анализ кода, динамический анализ кода, анализ зависимостей ПО. За основу можно взять `ГОСТ Р 56939-2016 Защита информации. Разработка безопасного программного обеспечения. Общие требования.`
- Внедрить систему менеджемта ИБ (`ГОСТ Р ИСО/МЭК 27001-2021`)
