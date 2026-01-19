Лабораторная работа 1-2: Создание базы данных «CarShowroom» {#лаб1}
1.1. Анализ ER-диаграммы
На основе предоставленной ER-диаграммы была спроектирована физическая модель базы данных, включающая 8 таблиц:


<img width="734" height="462" alt="image" src="https://github.com/user-attachments/assets/9afba545-38fe-4dd3-824e-1678a590efcb" />
<img width="1057" height="682" alt="image" src="https://github.com/user-attachments/assets/d3bc45e4-6f5b-42f8-b57f-884a473d7287" />


1.2. DDL-скрипты создания таблиц
```sql
-- Создание схемы
CREATE SCHEMA "Lineycev_Stepan_2272";

-- Таблица Brands
CREATE TABLE "Lineycev_Stepan_2272".Brands (
    BrandID SERIAL PRIMARY KEY,
    brand_name VARCHAR(100) NOT NULL,
    country VARCHAR(50),
    foundation_year INTEGER
);

-- Таблица CarModels
CREATE TABLE "Lineycev_Stepan_2272".CarModels (
    ModelID SERIAL PRIMARY KEY,
    BrandID INTEGER NOT NULL REFERENCES Brands(BrandID) ON DELETE CASCADE,
    model_name VARCHAR(100) NOT NULL,
    vehicle_type VARCHAR(50),
    generation VARCHAR(50)
);

-- Остальные таблицы создаются аналогично...
```
1.3. Заполнение таблиц тестовыми данными
В каждую таблицу было добавлено не менее 4 записей:

Brands: Toyota, BMW, Tesla, Lada

CarModels: Camry, X5, Model 3, Vesta и др.

Cars: 5 автомобилей с различными характеристиками

Sales: 4 записи о продажах

и т.д.

1.4. Содержательный запрос с JOIN (2-3 таблицы)
```sql
SELECT 
    s.sale_date,
    c.first_name || ' ' || c.last_name AS customer_name,
    b.brand_name,
    cm.model_name,
    car.color,
    car.year_of_manufacture,
    s.sale_price,
    e.first_name || ' ' || e.last_name AS employee_name,
    sh.address AS showroom_address
FROM "Lineycev_Stepan_2272".Sales s
JOIN "Lineycev_Stepan_2272".Customers c ON s.CustomerID = c.CustomerID
JOIN "Lineycev_Stepan_2272".Cars car ON s.CarID = car.CarID
JOIN "Lineycev_Stepan_2272".CarModels cm ON car.ModelID = cm.ModelID
JOIN "Lineycev_Stepan_2272".Brands b ON cm.BrandID = b.BrandID
JOIN "Lineycev_Stepan_2272".Employees e ON s.EmployeeID = e.EmployeeID
JOIN "Lineycev_Stepan_2272".Showroom sh ON car.ShowroomID = sh.ShowroomID
ORDER BY s.sale_date DESC;
```
Результат: выборка содержит информацию о продажах, клиентах, автомобилях, продавцах и автосалонах.

Лабораторная работа 3: Представления и функции {#лаб3}
3.1. Создание представления (View)
```sql
CREATE OR REPLACE VIEW "Lineycev_Stepan_2272"."Sales_Report" AS
SELECT 
    s.SaleID AS saleid,
    s.sale_date AS "Дата продажи",
    CONCAT(c.first_name, ' ', c.last_name) AS "Клиент",
    c.phone_number AS "Телефон клиента",
    CONCAT(b.brand_name, ' ', cm.model_name) AS "Автомобиль",
    car.color AS "Цвет",
    car.year_of_manufacture AS "Год выпуска",
    car.vin AS "VIN",
    s.sale_price AS "Цена продажи",
    s.payment_method AS "Метод оплаты",
    s.warranty_months AS "Гарантия (мес.)",
    CONCAT(e.first_name, ' ', e.last_name) AS "Продавец",
    e.job_title AS "Должность",
    sh.address AS "Автосалон",
    sh.phone_number AS "Телефон салона"
FROM "Lineycev_Stepan_2272".Sales s
JOIN ...  -- все JOIN как в запросе выше
ORDER BY s.sale_date DESC;
```
Назначение: виртуальная таблица для удобного формирования отчётов о продажах.

3.2. Создание функции с параметрами
```sql
CREATE OR REPLACE FUNCTION "Lineycev_Stepan_2272"."Get_Sales_Report"(
    p_start_date DATE DEFAULT NULL,
    p_end_date DATE DEFAULT NULL,
    p_brand_name VARCHAR(100) DEFAULT NULL
)
RETURNS SETOF "Lineycev_Stepan_2272"."Sales_Report"
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT *
    FROM "Lineycev_Stepan_2272"."Sales_Report" sr
    WHERE 
        (p_start_date IS NULL OR sr."Дата продажи" >= p_start_date)
        AND (p_end_date IS NULL OR sr."Дата продажи" <= p_end_date)
        AND (p_brand_name IS NULL OR sr."Автомобиль" ILIKE '%' || p_brand_name || '%')
    ORDER BY sr."Дата продажи" DESC;
END;
$$;
```
Пример вызова:
```sql
SELECT * FROM "Lineycev_Stepan_2272"."Get_Sales_Report"(
    p_start_date => '2024-03-01',
    p_end_date => '2024-03-31',
    p_brand_name => 'Toyota'
);
```

Лабораторная работа 4: Анализ производительности и оптимизация {#лаб4}
4.1. Генерация больших объёмов данных
Созданы функции для генерации случайных данных и заполнены таблицы:

Cars: 20 000 записей

Customers: 20 000 записей

Sales: 15 000 записей

и др.

4.2. Анализ планов выполнения запросов
До оптимизации:
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM "Lineycev_Stepan_2272"."Sales_Report" LIMIT 100;
```
Результат: Seq Scan, Hash Join, время выполнения ~85 мс.

После создания индексов:
```sql
CREATE INDEX idx_sales_saledate ON "Lineycev_Stepan_2272".Sales(sale_date);
CREATE INDEX idx_sales_customerid ON "Lineycev_Stepan_2272".Sales(customerid);
-- и другие индексы...
```
Результат: Index Scan, Nested Loop, время выполнения ~3.5 мс.
<img width="705" height="260" alt="image" src="https://github.com/user-attachments/assets/087a2e8f-cba6-4849-9923-08077f693ab9" />
Вывод: Индексы кардинально улучшили производительность запросов.

Лабораторная работа 5: Триггеры и журналирование {#лаб5}
5.1. Создание таблицы-журнала
```sql
CREATE TABLE "Lineycev_Stepan_2272"."Change_Log" (
    log_id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    operation VARCHAR(10) NOT NULL,
    record_id INTEGER,
    changed_data JSONB,
    changed_by VARCHAR(100) DEFAULT CURRENT_USER,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
5.2. Функция для журналирования
```sql
CREATE OR REPLACE FUNCTION "Lineycev_Stepan_2272"."log_change"()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO "Lineycev_Stepan_2272"."Change_Log" 
        (table_name, operation, record_id, changed_data)
    VALUES 
        (TG_TABLE_NAME, TG_OP, 
         CASE WHEN TG_OP = 'DELETE' THEN OLD.saleid ELSE NEW.saleid END,
         CASE WHEN TG_OP = 'INSERT' THEN row_to_json(NEW)
              WHEN TG_OP = 'UPDATE' THEN jsonb_build_object('old', row_to_json(OLD), 'new', row_to_json(NEW))
              ELSE row_to_json(OLD) END);
    RETURN COALESCE(NEW, OLD);
END;
$$;
```
5.3. Триггер каскадного удаления
```sql
CREATE OR REPLACE FUNCTION "Lineycev_Stepan_2272"."cascade_delete_brand"()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    DELETE FROM "Lineycev_Stepan_2272"."CarModels"
    WHERE BrandID = OLD.BrandID;
    RETURN OLD;
END;
$$;

CREATE TRIGGER trigger_cascade_delete_brand
BEFORE DELETE ON "Lineycev_Stepan_2272"."Brands"
FOR EACH ROW
EXECUTE FUNCTION "Lineycev_Stepan_2272"."cascade_delete_brand"();
```
5.4. Триггер для автоматического журналирования
```sql
CREATE TRIGGER trigger_log_sales_changes
AFTER INSERT OR UPDATE OR DELETE ON "Lineycev_Stepan_2272"."Sales"
FOR EACH ROW
EXECUTE FUNCTION "Lineycev_Stepan_2272"."log_change"();
```
5.5. Тестирование триггеров
Журналирование: При любой операции с таблицей Sales в Change_Log добавляется запись.

Каскадное удаление: При удалении бренда автоматически удаляются все его модели.

Проверка данных: Триггер предотвращает продажи с будущей датой.

Общие выводы {#выводы}
В ходе выполнения лабораторных работ были успешно освоены:

Проектирование БД – от ER-диаграммы до физической реализации в PostgreSQL.

Работа в pgAdmin – создание схем, таблиц, представлений, функций, триггеров.

SQL-запросы – DDL, DML, сложные SELECT с JOIN, агрегатные функции.

Оптимизация – анализ планов выполнения, создание индексов, улучшение производительности.

Триггеры – реализация каскадного удаления и аудита изменений.

Результат: Полностью функционирующая база данных «CarShowroom» с:

8 таблицами

20 000+ тестовых записей

Оптимизированными запросами

Системой журналирования изменений

Все задания выполнены в соответствии с требованиями. База данных готова к использованию в реальных условиях для учёта продаж автомобилей и анализа данных.
