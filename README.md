# Дизайн архитектуры решения продающего контент через мобильные приложения

## Оглавление

- [Дизайн архитектуры решения продающего контент через мобильные приложения](#дизайн-архитектуры-решения-продающего-контент-через-мобильные-приложения)
  - [Оглавление](#оглавление)
  - [Исходные данные](#исходные-данные)
      - [Требования](#требования)
      - [Ограничения](#ограничения)
    - [Допущения](#допущения)
    - [Stakeholders и перечень их интересов](#stakeholders-и-перечень-их-интересов)
    - [Пользовательские сценарии](#пользовательские-сценарии)
    - [Диаграмма контекста](#диаграмма-контекста)
    - [Диаграмма контейнеров](#диаграмма-контейнеров)
      - [Описание элементов диаграммы контейнеров](#описание-элементов-диаграммы-контейнеров)
    - [Ключевые атрибуты качества](#ключевые-атрибуты-качества)
    - [Диаграмма развертывания](#диаграмма-развертывания)
      - [Краткое описание диаграммы](#краткое-описание-диаграммы)
      - [Перечень используемых сервисов](#перечень-используемых-сервисов)
    - [Безопасность](#безопасность)
      - [Аутентификация и авторизация](#аутентификация-и-авторизация)
      - [Криптография](#криптография)
      - [Self-testing](#self-testing)
      - [Приватные подсети](#приватные-подсети)
      - [Технический аудит](#технический-аудит)
    - [Компоненты архитектуры в приземлении на AWS и OpenSource](#компоненты-архитектуры-в-приземлении-на-aws-и-opensource)

## Исходные данные

#### Требования

В данном разделе представлены бизнес-требования, которые сформулировал заказчик:

- REQ-1: Необходимо спроектировать архитектуру приложения продающего контент
- REQ-2: Витрина: мобильное приложение
- REQ-3: Оплата App Store, Google Play

#### Ограничения

В данном разделе представлены ограничения, которые сформулировал заказчик:

- CON-1: Количество зарегистрированных пользователей - 100 000
- CON-2: 100 TPS

### Допущения

Здесь описаны основные допущения, которые были выявлены в ходе исследования:

- ASM-1: Инфраструктура размещена в облаке (Amazon Web Services). На самом деле это не принципиально. Решение может быть развернуто также On Premise с использованием Open Source (перечислены в приложении).
- ASM-2: Предполагаем, что для разработки мы можем задействовать команду разработки (Java - Spring, Javascript - React, Swift, Kotlin)
- ASM-3: Предполагаем, что если у нас нагрузка 100 TPS, то количество запросов на чтение каталога может быть 500 RPS (в пять раз больше)
- ASM-4: Предполагаем, что каталог контента не очень большой (1000 наименований товаров)
- ASM-5: Берем за основу средний размер контента равный 10МБ.
- ASM-6: Предполагаем, что после покупки контент доступен клиенту на определенный период времени. По истечению этого периода контент блокируется.
- ASM-7: Предполагаем, что контент создают сотрудники нашего сервиса
- ASM-8: Берем за основу, что уровень доступности должен быть не менее 99,95

### Stakeholders и перечень их интересов

В данном разделе описаны группы заинтересованных лиц, которые могут оказывать влияние на дизайн решения, а также перечень их интересов, которые необходимо учитывать при проектировании решения

- **SH-1: Customer** - наш клиент, который приобретает товары
  - Простота пользовательского интерфейса и удобство работы
  - Отсутсвие ошибок при навигации по каталогу и приобретении контента
  - Скорость работы приложений
- **SH-2: Administrator** - сотрудник сервиса, который обеспечивает обработку заказов и актуализацию каталога контента
  - Простота пользовательского интерфейса и удобство работы
  - Отсутсвие ошибок при администрировании конетнта
  - Скорость работы веб-приложения
  - Минимальное количество проблемных обращений от клиентов
- **SH-3: Content producer** - сотрудник сервиса, который отвечает за создание и актуализацию качественного контента
  - Удобный пользовательский интерфейс для добавления конетента
- **SH-4: Product owner** - сотрудник сервиса, который формирует скоуп разработки, отслеживает основные метрики сервиса и определяет приоритеты
  - Наличие интрументов мониторинга бизнесовых метрик
  - Стабильность работы сервиса
  - Возможность выдерживания пиковых нагрузок (например, в случае активации распродаж)
  - Возможность масштабирования решения (плановое увеличение нагрузки, количества пользователей, объема хранимой информации)
- **SH-5: Developer** - сотрудник компании, который разработывает функционал сервиса
  - Современный стэк
  - Простота разработки и деплоя фич
  - Простота отладки
- **SH-6: DevSecOps** - сотрудник компании, который обеспечивает безотказную и безопасную работу сервиса
  - Понятные средства мониторинга логов, технических метрик
  - Наличие механизмов оперативного уведомления и сбоях и других проблемах
  - Простые механизмы автоматического тестирования и деплоя новых фич

### Пользовательские сценарии

Здесь описаны основные пользовательсткие сценарии, которые удалось выявить в ходе исследования:

- Создание контента и наполнение им каталога
  - UС-1.1: Content producer заходит в Product Management Backoffice
  - UС-1.2: Content producer загружает новый контент в виде ZIP архива и добавляет сопровождающую мета-информацию (наименование контента, дата публикации, автор и т.д.)
- Приобретение клиентом контента
  - UС-2.1: Customer заходит в Google Play или Apple App Store и скачивает наше мобильное приложение
  - UС-2.2: Customer устанавливает приложение на устройство
  - UС-2.3: Customer регистрируется в приложении используя SSO (учетная запись Google ID, либо Apple ID), либо вручную.
  - UС-2.4: Customer используя функции поиска или навигацию по каталогу контента находит требуемый “товар”.
  - UС-2.5: Customer открывает карточку товара.
  - UС-2.6: Customer нажимает кнопку “Купить”.Откроется помощник для осуществления покупки:
  - UС-2.7: Customer заполняет форму ввода персональных данных (ФИО, e-mail, страна, город, etc)
  - UС-2.8: Customer заполняет форму ввода платежных реквизитов (Visa, Mastercard, Paypal, etc)
  - UС-2.9: Customer подтверждает введенные данные и нажимает кнопку “Оплатить”
  - UС-2.10: Customer на на указанный им e-mail получает квитанцию об оплате контента.
  - UС-2.11: После обработки транзакции, Customer-у становится доступен приобретенный контент в разделе “Мои покупки” на определенный период времени.
  - UС-2.12: После истечения периода времени приобретенный контент становится недоступен для просмотра.
- Адмитристрирование контента:
  - UС-3.1: Administrator заходит в Product Management Backoffice
  - UС-3.2: Administrator может добавить новый контент
  - UС-3.3: Administrator может изменить мета-информацию существующего контента
  - UС-3.4: Administrator может удалить существующий контент из каталога

### Диаграмма контекста

Ниже представлена упрощенная диаграмма решения на уровне бизнес-контекста. Все пользователи взаимодействуют только с сервисами Google или Apple, а также используют почтовый сервис. Наш сервис интегрован с API Google и API Apple для деплоя приложений, реализации inApp платежей и других функций.

![Context diagram!](images/context.png 'Context diagram')

### Диаграмма контейнеров

Ниже представлена диаграмма решения на уровне отдельных контейнеров (по концепции C4Model). Данная диаграмма показывает из каких элементов на среднем уровне должно состоять проектируемое решение и как эти элементы будут взаимодействовать между собой.

![Container diagram!](images/container.png 'Container diagram')

#### Описание элементов диаграммы контейнеров

- **Mobile Application iOS** - мобильное приложение на платформе Apple iOS, доступное для скачивания в Apple Store
- **Mobile Application Android** - мобильное приложение на платформе Android, доступное для скачивания в Google Play
- **Google Play API** - API для взаимодействия с сервисами Google Play. В том числе Google Pay API.
- **Apple Store API** - API для взаимодействия с сервисами Apple Store. В том числе Apple Pay.
- **DNS** - сервис доменных имен
- **CDN** - сервис распределенной доставки и дистрибуции контента
- **WAF** - сервис для обнаружения и блокирования сетевых атак на сервис
- **API Gateway** - шлюз, обеспечивающий взаимодействие между клиентскими приложениями и нашими сервисами (API)
- **Auth Service** - сервис безопасной аутентификации и авторизации пользователей
- **Products API** - программный интерфейс приложения обеспечивающего работу с каталогом контента
- **Product and Order Management Backoffice** - веб-интерфейс обеспечивающий работу с каталогом контента, добавление нового контента и актуализацию уже загруженного, а также управление заказами и покупками клиентов
- **S3 Storage** - объектное хранилище для статического контента и исторических данных
- **Products Database** - БД для хранении информации о продуктах
- **Orders Database** - БД для хранения информации о заказах
- **Products Cache** - промежуточное in-memory хранилище данных о продуктах
- **Orders API** - программный интерфейс приложения обеспечивающего работу с заказами
- **Message Broker** - брокер сообщений обеспечивающий асинхронную обработку различных событий (например, отправка Push уведомлений клиентам, или квитанций об оплате)
- **Reporting** - система построение аналитических отчетов на основе данных S3
- **CI/CD** - здесь можно будет хранить артефакты разработки, реализовать репозиторий код, реализовать пайплайны CI/CD
- **Log and tracing** - ElasticSearch для поиска информации, LogStash для сбора, обработки логов, Kibana для визуализации

### Ключевые атрибуты качества

Поскольку проектируемая архитектура должна опираться на определенные бизнес-требования, ограничения и допущения ниже описаны основные атрибуты качества.

**Products Service**

- **QA-1**: Perfomance
  - Сервис должен выдерживать не менее 500RPS (в соответствии с допущениями которые мы определили ранее)
- **QA-2**: Availability
  - Сервис должен быть доступен 24/7. Показатель доступности 99.95.
- **QA-3**: Fault tolerance
  - Сервис должен иметь возможность в случае падения одного из инстансов быстро переключаться на работающий инстанс
  - Сервис должен иметь возможности по предотвращению сбоев с использованием территориально распределенных ЦОДов

**Orders Service**

- **QA-4**: Security
  - Необходимо обеспечить безопасность персданных, сведений и платежах и заказах
  - Вся информация о сделанных клиентом заказах и проведенных платежах должна быть доступна длительное время и защищена от аттак
- **QA-5**: Reliability
  - Сервис заказов должен быть устойчив к нагрузке (в том числе созданной исскуственно) и выдерживать не менее 100TPS
- **QA-6**: Fault tolerance
  - Сервис должен иметь возможность в случае падения одного из инстансов быстро переключаться на работающий инстанс
  - Сервис должен иметь возможности по предотвращению сбоев с использованием территориально распределенных ЦОДов

**Mobile Applications**

- **QA-7**: Usability
  - Мобильные приложения должны быть спроектированы таким образом, чтобы пользователю не приходилось привыкать к пользовательскому интерфейсу и проходить какое-либо обучение
- **QA-8**: Modifiability
  - Приложения должны быть написаны с использованием слабо-связанных программных компонентов для возможности быстрого добавления нового функционала и изменения существующего.
- **QA-9**: Availability
  - Приложения должны иметь высокий уровень доступности. Для нас это означает высокое качество кода мобильных приложений и высокий уровень доступности сервисов Order Service и Product Service

**Backoffice**

- **QA-10**: Usability
  - Веб-приложение должно быть интуитивно понятным для пользователя

### Диаграмма развертывания

Ниже представлена диаграмма развертывания для публикации решения в облаке. Данная диаграмма показывает как логические элементы решения (см. Контейнерная диаграмма) приземляются на сервисы PaaS и SaaS вендора облачной инфраструктуры.

![Deployment diagram!](images/deployment.png 'Deployment diagram')

#### Краткое описание диаграммы

As stated in the Target container diagram we propose to use AWS Managed Streaming for Apache Kafka for synchronous and asynchronous calls between our services.

We could use AWS S3 as a central datastore for assets, data for Artificial Intelligence Modules and bucketsbucket for our Farmacy Food and Farmacy Family backups.

We propose to use additional API layer with AWS API Gateway before Web and Mobile apps for additional security, Insulates the clients from how the application is partitioned into services, reduces the number of requests/roundtrips, etc

We place the Social Network Module application engine into AWS EC2 Container and connect it to the RDS Database for storing all domain-specific information. For storing assets and user files we propose to use AWS S3 buckets.

Inventory Replenishment Module and eDietary Module. All modules related to artificial intelligence based on serverless computing approach and requires services such as AWS Forecast, AWS Lambda, AWS RDS for storing relational data and AWS DynamoDB for NoSQL.

Users Engagement Analys module requires such services as AWS Glue for ETL operations and AWS Authena for data analysis (see ADR-11 Data Analysis Capability).

Profile Module should use Lambda for computing and AWS RDS for storing data.

Clinics Gateway Module is useduses for secure communication between clinics and Farmacy Familyour system and consists of AWS Lambda for computing, DynamoDB for storing data and also AWS API Gateway. It also requires a secure VPN connection with AWS Transit Gateway.

We have prepared a list of all the necessary services for your convenience.

The Multi-zone approach will be used to allow redundancy of critical data and high availability on the next step of the project.

#### Перечень используемых сервисов

The table below provides a list of infrastructure elements that are required for the implementation of the project.

<table>
  <tr>
   <td><strong>Наименование сервиса</strong>
   </td>
   <td><strong>Вендор</strong>
   </td>
   <td><strong>Описание</strong>
   </td>
  </tr>
  <tr>
   <td>Application Load Balancer
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/network-firewall">AWS Network Firewall </a>is a managed service that makes it easy to deploy essential network protections for all of your Amazon Virtual Private Clouds (VPCs)
   </td>
  </tr>
  <tr>
   <td>Route 53
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/route53">Amazon Route 53</a> is a scalable and highly available Domain Name System (DNS) service
   </td>
  </tr>
  <tr>
   <td>Transit Gateway
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/ru/transit-gateway">Amazon Transit Gateway</a> connect Amazon VPCs, AWS accounts, and on-premises networks to a single gateway
   </td>
  </tr>
  <tr>
   <td>EC2
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/ec2">Amazon EC2</a> provides Secure and resizable compute capacity to support virtually any workload
   </td>
  </tr>
  <tr>
   <td>Cognito
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/cognito">Amazon Cognito</a> provides Simple and Secure User Sign-Up, Sign-In, and Access Control
   </td>
  </tr>
  <tr>
   <td>S3
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/s3">Amazon S3</a> - Object storage built to retrieve any amount of data from anywhere
   </td>
  </tr>
  <tr>
   <td>API Gateway
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/api-gateway">Amazon Api Gateway</a> for create, maintain, and secure APIs at any scale
   </td>
  </tr>
  <tr>
   <td>CloudFront
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/cloudfront">Amazon CloudFront</a> securely deliver content with low latency and high transfer speeds
   </td>
  </tr>
  <tr>
   <td>SNS
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/sns">Amazon SNS</a> for Fully managed pub/sub messaging, SMS, email, and mobile push notifications
   </td>
  </tr>
  <tr>
   <td>MSK
   </td>
   <td>AWS
   </td>
   <td>Securely stream data with a fully managed, highly available <a href="https://aws.amazon.com/msk">Apache Kafka service</a>
   </td>
  </tr>
  <tr>
   <td>DynamoDB
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/dynamodb">Amazon DynamoDB</a> - fast, flexible NoSQL database service for single-digit millisecond performance at any scale
   </td>
  </tr>
  <tr>
   <td>RDS
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/rds/">Amazon RDS</a> - Set up, operate, and scale a relational database in the cloud with just a few clicks
   </td>
  </tr>
  <tr>
   <td>Glue
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/glue/">Amazon Glue</a> - is a fully-managed, pay-as-you-go, extract, transform, and load (ETL) service that automates the time-consuming steps of data preparation for analytics.
   </td>
  </tr>
  <tr>
   <td>Athena
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/athena/">Amazon Athena</a> - Amazon Athena is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL. Athena is serverless, so there is no infrastructure to manage, and you pay only for the queries that you run.
   </td>
  </tr>
  <tr>
   <td>Lambda
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/">Amazon Lambda</a> for run code without thinking about servers or clusters
   </td>
  </tr>
  <tr>
   <td>Forecast
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/forecast/">Forecast</a> business outcomes easily and accurately using machine learning
   </td>
  </tr>
  <tr>
   <td>DataDog
   </td>
   <td>DataDog
   </td>
   <td><a href="https://www.datadoghq.com/">Datadog</a> is an observability service for cloud-scale applications, providing monitoring of servers, databases, tools, and services, through a SaaS-based data analytics platform
   </td>
  </tr>
  <tr>
   <td>CloudFormation
   </td>
   <td>AWS
   </td>
   <td><a href="https://aws.amazon.com/cloudformation">Amazon CloudFormation</a> helps speed up cloud provisioning with infrastructure as code
   </td>
  </tr>
</table>

### Безопасность

Для того, чтобы минимизировать риски утечек чувствительной информации о пользователях, их платежных данных и совершенных покупках, а также защитить систему от различных сбоев в ходе аттак на сервис, ниже предложен ряд мер.

#### Аутентификация и авторизация

Для реализации функций безопасной аутентификации и авторизации пользователей (компонент Authentication Service) Amazon предлагает AWS Cognito. Этот сервис можно будет использовать как для авторизации клиентов, так и внутренних пользователей с использованием OpenID Connect, OAuth 2.0 или SAML 2.0. Также он позволяет защитить пользовательские данные от взлома в ходе хаккерских атак.

#### Криптография

Наши данные должны быть покрыты шифрованием с использованием 256-bit Secure Socket Layer (SSL) как при использовании мобильных клиентских приложений, так и веб-приложений во внутреннем контуре.

#### Self-testing

Необходимо обеспечить регулярный прогон тестов: сканирование портов, тестирование на предмет SQL injection. Для хаотичного тестирования можно использовать наработки Netflix (https://github.com/Netflix/SimianArmy)

#### Приватные подсети

Необходимо использовать приватные подсети для защиты от прямого доступа к данным и сервисам из сети Интернет.

#### Технический аудит

Необходимо регулярно проводить внешний аудит системы для того, чтобы иметь независимую оценку о защищенности системы и хранимых данных.

### Компоненты архитектуры в приземлении на AWS и OpenSource

-
