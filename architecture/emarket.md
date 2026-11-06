# Общая архитектура всей платформы
```
# emarket — Senior Java Fullstack Developer Portfolio Project
```

```
Я рекомендую построить emarket — полноценную e-commerce платформу с
микросервисной архитектурой, в которой пользователь сможет регистрироваться,
просматривать товары, добавлять их в корзину, оформлять заказы, оплачивать
покупки и отслеживать историю заказов.
```

```
Отдельный интерфейс будет предназначен для администраторов: управление товарами,
категориями, заказами, пользователями, оплатами и уведомлениями.
```

```
Но главная цель проекта — не просто создать интернет-магазин. Нужно построить
систему, архитектуру которой можно обсуждать на собеседовании Senior Java
Developer.
```

```
Предлагаю следующие архитектурные принципы:
```

`1. Никакого монолита.` 

`2. Каждый микросервис — отдельный GitHub repository.` 

`3. Каждый микросервис имеет собственный CI/CD и независимый deployment.` 

`4. Микросервисы добавляются постепенно, от простых к сложным.` 

`5. API Gateway — единственная публичная точка входа в backend.` 

`6. Web и Mobile используют отдельные BFF.` 

`7. Каждый микросервис владеет собственной базой данных.` 

`8. Все backend-сервисы используют Java 21 и Gradle Kotlin DSL.` 

`9. Поддерживаются два режима: `default` и `deploy`.` 

```
10. Вся инфраструктура должна быть доступна для локального обучения без платных
облачных сервисов.
```

```
## 1. Общая архитектура
```

```
[Designing an E-commerce Platform in 45 Minutes: A Complete System Design
Interview
```

```
Guide](https://images.openai.com/static-rsc-4/NPRJxSE_cIsiPiefcQV2TWyRq8cxqtmpeG
1dmZWV-6V1xiSb3-
```

```
53LKhHdrrd5YmL4Yfp6jgoOo5BKut_3K5fuOq9wEiV7Xmb_jlCSRHE39JefwOvSeYenT0trBczo2VyJU
WvtdpvddWW0k-Zp2PjbfaD5t3lWMy5g_SofmAMLUs?purpose=inline)
```

```
[designgurus.substack.com](https://designgurus.substack.com/p/designing-an-e-
commerce-platform?utm_source=chatgpt.com)
```

```
Предлагаемая архитектура:
```

```
                   USERS
                     |
          +----------+----------+
          |                     |
       Web App              Mobile App
      React + TS              Mobile
          |                     |
          +----------+----------+
                     |
              API GATEWAY
          Spring Cloud Gateway
              Port 8080
                     |
          +----------+----------+
          |                     |
       Web BFF               Mobile BFF
          |                     |
          +----------+----------+
                     |
        +------------+-------------+
        |            |             |
```

```
     Identity      User         Catalog
     Service      Service       Service
        |            |             |
      DB Auth      DB User       DB Catalog
        |
        +-----------------------------+
        |             |               |
      Cart          Order          Payment
     Service       Service         Service
        |             |               |
      Redis       PostgreSQL      PostgreSQL
                      |
                    Kafka
                      |
          +-----------+-----------+
          |           |           |
      Inventory   Notification   Analytics
       Service      Service      Service
```
```

```
Это логическая схема, а не требование реализовать все сервисы одновременно.
Вспомогательная инфраструктура:
```

```
```
Spring Cloud Config
HashiCorp Vault
Kafka + Schema Registry
Redis
PostgreSQL
MongoDB
OpenSearch
SeaweedFS
Debezium
Prometheus
Grafana
ELK
Jaeger
WireMock
```
```

```
### Принцип взаимодействия
Web:
```
React
  |
  v
API Gateway
  |
  v
BFF Web
  |
  +---- Identity Service
  +---- User Service
  +---- Catalog Service
  +---- Cart Service
```
Mobile:
```

```
Mobile Application
```

```
  |
  v
API Gateway
  |
  v
BFF Mobile
  |
  +---- Identity Service
  +---- User Service
  +---- Catalog Service
```
```

```
Важное архитектурное решение: Web и Mobile никогда не обращаются непосредственно
к внутренним микросервисам.
```

```
Также BFF не является публичной точкой входа. Весь внешний HTTP-трафик проходит
через Gateway.
```

```
## 2. Организация GitHub
```

```
Чтобы независимые репозитории не были разбросаны по личному аккаунту, рекомендую
создать отдельную GitHub Organization:
```

```
GitHub Organization
```

```
# emarket-platform
```

```
github.com/emarket-platform
```

```
Backend repositories
```

```
Independent
```

```
Каждый сервис — отдельный репозиторий и отдельный pipeline.
```

```
Frontend repositories
```

```
Independent
```

```
Web и Mobile имеют собственные циклы разработки.
```

```
Infrastructure repositories
```

```
Independent
```

```
Docker, Kubernetes, Observability, WireMock.
```

```
Documentation
```

```
Centralized
```

```
Общая архитектурная документация и ADR.
```

```
Рекомендуемая организация:
```

```
- Organization: `emarket-platform`.
```

- `Team: `backend`.` 

```
- Team: `frontend`.
```

```
- Team: `infrastructure`.
```

```
- Team: `maintainers`.
```

```
Для управления проектом удобно использовать GitHub Projects с доской:
```

```
`Backlog → Ready → In Progress → Review → Done`.
```

```
### 2.1. Полный список GitHub repositories
```

```
Предлагаю 25 репозиториев. Не нужно создавать и реализовывать их все сразу:
список определяет целевую архитектуру.
```

# `Frontend` 

```
| Repository       | Назначение
|
| ---------------- |
------------------------------------------------------------------------ |
| `emarket-web`    | React + TypeScript + Vite, пользовательский и
административный интерфейс |
| `emarket-mobile` | Мобильное приложение
```

```
|
```

# `API и BFF` 

```
| Repository            | Назначение                         |
| --------------------- | ---------------------------------- |
| `emarket-api-gateway` | Единственная публичная точка входа |
| `emarket-bff-web`     | Backend for Frontend для Web       |
| `emarket-bff-mobile`  | Backend for Frontend для Mobile    |
```

# `Основные бизнес-микросервисы` 

```
| Repository                     | Назначение
|
| ------------------------------ |
-------------------------------------------------------- |
| `emarket-identity-service`     | Регистрация, OAuth2/OIDC, JWT, refresh token,
RBAC       |
| `emarket-user-service`         | Профиль, адреса, настройки пользователя
|
| `emarket-catalog-service`      | Товары, категории, характеристики
|
| `emarket-inventory-service`    | Остатки и резервирование товаров
|
| `emarket-cart-service`         | Корзина покупателя
|
| `emarket-order-service`        | Заказы и история покупок
|
| `emarket-payment-service`      | Оплата и Fake Payment Provider
|
| `emarket-pricing-service`      | Цены, скидки, промокоды
|
| `emarket-notification-service` | Email и in-app уведомления
|
```

- `| `emarket-review-service`       | Отзывы и рейтинги | | `emarket-search-service`       | Полнотекстовый поиск | | `emarket-admin-service`        | Административные операции и агрегированные представления | | `emarket-analytics-service`    | Аналитика и отчёты |` 

# `Инфраструктура и инструменты` 

- `| Repository               | Назначение | | ------------------------ | -------------------------------------------------|` 

```
| `emarket-config-server`  | Spring Cloud Config Server
```

```
|
| `emarket-config-repo`    | Централизованные конфигурации
|
```

```
| `emarket-api-mock`       | WireMock, независимые Web/Mobile mock responses
|
```

```
| `emarket-infrastructure` | Docker Compose, Minikube, Kubernetes manifests
|
```

```
| `emarket-observability`  | Prometheus, Grafana, ELK, Jaeger
|
```

```
| `emarket-docs`           | HLD, LLD, ADR, UML, C4, Mermaid, API documentation
|
```

```
Итого: 25 repositories
```

```
Каждый бизнес-сервис должен содержать собственную реализацию, тесты, Dockerfile,
CI и deployment manifests. Не нужно создавать общий репозиторий со всеми Java-
сервисами.
```

```
### 2.2. Как управлять независимыми репозиториями
```

```
Рекомендую два уровня организации:
```

```
```
GitHub Organization: emarket-platform
                  |
                  +--- 25 independent repositories
                  |
                  +--- GitHub Projects
                  |
                  +--- GitHub Actions
                  |
                  +--- GitHub Teams
```
```

```
Дополнительно можно создать локальную рабочую директорию:
```

```
~/workspace/
    emarket/
        frontend/
            emarket-web/
            emarket-mobile/
        backend/
            emarket-api-gateway/
            emarket-bff-web/
            emarket-bff-mobile/
            emarket-identity-service/
            emarket-user-service/
            ...
        infrastructure/
            emarket-infrastructure/
            emarket-observability/
        tools/
            emarket-api-mock/
        documentation/
            emarket-docs/
```
```

```
Это только локальная структура. Она не является Git monorepo.
```

```
Для управления множеством репозиториев можно использовать GitHub CLI (`gh`) и
отдельные скрипты запуска/обновления сервисов.
```

```
## 3. Архитектура Backend
```

```
### 3.1. Стандартный технологический стек
```

```
| Категория           | Технологии                      |
| ------------------- | ------------------------------- |
| Language            | Java 21                         |
| Framework           | Spring Boot                     |
| Build               | Gradle Kotlin DSL               |
| Gateway             | Spring Cloud Gateway            |
| Security            | Spring Security                 |
| Authentication      | OAuth2, OIDC, JWT               |
| Persistence         | Spring Data JPA, Hibernate      |
| Database            | PostgreSQL                      |
| NoSQL               | MongoDB                         |
| Cache               | Redis                           |
| Messaging           | Apache Kafka                    |
| Serialization       | Avro                            |
| Schema Registry     | Apicurio Registry               |
| CDC                 | Debezium                        |
| Search              | OpenSearch                      |
| Object Storage      | SeaweedFS                       |
| Resilience          | Resilience4j                    |
| Configuration       | Spring Cloud Config             |
| Secrets             | HashiCorp Vault                 |
| API Documentation   | Springdoc OpenAPI               |
| Testing             | JUnit 5, Mockito, AssertJ       |
| Integration Testing | Testcontainers                  |
| Metrics             | Prometheus                      |
| Dashboard           | Grafana                         |
| Logging             | Elasticsearch, Logstash, Kibana |
| Tracing             | OpenTelemetry, Jaeger           |
| Deployment          | Docker, Minikube                |
```

```
Почему SeaweedFS?
```

```
Это open-source альтернатива Amazon S3 для локального объектного хранения. Она
подходит для фотографий товаров, документов, изображений и других файлов.
```

```
Рекомендую использовать S3-compatible API, чтобы приложение не зависело от
конкретного object storage.
```

```
### 3.2. Структура отдельного микросервиса
```

```
Например, `emarket-order-service`:
```

```
emarket-order-service/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── security.yml
│       └── release.yml
│
├── .husky/
│   └── pre-commit
│
├── .editorconfig
├── .gitignore
```

```
├── .pre-commit-config.yaml
├── .commitlintrc.json
├── .java-version
│
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
├── gradlew.bat
│
├── Dockerfile
├── compose.yaml
│
├── docs/
│   ├── architecture.md
│   ├── api.md
│   ├── database.md
│   ├── events.md
│   └── decisions/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/emarket/order/
│   │   │       │
│   │   │       ├── OrderApplication.java
│   │   │       │
│   │   │       ├── api/
│   │   │       │   ├── controller/
│   │   │       │   ├── request/
│   │   │       │   ├── response/
│   │   │       │   └── mapper/
│   │   │       │
│   │   │       ├── application/
│   │   │       │   ├── command/
│   │   │       │   ├── query/
│   │   │       │   ├── service/
│   │   │       │   └── port/
│   │   │       │
│   │   │       ├── domain/
│   │   │       │   ├── model/
│   │   │       │   ├── event/
│   │   │       │   ├── repository/
│   │   │       │   └── exception/
│   │   │       │
│   │   │       ├── infrastructure/
│   │   │       │   ├── persistence/
│   │   │       │   ├── messaging/
│   │   │       │   ├── client/
│   │   │       │   ├── config/
│   │   │       │   └── outbox/
│   │   │       │
│   │   │       └── shared/
│   │   │
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-default.yml
│   │       ├── application-deploy.yml
│   │       └── db/migration/
│   │
│   └── test/
│       ├── java/
│       │   └── com/emarket/order/
│       │       ├── unit/
│       │       ├── integration/
```

```
│       │       ├── architecture/
│       │       └── contract/
│       │
│       └── resources/
│           ├── application-test.yml
│           └── fixtures/
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
│
└── README.md
```
```

```
Это рекомендуемый шаблон, который можно адаптировать под каждый микросервис.
```

```
### 3.3. Архитектурные слои
```

```
Рекомендую придерживаться Hexagonal Architecture / Ports and Adapters с
элементами DDD.
```

- ``api` — REST/GraphQL интерфейсы.` 

- ``application` — use cases, команды, запросы, orchestration.` 

- ``domain` — бизнес-правила и доменная модель.` 

- ``infrastructure` — PostgreSQL, Kafka, Redis, внешние клиенты.` 

```
Важный принцип: доменная логика не должна зависеть от Spring, Kafka или
PostgreSQL.
```

```
Не нужно делать один универсальный `common` repository с бизнес-классами для
всех сервисов. Это создаст скрытую связанность и усложнит независимые
deployment.
```

```
## 4. Database-per-Service
```

```
Каждый сервис самостоятельно владеет данными.
```

|`| Сервис`<br>`|`<br>`| ------------`<br>`---------------`<br>|`| Основная БД             | Назначение`<br>`| ----------------------- |`<br>`-------------------------------------- |`<br>|
|---|---|
|`| Identity`<br>`роли, refresh t`|`| PostgreSQL              | Пользователи для аутентификации,`<br>`okens |`|
|`| User`<br>`|`|`| PostgreSQL              | Профили, адреса, настройки`|
|`| Catalog`<br>`характеристики`<br>|`| PostgreSQL + MongoDB    | Товары, категории, динамические`<br>`|`<br>|
|`| Inventory`<br>`|`|`| PostgreSQL              | Остатки и резервы`|
|`| Cart`<br>`данные`|`| Redis + PostgreSQL      | Активная корзина и её сохранённые`<br>`|`|
|`| Order`<br>`|`|`| PostgreSQL              | Заказы, позиции, история покупок`|
|`| Payment`<br>`|`|`| PostgreSQL              | Платежи и транзакции`|
|`| Notification`<br>|`| MongoDB + PostgreSQL    | Шаблоны, история отправок`|
|`|`<br>`| Search`<br>|`| OpenSearch              | Поисковый индекс`|
|`|`<br>`| Analytics`<br>`|`|`| PostgreSQL / ClickHouse | Аналитические представления`|



```
Это логическое распределение. На первом этапе можно использовать один PostgreSQL
```

```
instance, но разные базы данных или схемы с отдельными credentials.
```

```
Важно: отдельная база данных не обязательно означает отдельный физический
сервер. Главное — независимое владение данными и отсутствие прямого доступа
одного сервиса к таблицам другого.
```

```
### История покупок — 5 лет
```

```
Для `emarket-order-service`:
```

```
- Сохранять заказ и все его позиции.
```

```
- Сохранять snapshot цены, валюты, названия товара и количества на момент
покупки.
```

```
- Не полагаться на текущие данные Catalog для отображения старых заказов.
- Использовать `created_at`, `completed_at`.
```

```
- Настроить retention policy не менее пяти лет.
```

```
- Не удалять историю заказов при удалении пользователя.
```

```
- Для больших объёмов предусмотреть partitioning по дате.
```

```
## 5. Default и Deploy режимы
```

```
Это одно из наиболее важных требований проекта.
```

```
Я рекомендую использовать Spring Profiles.
```

```
Default mode
```

```
Development
```

```
# Host machine
```

```
Все Java-сервисы запускаются непосредственно на Ubuntu, без Docker-контейнеров
для самих микросервисов.
```

```
./gradlew bootRun \
  --args='--spring.profiles.active=default'
```

```
Deploy mode
```

```
Deployment
```

```
# Docker + Minikube
```

```
Каждый микросервис собирается в Docker image и разворачивается независимо в
Kubernetes.
```

```
kubectl apply -f k8s/
```
### 5.1. Default mode
```

```
Ubuntu Host
│
├── emarket-api-gateway      :8080
├── emarket-bff-web          :8081
├── emarket-bff-mobile       :8082
├── emarket-identity         :8083
├── emarket-user             :8084
├── emarket-catalog          :8085
├── emarket-cart             :8086
```

```
├── emarket-order            :8087
│
└── Docker Compose (infrastructure only)
    ├── PostgreSQL
    ├── Redis
    ├── Kafka
    ├── MongoDB
    ├── OpenSearch
    └── SeaweedFS
```
```

```
Здесь Docker используется только для инфраструктурных зависимостей. Все Java-
приложения работают непосредственно на host.
```

```
### 5.2. Deploy mode
```

```
```
Minikube
│
├── namespace: emarket
│   ├── api-gateway
│   ├── bff-web
│   ├── bff-mobile
│   ├── identity-service
│   ├── user-service
│   ├── catalog-service
│   ├── cart-service
│   └── order-service
│
├── namespace: emarket-infrastructure
│   ├── kafka
│   ├── redis
│   ├── postgres
│   ├── mongodb
│   └── opensearch
│
└── namespace: observability
    ├── prometheus
    ├── grafana
    ├── elk
    └── jaeger
```
```

```
В Kubernetes:
```

- `Каждый сервис — отдельный Deployment.` 

- `Каждый сервис имеет собственный Service.` 

- `Конфигурации передаются через ConfigMap.` 

- `Секреты — через Vault.` 

- `Health checks — readiness, liveness, startup probes.` 

- `Ресурсы — CPU/memory requests и limits.` 

- `Gateway публикуется наружу через NodePort или Ingress.` 

- `BFF и остальные сервисы доступны только внутри кластера.` 

```
Для независимого deployment каждый сервис должен иметь собственный Docker image
и release pipeline.
```

```
### 5.3. Конфигурация Spring Cloud
```

```
В каждом сервисе:
```

```
```
spring:
  application:
```

```
    name: emarket-order-service
```

```
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:default}
```

```
  config:
    import: optional:configserver:${CONFIG_SERVER_URL:http://localhost:8888}
```
```

```
Примеры конфигураций в `emarket-config-repo`:
```

```
emarket-config-repo/
│
├── application.yml
├── application-default.yml
├── application-deploy.yml
│
├── emarket-api-gateway.yml
├── emarket-bff-web.yml
├── emarket-identity-service.yml
├── emarket-user-service.yml
├── emarket-catalog-service.yml
└── emarket-order-service.yml
```
```

```
Секреты не должны храниться в Git. Spring Cloud Config отвечает за конфигурацию,
Vault — за секреты.
```

```
## 6. Отдельный WireMock для Web и Mobile
```

```
Рекомендую оставить `emarket-api-mock` отдельным GitHub repository.
```

```
Он будет имитировать публичный контракт API Gateway, позволяя независимо
разрабатывать Web и Mobile, пока реальные backend-сервисы ещё не готовы.
```

```
Один WireMock — два независимых API-контракта
```

```
```
                  emarket-api-mock
                    WireMock 3
                    Port 9090
                         |
              +----------+----------+
              |                     |
           WEB API               MOBILE API
              |                     |
       /api/v1/web/**        /api/v1/mobile/**
              |                     |
        Web mappings          Mobile mappings
        Web responses         Mobile responses
```
```

```
Структура:
```

```
emarket-api-mock/
│
├── mappings/
│   ├── web/
│   │   ├── auth/
│   │   │   ├── register.json
│   │   │   ├── login.json
│   │   │   └── logout.json
```

```
│   │   ├── users/
│   │   ├── products/
│   │   └── errors/
│   │
│   └── mobile/
│       ├── auth/
│       ├── users/
│       ├── products/
│       └── errors/
│
├── __files/
│   ├── web/
│   │   ├── auth/
│   │   ├── users/
│   │   └── products/
│   │
│   └── mobile/
│       ├── auth/
│       ├── users/
│       └── products/
│
├── Dockerfile
├── compose.yaml
└── README.md
```
```

```
Например, один и тот же бизнес-сценарий может иметь разные ответы.
Web:
```

```
```
{
  "user": {
    "id": "1001",
    "firstName": "Alex",
    "lastName": "Smith"
  },
  "permissions": [
    "USER_READ",
    "ORDER_CREATE"
  ]
}
```
```

```
Mobile:
```
{
  "id": "1001",
  "name": "Alex Smith",
  "permissions": [
    "USER_READ",
    "ORDER_CREATE"
  ]
}
```
```

```
В результате можно тестировать разные DTO и контракты независимо.
```

```
### Переключение mock / real backend
```

```
Frontend:
```

```
VITE_API_MODE=mock
VITE_API_BASE_URL=http://localhost:9090/api/v1
```
```

```
Реальный backend:
```

```
VITE_API_MODE=real
VITE_API_BASE_URL=http://localhost:8080/api/v1
```
```

```
В production mock должен быть отключён.
```

```
## 7. Архитектура Frontend
```

```
Технологии:
```

```
| Категория            | Решение                   |
| -------------------- | ------------------------- |
| Framework            | React                     |
| Language             | TypeScript                |
| Build                | Vite                      |
| Routing              | React Router              |
| State                | Zustand                   |
| Server State         | TanStack Query            |
| Forms                | React Hook Form           |
| Validation           | Zod                       |
| Styling              | Tailwind CSS              |
| Components           | shadcn/ui                 |
| Internationalization | i18next                   |
| Testing              | Vitest                    |
| Component Testing    | React Testing Library     |
| E2E                  | Playwright                |
| API                  | OpenAPI generated clients |
| Lint                 | ESLint                    |
| Formatting           | Prettier                  |
```

```
### 7.1. Soft Modern UI
```

```
[E-commerce Custom Product Page by Ronas IT | UI/UX Team on
Dribbble](https://images.openai.com/static-rsc-4/nRotIcHs2jq5nVWvDA-
kJZ0nWHg9Z5lyeNQH4O5Tfa-YSyY_eFpdCKQZXqq3EimLkahFMaWEOJJUfxeoIJ7-
LV_i5kXjdVGG5XnVttWvbEuMWqMLZslyzP5oS5f6Lvo5WNDyI-OBL5OxJ2VaOOBXMW-
fCQRuvz9r1KG4feGAhTs?purpose=inline)
```

```
[Storeshop - Dashboard by Dipa Product for Dipa Inhouse on
Dribbble](https://images.openai.com/static-rsc-4/tT0jGjKK_qcj8CjiKK2dkuz5hSBonNI
rP1RxSVPyX5kUDK7v3_p68ytAUKHB9MJQ0lnWSf5kGT-V4FqJXlw_YFTuRQ8Kmet7K-
oi9Y8zXlkYtIeFdni7HQQT3Wm6GflFkZzWgqtivVpmMF-cdmzQ5vk5xos4FQNOSdavLcfyVZo?
purpose=inline)
```

```
[E-commerce Admin Panel Dashboard UI by Eljan Huseynaliyev on Dribbble]
(https://images.openai.com/static-rsc-4/r6oJPN7VpySJRXJTmN2xdzaCCRFsIgoauXzcU99D
tpIy3RVXdsZx3mpODxDEx0uskiTINdHfN4cFn4QPI3D5ybd2GiqaRIRvH-
MGd0B9bu74U1uawgW1HgkY5HIMzpXEtQP-riO5qv1L9p3yBQ24kDn8SVRY3vSH7KygB-0EuAo?
purpose=inline)
```

```
12
```

```
Предлагаю единый визуальный стиль:
```

- `Светлый и тёмный режим.` 

- `Мягкие нейтральные цвета.` 

- `Скруглённые карточки.` 

- `Ненавязчивые тени.` 

- `Простая типографика.` 

- `Адаптивный интерфейс.` 

- `Единая дизайн-система.` 

- `Раздельные пользовательский и административный layout.` 

```
Figma желательно организовать по страницам:
```

`1. Design System.` 

`2. Customer Web.` 

```
3. Admin Dashboard.
```

`4. Mobile.` 

`5. Dark Theme.` 

```
6. Components and States.
```

```
### 7.2. Production-ready frontend structure
Для `emarket-web`:
```
emarket-web/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── public/
│   ├── locales/
│   │   ├── en/
│   │   ├── ru/
│   │   └── bg/
│   └── images/
│
├── src/
│   ├── app/
│   │   ├── providers/
│   │   ├── router/
│   │   ├── layouts/
│   │   └── config/
│   │
│   ├── assets/
│   │
│   ├── components/
│   │   ├── ui/
│   │   ├── common/
│   │   └── feedback/
│   │
│   ├── features/
│   │   ├── auth/
│   │   ├── profile/
│   │   ├── catalog/
│   │   ├── cart/
│   │   ├── checkout/
│   │   ├── orders/
│   │   ├── payments/
│   │   └── reviews/
│   │
│   ├── pages/
│   │   ├── customer/
│   │   └── admin/
│   │
│   ├── services/
│   │   ├── api/
```

```
│   │   ├── mock/
│   │   └── generated/
│   │
│   ├── hooks/
│   ├── store/
│   ├── i18n/
│   ├── themes/
│   ├── types/
│   ├── utils/
│   ├── test/
│   │
│   ├── App.tsx
│   └── main.tsx
│
├── e2e/
│   ├── auth/
│   ├── catalog/
│   ├── checkout/
│   └── admin/
│
├── .env.example
├── .prettierrc
├── .editorconfig
├── eslint.config.js
├── package.json
├── vite.config.ts
├── tsconfig.json
└── README.md
```
```

```
Рекомендую feature-based архитектуру: компоненты, API, hooks и типы конкретной
функциональности находятся рядом.
```

```
Для мультиязычности начнём с английского, русского и болгарского. Все тексты
должны храниться в translation files, а не непосредственно в компонентах.
```

```
Для авторизации Web лучше использовать BFF с серверной сессией и `HttpOnly`,
`Secure`, `SameSite` cookie. Это безопаснее, чем хранить долгоживущие JWT в
`localStorage`.
```

```
Административные права должны проверяться на backend, а не только скрытием
кнопок на frontend.
```

```
## 8. Постепенное внедрение микросервисов — Roadmap
```

```
Я бы разбил разработку на 12 этапов. Каждый этап должен давать работающий
результат, который можно продемонстрировать на техническом собеседовании.
```

```
Не стоит начинать с Kafka, Debezium, CQRS и Kubernetes. Сначала нужно научиться
правильно проектировать границы сервисов, контракты и независимые deployment.
```

- `# План разработки` 

`1. Этап 0. Engineering Foundation` 

   - `Начать здесь` 

```
   GitHub Organization, repositories, правила веток, Conventional Commits, Java
formatter, frontend formatter, CI templates, документация, Docker и Minikube.
2. Этап 1. Frontend + API Mock
```

```
   React, TypeScript, Vite, Soft Modern UI, Figma, темы, i18n, WireMock
Web/Mobile, пользовательский и административный layout.
3. Этап 2. Gateway + BFF
```

```
   Spring Cloud Gateway, BFF Web, BFF Mobile, маршрутизация, единый API
contract, rate limiting, CORS, API documentation.
4. Этап 3. Authentication + User
```

```
   Identity Service, User Service, PostgreSQL, Spring Security, OAuth2/OIDC,
JWT, refresh token, RBAC, Flyway.
5. Этап 4. Catalog + Object Storage
```

```
   Catalog Service, PostgreSQL, MongoDB, SeaweedFS, товары, категории,
динамические характеристики, фотографии.
```

```
6. Этап 5. Cart + Redis
```

```
   Cart Service, Redis, TTL, caching, optimistic locking, работа с корзиной.
7. Этап 6. Inventory + Order
```

```
   Inventory Service, Order Service, Database-per-Service, транзакции,
резервирование, Saga, Transaction Outbox.
```

```
8. Этап 7. Payment
```

```
   Payment Service, Fake Payment Provider, идемпотентность, Retry, Circuit
Breaker, обработка ошибок оплаты.
```

```
9. Этап 8. Kafka + Event-driven Architecture
```

```
   Kafka, Avro, Schema Registry, Debezium CDC, Outbox relay, асинхронная
коммуникация.
```

```
10. Этап 9. Notifications + Search
```

```
    Notification Service, Email templates, in-app notifications, OpenSearch,
полнотекстовый поиск.
11. Этап 10. Admin + CQRS + Analytics
```

```
    Admin Service, CQRS, read models, история покупок, отчёты, генератор
тестовых данных.
12. Этап 11. Production Infrastructure
```

```
    Spring Cloud Config, Vault, Docker, Minikube, Kubernetes, Prometheus,
Grafana, ELK, Jaeger, alerts.
13. Этап 12. Production Readiness
```

```
    Нагрузочные тесты, security scanning, chaos testing, recovery,
backup/restore, документация, демонстрационный deployment.
```

```
### Подробности по этапам
```

```
#### Этап 0. Engineering Foundation
```

```
Создать GitHub Organization и базовые репозитории.
```

```
Результат:
```

- `Все репозитории созданы в Organization.` 

- `Настроены `main`, `develop` и feature branches.` 

- `GitHub Actions выполняет базовые проверки.` 

- `Java formatter и frontend formatter работают локально.` 

- `Conventional Commits проверяются автоматически.` 

- `Создана документация архитектуры.` 

- `Подготовлен Docker Compose для инфраструктуры.` 

- `Minikube запускается и доступен.` 

```
Здесь же создаём стандартные шаблоны для всех будущих Java-сервисов.
```

```
#### Этап 1. Frontend + WireMock
```

```
Первый полноценный пользовательский результат.
```

```
Функциональность:
```

```
- Главная страница.
```

- `Регистрация.` 

- `Login / Logout.` 

- `User Profile.` 

- `Admin Login.` 

- `Admin Dashboard.` 

- `Light / Dark theme. - EN / RU / BG.` 

- `API Mock.` 

- `Независимые Web/Mobile responses.` 

```
Пока реальные backend-сервисы не нужны. Frontend разрабатывается на основе mock
API.
#### Этап 2. API Gateway + BFF
Создаём:
`emarket-api-gateway`
`emarket-bff-web`
`emarket-bff-mobile`
Основные темы:
- Spring Cloud Gateway.
- Routing.
- Filters.
- Global filters.
- Rate limiting.
- CORS.
- Request correlation ID.
- BFF aggregation.
- REST API.
- OpenAPI / Swagger.
- Error handling.
- API versioning.
Gateway не должен содержать бизнес-логику.
Пример:
```
spring:
  cloud:
    gateway:
      routes:
        - id: web-bff
          uri: ${WEB_BFF_URL}
          predicates:
            - Path=/api/v1/web/**
        - id: mobile-bff
          uri: ${MOBILE_BFF_URL}
          predicates:
            - Path=/api/v1/mobile/**
```
```

```
#### Этап 3. Identity + User
```

```
Это первые настоящие бизнес-сервисы.
```

```
Identity Service отвечает за:
```

- `Registration.` 

- `Login.` 

- `Logout.` 

- `JWT access token.` 

- `Refresh token rotation.` 

- `Password hashing.` 

- `OAuth2/OIDC.` 

- `Role-based authorization.` 

```
User Service:
```

```
- Profile.
- Addresses.
- Preferences.
- User settings.
```

```
Роли:
```

```
```
ROLE_CUSTOMER
ROLE_ADMIN
ROLE_SUPER_ADMIN
```
```

```
В этом этапе важно изучить:
```

- `Spring Security Filter Chain.` 

- `Authentication Provider.` 

- `JWT validation.` 

- `Method Security.` 

- `PasswordEncoder.` 

- `Refresh token revocation.` 

- `CORS и CSRF.` 

```
#### Этап 4. Catalog + Object Storage
```

```
Товары должны иметь как фиксированные, так и динамические характеристики.
Например:
```

```
```
{
  "name": "Laptop",
  "category": "Electronics",
  "attributes": {
    "ram": "32GB",
    "storage": "1TB",
    "processor": "Intel i7"
  }
}
```
```

```
Здесь MongoDB полезна для гибкой структуры характеристик, но основную товарную
информацию, цены и связи лучше хранить в PostgreSQL.
```

```
Фотографии товаров — SeaweedFS.
```

```
#### Этап 5. Cart + Redis
```

```
Функциональность:
```

- `Add to cart.` 

- `Remove from cart.` 

- `Update quantity.` 

- `Calculate total.` 

- `Cart expiration.` 

- `Restore cart.` 

- `Concurrent cart updates.` 

```
Здесь можно изучить Redis TTL, cache invalidation, optimistic locking и
распределённые блокировки.
```

```
Не стоит использовать Redis как единственное постоянное хранилище корзины, если
требуется восстановление после истечения срока жизни ключа.
```

```
#### Этап 6. Inventory + Order
```

```
Это один из самых важных этапов для Senior Java.
```

```
Сценарий покупки:
```

```
```
Customer
   |
   v
Order Service
   |
   v
Inventory Service
   |
   v
Reserve Product
   |
   v
Create Order
   |
   v
Persist Order + Outbox Event
```
```

```
Внедряем:
```

- `Database-per-Service.` 

- `Transaction Outbox.` 

- `Saga Pattern.` 

- `Idempotency.` 

- `Optimistic locking.` 

- `Distributed transaction alternatives.` 

- `Order state machine.` 

- `Inventory reservation.` 

- `Eventual consistency.` 

```
Важно понимать, что обычная транзакция PostgreSQL не может атомарно охватить
базы двух микросервисов.
```

```
#### Этап 7. Payment
```

```
Fake Payment Provider должен имитировать:
```

- `Успешную оплату.` 

- `Отклонённую оплату.` 

- `Timeout.` 

- `Временную недоступность.` 

```
- Повторное получение результата.
```

```
- Дублирование callback.
```

```
Пример:
```

```
```
Order Service
     |
     v
Payment Service
     |
     v
Fake Payment Provider
     |
     +---- SUCCESS
     |
     +---- FAILED
     |
     +---- TIMEOUT
```
```

```
Изучаем:
```

```
- Circuit Breaker.
- Retry.
- Timeout.
- Bulkhead.
- Idempotency key.
- Payment state transitions.
- Compensation transactions.
```

```
Retry нельзя применять безусловно к операциям оплаты: повтор должен быть
безопасным и учитывать идемпотентность.
```

```
#### Этап 8. Kafka + CDC
```

```
После того как REST-коммуникация и Outbox уже понятны, переходим к Event-driven
Architecture.
```

```
Пример:
```

```
```
Order Service
     |
     v
PostgreSQL
     |
     v
Outbox Table
     |
     v
Debezium CDC
     |
     v
Kafka
     |
     +---- Inventory Service
     |
     +---- Payment Service
     |
     +---- Notification Service
     |
     +---- Analytics Service
```
```

```
Kafka topics:
```

```
```
order.created
order.confirmed
order.cancelled
inventory.reserved
inventory.released
payment.completed
payment.failed
notification.requested
```
```

```
Используем Avro и Schema Registry.
```

```
Изучаем:
```

- `Consumer groups.` 

- `Partitioning.` 

- `Offset management.` 

- `At-least-once delivery.` 

- `Idempotent consumers.` 

- `Schema evolution.` 

- `Dead Letter Queue.` 

- `Retry topics.` 

- `Outbox pattern.` 

- `CDC.` 

```
#### Этап 9. Notifications + Search
```

```
Notification Service:
```

- `Email templates.` 

- `In-app notifications.` 

- `Notification preferences.` 

- `Retry failed delivery.` 

- `Notification history.` 

```
Search Service:
```

- `OpenSearch.` 

- `Product indexing.` 

- `Full-text search.` 

- `Filtering.` 

- `Sorting.` 

- `Faceted search.` 

- `Search suggestions.` 

```
Kafka позволяет синхронизировать поисковый индекс с Catalog Service без прямого
доступа к его БД.
```

```
#### Этап 10. CQRS + Admin
```

```
На этом этапе появятся полноценные административные функции.
```

```
Пример:
```

```
```
                 Order Service
                      |
```

```
                      v
                   Kafka
                      |
                      v
                 Admin Read Model
                      |
                      v
                Admin Dashboard
```
```

```
CQRS:
```

- `Command Model — операции изменения.` 

- `Query Model — оптимизированные представления.` 

- `Read Model обновляется асинхронно.` 

- `Отдельные запросы для аналитики.` 

```
Admin Service не должен становиться универсальным владельцем бизнес-данных
других сервисов. Он может агрегировать представления и координировать
административные use cases.
```

```
#### Этап 11–12. Production Readiness
```

```
Здесь внедряем полную эксплуатационную инфраструктуру:
```

- `Config Server.` 

- `Vault. - Prometheus.` 

- `Grafana.` 

- `ELK.` 

- `Jaeger.` 

- `Alertmanager. - Kubernetes probes.` 

- `Resource limits.` 

- `Autoscaling.` 

- `Network policies.` 

- `Backups. - Disaster recovery. - Security scans. - Load testing.` 

```
## 9. Независимый deployment и версии сервисов
```

```
Важно, чтобы изменение `emarket-order-service` не заставляло пересобирать все
остальные сервисы.
```

```
Для каждого репозитория:
```

```
Developer pushes code
        |
        v
GitHub Actions
        |
        v
Compile + Tests
        |
        v
Quality + Security
        |
        v
Build Docker Image
        |
        v
```

```
Publish Image
        |
        v
Deploy Only This Service
```
```

```
Например:
```

```
```
emarket-order-service:1.2.0
emarket-catalog-service:2.1.0
emarket-payment-service:1.0.5
```
```

```
Сервисы должны поддерживать обратную совместимость API и событий. При изменении
контрактов используем versioning и contract tests.
```

```
Не следует создавать единый release, требующий синхронного обновления всех
микросервисов.
```

```
## 10. Обязательные CI-проверки
```

```
Я рекомендую разделить CI на три уровня: обязательные проверки каждого pull
request, проверки release и deployment.
```

```
### 10.1. Backend CI
```

```
Для каждого Java repository:
```

|`| Проверка`<br>`|`|`| Инструмент             | Обязательность`|
|---|---|
|`| --------------------------`<br>`-----------------------------`<br>|`| ---------------------- |`<br>`- |`<br>|
|`| Compilation`<br>`|`|`| Gradle                 | Да`|
|`| Java formatting`<br>`|`|`| Spotless               | Да`|
|`| Unit tests`<br>`|`|`| JUnit 5                | Да`|
|`| Assertions`<br>`|`|`| AssertJ                | Да`|
|`| Mocking`<br>`|`|`| Mockito                | Да`|
|`| Integration tests`<br>`|`|`| Testcontainers         | Да`|
|`| Architecture tests`<br>`|`|`| ArchUnit               | Да`|
|`| Code coverage`<br>`|`|`| JaCoCo                 | Да`|
|`| Static analysis`<br>`|`|`| SonarQube              | Да`|
|`| Dependency vulnerabilities`<br>`|`|`| OWASP Dependency-Check | Да`|
|`| Secret scanning`<br>`|`|`| GitHub Secret Scanning | Да`|
|`| Docker build`<br>`|`|`| Docker                 | Да`|
|`| OpenAPI validation`<br>`|`|`| OpenAPI validator      | Для API-сервисов`|
|`| Contract tests`<br>`сервисов |`|`| Spring Cloud Contract  | Для взаимодействующих`|



```
Рекомендую минимальный coverage gate 80% для нового кода, с отдельным контролем
критической бизнес-логики. Не нужно искусственно покрывать простые getters ради
```

```
процента.
```

```
### 10.2. Frontend CI
```

```
| Проверка        | Инструмент            |
| --------------- | --------------------- |
| Type checking   | TypeScript            |
| Lint            | ESLint                |
| Formatting      | Prettier              |
| Unit tests      | Vitest                |
| Component tests | React Testing Library |
| Coverage        | Vitest                |
| E2E tests       | Playwright            |
| Dependency scan | Dependabot            |
| Build           | Vite                  |
| Accessibility   | axe-core              |
| Secret scan     | GitHub                |
```

```
### 10.3. Infrastructure CI
```

```
- Dockerfile lint — Hadolint.
- Docker Compose validation.
- Kubernetes manifests validation.
- Helm lint, если используем Helm.
- YAML lint.
- Trivy image scanning.
- Kubernetes security checks.
- Terraform validation, если позже добавим Terraform.
- Проверка конфигураций и отсутствия секретов.
### 10.4. Пример Backend GitHub Actions
Файл `.github/workflows/ci.yml`:
```
name: Backend CI
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]
permissions:
  contents: read
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Java 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
      - name: Verify formatting
```

- `run: ./gradlew spotlessCheck - name: Compile run: ./gradlew compileJava` 

- `name: Run unit tests run: ./gradlew test` 

- `name: Run integration tests run: ./gradlew integrationTest` 

```
      - name: Check architecture
        run: ./gradlew test
      - name: Build application
        run: ./gradlew build
      - name: Upload test reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-reports
          path: build/reports/
```

```
В реальном проекте стоит разделить unit и integration test задачи в Gradle, а
также использовать отдельные jobs для security scanning, coverage и Docker
build.
```

```
Версии GitHub Actions следует фиксировать на проверенные версии или commit SHA,
а не бесконтрольно обновлять.
```

```
## 11. Защита GitHub main
```

```
Для всех репозиториев рекомендую одинаковые правила.
```

```
Branch protection: main
```

```
Required
```

```
Require a pull request before merging
```

```
Require at least 1 approval
```

```
Require status checks to pass
```

```
Require conversation resolution
```

```
Require branches to be up to date
```

```
Block force pushes
```

```
Block branch deletion
```

```
Restrict direct pushes
```

- `Дополнительно:` 

- `Require linear history.` 

- `Запретить обход branch protection.` 

- `Включить Dependabot.` 

- `Включить secret scanning и push protection, если доступны. - Использовать squash merge.` 

- `Для `main` разрешать только проверенные PR.` 

```
Для личного проекта можно установить минимум один approval, но если ты
единственный участник, нужно учитывать, что сам себе approval дать нельзя. Для
такого сценария используй отдельную настройку обязательных CI checks и PR без
обязательного approval либо второго reviewer.
```

```
## 12. Conventional Commits
```

```
Используем стандарт:
```

```
`<type>(<scope>): <description>`
```

```
Основные типы:
```

```
| Type       | Назначение                                      |
| ---------- | ----------------------------------------------- |
| `feat`     | Новая функциональность                          |
| `fix`      | Исправление ошибки                              |
| `refactor` | Рефакторинг                                     |
| `test`     | Тесты                                           |
| `docs`     | Документация                                    |
| `build`    | Сборка                                          |
| `ci`       | CI/CD                                           |
| `chore`    | Технические задачи                              |
| `perf`     | Производительность                              |
| `style`    | Изменения форматирования, не влияющие на логику |
```

```
Примеры:
```

```
feat(auth): implement JWT authentication
feat(order): add order creation endpoint
fix(payment): handle provider timeout
refactor(catalog): extract product mapper
test(order): add integration tests
docs(architecture): add C4 diagrams
ci(backend): add SonarQube analysis
build(backend): upgrade Spring Boot
perf(catalog): optimize product search
```
```

```
Для несовместимых изменений:
```

```
feat(api)!: change authentication contract
```
```

```
Или:
```

```
feat(api): change authentication contract
BREAKING CHANGE: refresh token response structure changed
```
```

```
### 12.1. Автоформатирование Java через pre-commit
```

```
Рекомендую использовать:
```

- `Spotless Gradle Plugin.` 

- `Google Java Format.` 

- `pre-commit framework.` 

- `Commitlint для проверки commit message.` 

```
В `build.gradle.kts`:
```
plugins {
    java
    id("com.diffplug.spotless") version "7.0.2"
}
spotless {
    java {
        googleJavaFormat()
        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
    }
}
```
В `Makefile`:
```
format:
./gradlew spotlessApply
format-check:
./gradlew spotlessCheck
```
`.pre-commit-config.yaml`:
```
repos:
  - repo: local
    hooks:
      - id: java-format
        name: Java auto-format
        entry: ./gradlew spotlessApply
        language: system
        pass_filenames: false
        always_run: true
        stages: [pre-commit]
```
Установка:
```
pip install pre-commit
pre-commit install
pre-commit install --hook-type commit-msg
```
```

```
Важный нюанс: если Spotless изменил Java-файлы, Git не должен автоматически
включать посторонние изменения в commit. После форматирования проверь изменения
и добавь исправленные файлы в staging. Если hook изменил файлы, commit лучше
остановить и повторить после staging.
```

```
### 12.2. Commit message validation
```

```
Пример `.commitlintrc.json`:
```

```
```
{
  "extends": [
```

```
    "@commitlint/config-conventional"
  ]
}
```
```

```
Commitlint должен проверять сообщение на `commit-msg` hook, а CI должен
выполнять дополнительную проверку PR title или commit messages.
```

```
Для frontend:
```

```
- ESLint.
- Prettier.
- lint-staged.
- Husky.
```

```
Для Java:
```

```
- Spotless.
- pre-commit.
```

```
Таким образом, каждый репозиторий имеет одинаковые правила качества, но
использует подходящий инструмент для своего языка.
```

```
## 13. Observability, monitoring и logging
```

```
Все микросервисы должны поддерживать единый подход к observability.
```

```
[🔎 Observability in Microservices with OpenTelemetry (Complete Flow): | N
Sivakumar](https://images.openai.com/static-rsc-4/Wr82vAhPFnsD43c2TF_rW8YtAlTiil
NXWxJkiNrGgF4tWojr8Fl0tJJn0yMbU5arywsVDlRWzJwObcxCZ33_gwPgEFDUsEIxZ7BhYmW08BPSxb
yEDnKzWgpgh2uhG_qw2p-3N1CmTplm7uA405vSGepd1O6r97W0SjBgRdF_n1A?purpose=inline)
```

```
[linkedin.com](https://www.linkedin.com/posts/sivakumarnaguri_kubernetes-
microservices-opentelemetry-activity-7370334529422741504-DR4B?
utm_source=chatgpt.com)
```

```
### Архитектура
```

```
       All Microservices
              |
     +--------+--------+
     |        |        |
  Metrics   Logs    Traces
     |        |        |
     v        v        v
Prometheus  Logstash OpenTelemetry
     |        |        |
     v        v        v
  Grafana Elasticsearch Jaeger
              |
            Kibana
```
```

```
Распределение ответственности:
```

```
| Инструмент    | Назначение                        |
| ------------- | --------------------------------- |
| Actuator      | Health, metrics, application info |
| Micrometer    | Сбор метрик                       |
| Prometheus    | Хранение метрик                   |
| Grafana       | Dashboard                         |
| Alertmanager  | Уведомления об ошибках            |
| Logstash      | Обработка логов                   |
```

```
| Elasticsearch | Хранение логов                    |
| Kibana        | Поиск и анализ логов              |
| OpenTelemetry | Instrumentation                   |
| Jaeger        | Distributed tracing               |
```

```
### Обязательные метрики
```

```
Для каждого сервиса:
```

```
- HTTP request count.
```

```
- HTTP error rate.
- Request latency p50/p95/p99.
- JVM heap.
- GC pauses.
```

```
- Thread count.
- CPU / Memory.
- Database connection pool.
- Kafka consumer lag.
- Retry count.
- Circuit breaker state.
```

```
Для бизнес-метрик:
```

```
- Orders created.
```

```
- Payments successful/failed.
```

```
- Inventory reservations.
```

```
- Checkout conversion.
```

```
- Notification delivery failures.
```

```
### Distributed tracing
Пример:
```

```
```
Web Request
    |
    v
API Gateway
    |
    v
BFF Web
    |
    v
Order Service
    |
    v
Inventory Service
    |
    v
Payment Service
```
```

```
Все операции должны иметь единый trace context.
```

```
Так в Jaeger можно будет увидеть, какой именно микросервис увеличивает время
обработки заказа.
```

```
### Alerts
Примеры:
```

```
| Alert                 | Условие                      |
| --------------------- | ---------------------------- |
| High HTTP error rate  | 5xx выше заданного порога    |
```

```
| High latency          | p95 выше SLO                 |
| Kafka consumer lag    | Lag превышает порог          |
| Database unavailable  | DB health DOWN               |
| Payment failure spike | Резкий рост failed payments  |
| High JVM memory       | Heap utilization выше порога |
```

```
## 14. Как покрыть остальные Senior Java темы
```

```
Здесь важно не просто добавить технологии, а создать конкретные бизнес-сценарии,
в которых их применение оправдано.
```

```
| Тема                 | Где реализовать                         |
| -------------------- | --------------------------------------- |
| API Gateway          | `emarket-api-gateway`                   |
| BFF                  | `emarket-bff-web`, `emarket-bff-mobile` |
| Circuit Breaker      | Payment → Fake Provider                 |
| Retry                | Payment, Notification                   |
| Outbox               | Order, Payment                          |
| CQRS                 | Order, Admin                            |
| Database-per-Service | Все бизнес-сервисы                      |
| Saga                 | Checkout / Order                        |
| Kafka                | Order, Inventory, Payment               |
| Avro                 | Kafka event contracts                   |
| Schema Registry      | Event compatibility                     |
| Debezium             | PostgreSQL Outbox CDC                   |
| MongoDB              | Dynamic product attributes              |
| Redis                | Cart, caching                           |
| OpenSearch           | Product search                          |
| SeaweedFS            | Product images                          |
| Vault                | Credentials, signing secrets            |
| Spring Cloud Config  | Centralized configuration               |
| Spring Security      | Identity, service authorization         |
| JWT                  | Access tokens                           |
| OAuth2/OIDC          | Authentication                          |
| Hibernate            | PostgreSQL persistence                  |
| GraphQL              | Catalog query API                       |
| REST                 | Основной межсервисный API               |
| Testcontainers       | Integration tests                       |
| WireMock             | External API simulation                 |
| Jaeger               | Distributed tracing                     |
| ELK                  | Centralized logging                     |
| Prometheus           | Metrics                                 |
| Grafana              | Monitoring                              |
| Alertmanager         | Alerts                                  |
```

# `### GraphQL` 

```
Не рекомендую внедрять GraphQL одновременно во все сервисы.
```

```
Лучше выделить его для Catalog:
```

```
```
Web
 |
 v
API Gateway
 |
 v
BFF Web
 |
 v
Catalog GraphQL API
```
```

# `Пример запроса:` 

```
query {
  products(category: "electronics") {
    id
    name
    price
    attributes
  }
}
```

```
REST при этом останется основным интерфейсом для большинства операций.
```

```
### Генератор тестовых данных
```

```
Добавим отдельный инструмент `emarket-test-data-generator`.
```

```
Он должен уметь создавать:
```

- `1 000–100 000 пользователей.` 

- `Тестовые товары и категории.` 

- `Случайные остатки.` 

- `Корзины.` 

- `Заказы с различными статусами.` 

- `Успешные и неуспешные платежи.` 

- `Историю заказов за пять лет.` 

- `Данные для нагрузочного тестирования.` 

```
Рекомендую Java + Spring Shell или отдельный CLI-инструмент.
```

```
Для нагрузочного тестирования можно использовать k6.
```

```
### Fake Payment Provider
```

```
Предлагаю выделить его в отдельный репозиторий:
```

```
`emarket-fake-payment-provider`.
```

```
Он имитирует внешний платёжный API, а Payment Service обращается к нему по HTTP.
```

```
Это позволит отдельно тестировать:
```

- `Timeout.` 

- `HTTP 400/500.` 

- `Slow responses.` 

- `Duplicate callbacks.` 

- `Payment status polling.` 

- `Webhook delivery.` 

```
В production реальный провайдер будет подключаться через адаптер, а Fake
Provider останется инструментом разработки и тестирования.
```

```
## 15. Дополненная структура репозиториев
```

```
Чтобы не смешивать бизнес-сервисы с инфраструктурными инструментами, добавим ещё
два репозитория:
```

- `| Repository                      | Назначение | | ------------------------------- | ----------------------------------------------- |` 

- `| `emarket-fake-payment-provider` | Независимая имитация внешнего платёжного API` 

```
|
```

```
| `emarket-test-data-generator`   | CLI для генерации тестовых и нагрузочных
данных |
```

```
Итоговая структура — 26 независимых GitHub repositories.
```

```
Это целевая структура, а не список задач для одновременного создания.
```

```
## 16. Документация
```

```
`emarket-docs` должен быть центральным местом архитектурной документации.
Структура:
```

```
emarket-docs/
│
├── architecture/
│   ├── overview.md
│   ├── system-context.md
│   ├── containers.md
│   ├── components.md
│   ├── deployment.md
│   ├── security.md
│   └── scalability.md
│
├── diagrams/
│   ├── c4/
│   ├── sequence/
│   ├── database/
│   ├── deployment/
│   └── events/
│
├── adr/
│   ├── ADR-001-microservices.md
│   ├── ADR-002-database-per-service.md
│   ├── ADR-003-api-gateway.md
│   ├── ADR-004-bff.md
│   ├── ADR-005-kafka.md
│   └── ADR-006-outbox.md
│
├── api/
│   ├── gateway/
│   ├── web/
│   └── mobile/
│
├── development/
│   ├── getting-started.md
│   ├── default-mode.md
│   ├── deploy-mode.md
│   ├── coding-standards.md
│   └── git-conventions.md
│
├── operations/
│   ├── monitoring.md
│   ├── logging.md
│   ├── troubleshooting.md
│   └── disaster-recovery.md
│
└── README.md
```
```

```
Обязательно документировать не только архитектуру, но и причины принятия
решений.
```

```
Например:
```

```
ADR-002: Database-per-Service
```

- `Context: микросервисы должны независимо развиваться.` 

- `Decision: каждый сервис владеет собственной БД. - Consequences: межсервисные запросы выполняются через API или события.` 

- `Alternatives: Shared Database.` 

```
Это особенно полезно на Senior-собеседовании, где важно объяснять не только как,
но и почему.
```

```
## 17. Порядок создания репозиториев
```

```
Не рекомендую сразу создавать 26 пустых репозиториев.
```

```
Лучше двигаться следующими волнами.
```

- `| Волна | Репозитории | | ----- | ---------------------------------------------------------------------------- | | 1     | `emarket-docs`, `emarket-infrastructure` | | 2     | `emarket-web`, `emarket-api-mock` | | 3     | `emarket-api-gateway`, `emarket-bff-web`, `emarket-bff-mobile` | | 4     | `emarket-identity-service`, `emarket-user-service` | | 5     | `emarket-catalog-service` | | 6     | `emarket-cart-service`, `emarket-inventory-service`, `emarket-orderservice` | | 7     | `emarket-payment-service`, `emarket-fake-payment-provider` | | 8     | `emarket-config-server`, `emarket-config-repo` | | 9     | `emarket-notification-service`, `emarket-pricing-service` | | 10    | `emarket-search-service`, `emarket-review-service` | | 11    | `emarket-admin-service`, `emarket-analytics-service` | | 12    | `emarket-observability`, `emarket-test-data-generator` | | 13    | `emarket-mobile` |` 

```
`emarket-mobile` можно начинать параллельно с Web, но полноценную интеграцию с
Mobile BFF лучше проводить после стабилизации общих контрактов.
```

```
## 18. Что должно быть готово после первого MVP
```

```
Не нужно ждать завершения всех 26 репозиториев, чтобы получить работающий
проект.
```

```
Я предлагаю определить первый MVP следующим образом.
```

```
MVP — Customer Authentication Platform
```

```
Первый milestone
```

```
[Lumiere Skin –  Cosmetics HTML Template |
```

```
Codester](https://images.openai.com/static-rsc-4/DZr9J__GFOXrY8UQDVbo5bLmLs_juI7
rAE6_KGxu74mvfmIi4NLKVgPaowy8-
WTTjsFirdHqUidRaoXKMccrqgijb83p62HciVjXiauclyXQQF885e3PHiYnx_fA_9zYU8D17Qcix1zP9
dwXKGcBiHfi4dkvIhI2sfYo-DPyNGU?purpose=inline)
```

```
[codester.com](https://www.codester.com/items/63023/lumiere-skin-cosmetics-html-
template?utm_source=chatgpt.com)
```

```
Репозитории:
```

- ``emarket-web`` 

- ``emarket-api-mock`` 

- ``emarket-api-gateway`` 

- ``emarket-bff-web`` 

- ``emarket-bff-mobile`` 

- ``emarket-identity-service`` 

- ``emarket-user-service`` 

- ``emarket-infrastructure`` 

- ``emarket-docs`` 

- ``emarket-config-server`` 

- ``emarket-config-repo`` 

```
Работающая функциональность:
```

- `Регистрация пользователя.` 

- `Login / Logout.` 

- `JWT authentication.` 

- `Refresh token.` 

- `User profile.` 

- `Административная авторизация.` 

- `RBAC. - Light / Dark theme. - EN / RU / BG. - Swagger.` 

```
- WireMock Web / Mobile.
```

- `PostgreSQL.` 

- `Docker deployment.` 

- `Minikube deployment.` 

- `CI для каждого репозитория.` 

```
### Критерии завершения MVP
```

`1. Web работает в `default` и `deploy`.` 

`2. Web обращается только к Gateway.` 

`3. Gateway маршрутизирует запросы в Web BFF.` 

`4. Web BFF использует Identity и User Service.` 

`5. Mobile BFF имеет независимый API-контракт.` 

`6. WireMock независимо возвращает Web и Mobile responses.` 

`7. Сервисы используют собственные базы данных.` 

`8. CI успешно проходит для каждого репозитория.` 

`9. `main` защищена от прямого push.` 

`10. Java автоматически форматируется pre-commit. 11. Commit messages проверяются Conventional Commits. 12. Каждый сервис можно развернуть независимо.` 

```
## 19. Как этот проект подготовит к Senior Java Fullstack Interview
```

```
| Область собеседования | Что сможешь продемонстрировать
|
| --------------------- |
------------------------------------------------------- |
| Java 21               | Collections, Streams, Concurrency, Virtual Threads,
JVM |
```

```
| Spring Boot           | Configuration, DI, Actuator, Profiles
|
| Spring Security       | JWT, OAuth2, OIDC, RBAC
|
| Spring Cloud          | Gateway, Config, Resilience
|
| Microservices         | Service boundaries, independent deployment
|
| Distributed Systems   | Saga, Outbox, eventual consistency
|
| Databases             | PostgreSQL, Hibernate, MongoDB
|
| Performance           | Redis, caching, indexing, load testing
|
| Messaging             | Kafka, Avro, CDC
|
| Testing               | JUnit, Mockito, AssertJ, Testcontainers
|
| DevOps                | Docker, Kubernetes, CI/CD
|
| Observability         | Grafana, Prometheus, ELK, Jaeger
|
| Frontend              | React, TypeScript, Vite, React Query
|
| Architecture          | DDD, Hexagonal, CQRS, BFF
|
| Security              | Secrets, Vault, token management
|
```

```
Особенно рекомендую подготовить для каждого крупного архитектурного решения
ответы на вопросы:
```

- `Почему выбрал этот паттерн?` 

- `Какие альтернативы рассматривал?` 

- `Какие недостатки у выбранного решения?` 

- `Что произойдёт при отказе одного сервиса?` 

- `Как обеспечивается идемпотентность?` 

- `Как масштабировать систему?` 

- `Как избежать потери событий?` 

- `Как обеспечить backward compatibility?` 

```
Именно это позволит использовать emarket не просто как демонстрационный магазин,
а как полноценный материал для Senior System Design Interview.
```

```
## 20. Моя рекомендация: в каком порядке действительно начинать
```

```
Я бы зафиксировал следующие приоритеты:
```

```
## Приоритеты разработки
```

```
P0 — Foundation
```

```
Критично
```

```
GitHub Organization, CI, Git conventions, pre-commit, Docker, Minikube,
документация.
```

```
P1 — Working application
```

```
Высокий
```

```
Web, WireMock, Gateway, BFF, Identity, User.
```

```
P2 — E-commerce
```

# `Высокий` 

```
Catalog, Cart, Inventory, Order, Payment.
```

```
P3 — Distributed Architecture
```

# `Средний` 

```
Kafka, Avro, Schema Registry, Debezium, Outbox, Saga, CQRS.
```

```
P4 — Production Engineering
```

```
Финальный
```

```
Vault, Observability, Kubernetes, performance, security, disaster recovery.
```

```
Три архитектурных решения, которые я предлагаю закрепить с самого начала:
```

```
1. API Gateway — единственная публичная точка входа. Web и Mobile работают через
разные BFF.
2. Database-per-Service — никаких общих бизнес-таблиц между микросервисами.
3. Независимый deployment — отдельный GitHub repository, CI, Docker image и
Kubernetes Deployment для каждого сервиса.
```

```
Эта архитектура продолжает ранее определённое разделение `emarket-web`, Gateway,
Web/Mobile BFF, Identity и User Service.&#x20;
```
