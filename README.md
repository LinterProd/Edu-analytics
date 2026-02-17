# Edu Analytics

**Edu Analytics** — компактная, но мощная **система продуктовой и финансовой аналитики** для образовательных платформ: метрики аудитории, выручки и удержания — в одном дашборде и через API.

## Почему этот проект

MVP за 3 дня для нетехнического заказчика.

Задача: построить, объяснить и передать рабочий продукт человеку 
без технического бэкграунда — на его машине, с его данными, 
с документацией.

Результат:
- LTV, CAC, ARPPU, DAU/WAU/MAU, retention в одном дашборде
- Переключение периода через AJAX без перезагрузки страницы
- Экспорт JSON + CSV
- Полная документация для нетехнического пользователя
- Онбординг на машине заказчика

---

## Технологии

- **Java**: 21
- **Backend**: Spring Boot 3.5.7 (`spring-boot-starter-web`)
- **Данные**: Spring Data JPA / Hibernate (`spring-boot-starter-data-jpa`)
- **UI**: Thymeleaf (`spring-boot-starter-thymeleaf`), Bootstrap 5
- **Графики**: Chartist (рендер в браузере)
- **Экспорт**: OpenCSV
- **БД для быстрого старта**: H2 (runtime)
- **Удобство разработки**: Lombok, DevTools
- **Тесты**: `spring-boot-starter-test`

---

## Что именно считает Edu Analytics

- **Аудитория**
  - **DAU/WAU/MAU**: активная аудитория по дням/неделям/месяцам
- **Retention**
  - **Retention Rate**: доля вернувшихся пользователей
  - **Retention Trend**: динамика удержания (автоматическая агрегация по периоду)
- **Финансы**
  - **LTV** (Lifetime Value)
  - **CAC** (Customer Acquisition Cost) за выбранный период
  - **ARPPU** (Average Revenue Per Paying User) за выбранный период
- **Продукты**
  - **Top products**: продажи и выручка по курсам (ранжирование по спросу)
- **Сводка дашборда**
  - пользователи / курсы / заказы / выручка за период

---

## UI и сценарии

Основная страница: `GET /`  
Возможности:
- выбор периода: `last7days`, `last30days`, `last90days`, `last365days`
- карточки метрик + интерактивные графики
- экспорт данных: **JSON** и **CSV**

---

## API

### Получить метрики (для фронта и интеграций)

`GET /api/metrics?period=last30days`

Возвращает `CompleteMetricsResponse`:
- `dashboardStats`: `{ userCount, courseCount, orderCount, totalRevenue }`
- `audienceMetrics`: `{ "DAU": {date: value}, "WAU": {...}, "MAU": {...} }`
- `retentionRate`: number
- `ltv`, `cac`, `arppu`: number
- `productPerformance`: массив `{ courseId, courseName, salesCount, revenue }`
- `retentionTrend`: `{date: percent}`

Пример (сокращённый):

```json
{
  "dashboardStats": { "userCount": 1200, "courseCount": 18, "orderCount": 340, "totalRevenue": 125000.50 },
  "audienceMetrics": {
    "DAU": { "2026-02-01": 120, "2026-02-02": 135 },
    "WAU": { "2026-02-01": 520, "2026-02-02": 540 },
    "MAU": { "2026-02-01": 1800, "2026-02-02": 1820 }
  },
  "retentionRate": 62.5,
  "ltv": 2450.00,
  "cac": 410.00,
  "arppu": 980.00,
  "productPerformance": [
    { "courseId": 1, "courseName": "Java Pro", "salesCount": 120, "revenue": 60000.00 }
  ],
  "retentionTrend": { "2026-02-01": 61.2, "2026-02-02": 62.5 }
}
```

---

## Архитектура

Проект построен на **MVC + Service layer** и усилен паттерном **Facade**:

- **Controller**: `DashboardController`
  - `GET /` — рендер дашборда (Thymeleaf)
  - `GET /api/metrics` — JSON для AJAX/интеграций
- **Facade**: `MetricsFacadeService`
  - собирает полный пакет метрик за 1 вызов
  - автоматически выбирает агрегацию графиков: `day` / `week` / `month`
- **Services**
  - `AudienceMetricsService` — активная аудитория (DAU/WAU/MAU)
  - `RetentionMetricsService` — retention rate + тренд
  - `FinancialMetricsService` — LTV / CAC / ARPPU
  - `ProductMetricsService` — performance по курсам
- **Repositories (JPA)**
  - `UserRepository`, `OrderRepository`, `CourseRepository`
- **Domain model**
  - `User`, `Course`, `Order`

---

## Быстрый старт

### Требования

- **Java 21**
- Maven (или wrapper `mvnw`)

### Запуск

```bash
# Windows
mvnw.cmd spring-boot:run

# Linux/macOS
./mvnw spring-boot:run
```

Откройте:
- UI: `http://localhost:8080/`
- H2 Console: `http://localhost:8080/h2-console`
  - **JDBC URL**: `jdbc:h2:mem:testdb`
  - **Username**: `sa`
  - **Password**: (пусто)

---

## Конфигурация

Основные настройки: `src/main/resources/application.yml`

- порт: `server.port=8080`
- datasource: H2 in-memory `jdbc:h2:mem:testdb`
- `spring.jpa.hibernate.ddl-auto=update` (автогенерация схемы)

---

## Тестирование

В проекте есть тесты контроллера, сервисов, репозиториев и интеграционные сценарии.

```bash
# Windows
mvnw.cmd test

# Linux/macOS
./mvnw test
```

---

## Структура проекта

```
src/
  main/
    java/com/linter/eduanalitycs/
      controller/     # MVC controllers
      service/        # бизнес-логика + фасад метрик
      repository/     # JPA repositories
      model/
        entity/       # JPA entities: User/Course/Order
        dto/          # DTO контракт API
    resources/
      application.yml
      templates/      # Thymeleaf templates
      static/         # css/js
  test/               # unit/integration tests
pom.xml
```