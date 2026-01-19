# Лабораторные работы по базам данных

## Студент
**Линеевич Степан** (схема `Lineycev_Stepan_2272`)

## Структура проекта


## Выполненные лабораторные работы

Lineycev_Stepan_2272/
├── Таблицы (8 шт.)
├── Представления (Views)
├── Функции (Functions)
├── Триггеры (Triggers)
└── Индексы (Indexes)

###  Лабораторная 1-2: Создание БД "CarShowroom"
**Цель:** Спроектировать и реализовать реляционную БД автосалона.

#### Созданные таблицы:
1. **Brands** - производители автомобилей
2. **CarModels** - модели автомобилей
3. **Showroom** - автосалоны
4. **Cars** - автомобили в наличии
5. **Employees** - сотрудники
6. **Customers** - клиенты
7. **Sales** - продажи
8. **TestDrives** - тест-драйвы

#### Пример запроса с JOIN:
```sql
SELECT s.sale_date, c.first_name, b.brand_name, cm.model_name
FROM Sales s
JOIN Customers c ON s.CustomerID = c.CustomerID
JOIN Cars car ON s.CarID = car.CarID
JOIN CarModels cm ON car.ModelID = cm.ModelID
JOIN Brands b ON cm.BrandID = b.BrandID; 
```
Лабораторная 3: Представления и функции
Цель: Создать виртуальные представления и параметризованные функции.

Созданные объекты:
Представление Sales_Report - сводный отчет по продажам

Функция Get_Sales_Report() - фильтрация отчета по параметрам

Использование функции:
```sql
-- Все продажи
SELECT * FROM Get_Sales_Report();

-- Продажи Toyota за март 2024
SELECT * FROM Get_Sales_Report(
    p_start_date => '2024-03-01',
    p_end_date => '2024-03-31',
    p_brand_name => 'Toyota'
);
```
Лабораторная 4: Анализ производительности
Цель: Оптимизировать запросы через индексы и анализ планов выполнения.

Генерация тестовых данных:
Создано 20 000+ записей в основных таблицах

Разработаны функции генерации случайных данных

Созданные индексы:

```sql
CREATE INDEX idx_sales_saledate ON Sales(sale_date);
CREATE INDEX idx_sales_customerid ON Sales(customerid);
CREATE INDEX idx_cars_modelid ON Cars(modelid);
-- и другие...
```

<img width="867" height="265" alt="image" src="https://github.com/user-attachments/assets/ea3f447c-2759-4c44-9e62-4087a0f0d56c" />

Лабораторная 5: Триггеры
Цель: Реализовать каскадное удаление и систему аудита изменений.

Созданные объекты:
Таблица Change_Log - журнал всех изменений

Триггер trigger_log_sales_changes - автоматическое журналирование

Триггер trigger_cascade_delete_brand - каскадное удаление

Пример работы триггеров:

-- При удалении бренда автоматически удаляются все его модели
DELETE FROM Brands WHERE BrandID = 1;

-- Все изменения в таблице Sales фиксируются в Change_Log
INSERT INTO Sales (...) VALUES (...);

Технологический стек
СУБД: PostgreSQL 15+

Интерфейс: pgAdmin 4

Язык: SQL, PL/pgSQL

Как развернуть проект
Создать схему в PostgreSQL:
```sql
CREATE SCHEMA "Lineycev_Stepan_2272";
```
Выполнить DDL-скрипты в порядке:

01_tables.sql

02_data.sql

03_views_functions.sql

04_indexes.sql

05_triggers.sql

Для тестирования использовать скрипты из папки tests/
