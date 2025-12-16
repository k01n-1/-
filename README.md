
Лабораторные работы по Базам Данных  
Костюк Иван 02261-ДБ  
Вариант 20. Зарплата
Постановка задачи: Расчет заработной платы предприятия
1. Описание предметной области:
Система предназначена для автоматизации расчета ежемесячной заработной платы сотрудников предприятия, организованных в отделы. Расчет основывается на тарифной сетке (ставка за час) и данных об отработанном времени.

2. Перечень сущностей и их атрибутов (на основе ER-диаграммы):

ОТДЕЛЫ (departments)

id (PK) — Уникальный идентификатор отдела.

название (VARCHAR) — Наименование отдела.

СОТРУДНИКИ (employees)

id (PK) — Уникальный идентификатор сотрудника (внутренний).

табельный_номер (INT) — Уникальный табельный номер сотрудника.

фамилия (VARCHAR) — Фамилия сотрудника.

имя (VARCHAR) — Имя сотрудника.

отчество (VARCHAR) — Отчество сотрудника.

должность (VARCHAR) — Должность сотрудника (логически связана с ТАРИФНАЯ_СЕТКА.должность).

разряд (INT) — Разряд сотрудника (логически связан с ТАРИФНАЯ_СЕТКА.разряд).

id_отдела (FK -> ОТДЕЛЫ.id) — Идентификатор отдела, в котором работает сотрудник.

ТАРИФНАЯ_СЕТКА (pay_scales)

id (PK) — Уникальный идентификатор записи в тарифной сетке.

должность (VARCHAR) — Наименование должности.

разряд (INT) — Разряд (от 7 до 15).

ставка_руб_час (DECIMAL) — Часовая тарифная ставка для данной должности и разряда.

ТАБЕЛЬ (timesheets)

id (PK) — Уникальный идентификатор записи в табеле.

id_сотрудника (FK -> СОТРУДНИКИ.id) — Идентификатор сотрудника.

год (INT) — Год отчетного периода.

месяц (INT) — Месяц отчетного периода.

отработано_часов (DECIMAL) — Количество часов, отработанных сотрудником в указанном месяце.

3. Связи между сущностями (Relationships):

РАБОТАЕТ (рабочее место): Связь между СОТРУДНИКИ и ОТДЕЛЫ. Один сотрудник работает только в одном отделе (id_отдела). В одном отделе может работать много сотрудников. (1:N).

ИМЕЕТ_ТАРИФ: Связь между СОТРУДНИКИ и ТАРИФНАЯ_СЕТКА. Ставка сотрудника определяется парой атрибутов (должность, разряд), которые должны существовать в тарифной сетке. Это логическая связь по атрибутам.

ИМЕЕТ_ЗАПИСИ (табель учета): Связь между СОТРУДНИКИ и ТАБЕЛЬ. На одного сотрудника ежемесячно заводится одна запись в табеле (за конкретный год и месяц). Для одного сотрудника набирается много записей за разные периоды. (1:N).

4. Основные бизнес-процессы:

Управление справочниками: Ведение списка отделов и тарифной сетки.

Управление кадрами: Ведение списка сотрудников, их привязка к отделам и назначение должности/разряда.

Учет рабочего времени: Ежемесячное заполнение табеля учета рабочего времени для каждого сотрудника.

Расчет заработной платы: Автоматический расчет суммы к выплате для каждого сотрудника за период по формуле:
Зарплата = (Ставка_сотрудника) * (Отработано_часов)
где Ставка_сотрудника определяется поиском в ТАРИФНАЯ_СЕТКА по его должности и разряду.

5. Требуемые выходные документы (отчеты):

Расчетная ведомость за период: Список для выдачи зарплаты сотрудникам за указанный месяц и год. Должен содержать: отдел, табельный номер, ФИО, должность, разряд, часовую ставку, количество отработанных часов, сумму к выплате. Упорядочить по отделам, а внутри отдела — по ФИО сотрудника.

Сводка по квалификации отделов: Аналитический отчет, показывающий распределение сотрудников по разрядам в разрезе отделов. Должен содержать: название отдела, разряд, количество сотрудников данного разряда в отделе. Упорядочить по названию отдела.
## ER-диаграмма
<img width="726" height="437" alt="Снимок" src="https://github.com/user-attachments/assets/0944bf4b-bc37-4687-892f-3c3096337fc4" />




## Логическая модель по диаграмме

``` mermaid
classDiagram
    direction TB
    
    class Отдел {
        +int id
        +string название
        
        +getСотрудники() List
    }
    
    class Сотрудник {
        +int id
        +int табельныйНомер
        +string ФИО
        +int разряд
        
        +расчитатьЗарплату() decimal
    }
    
    class Должность {
        +int id
        +string название
        
        +getСтавки() List
    }
    
    class ТарифнаяСетка {
        +int id
        +int разряд
        +decimal ставка
    }
    
    class Табель {
        +int id
        +date месяц
        +decimal часы
    }
    
    %% Основные отношения
    Отдел "1" -- "*" Сотрудник : работает в
    Сотрудник "1" -- "1" Должность : занимает
    Должность "1" -- "*" ТарифнаяСетка : имеет ставки
    Сотрудник "1" -- "*" Табель : имеет табели
```
##Физическая модель 
```mermaid
erDiagram
    ОТДЕЛ {
        integer id PK "SERIAL PRIMARY KEY"
        varchar название "NOT NULL UNIQUE"
    }
    
    ДОЛЖНОСТЬ {
        integer id PK "SERIAL PRIMARY KEY"
        varchar название "NOT NULL UNIQUE"
    }
    
    ТАРИФНАЯ_СЕТКА {
        integer id PK "SERIAL PRIMARY KEY"
        integer должность_id FK "NOT NULL REFERENCES ДОЛЖНОСТЬ(id) ON DELETE CASCADE"
        integer разряд "NOT NULL CHECK (разряд BETWEEN 7 AND 15)"
        decimal ставка "NOT NULL CHECK (ставка > 0)"
    }
    
    СОТРУДНИК {
        integer id PK "SERIAL PRIMARY KEY"
        integer табельный_номер "NOT NULL UNIQUE"
        varchar фамилия "NOT NULL"
        varchar имя "NOT NULL"
        varchar отчество "NULL"
        integer разряд "NOT NULL CHECK (разряд BETWEEN 7 AND 15)"
        integer отдел_id FK "NOT NULL REFERENCES ОТДЕЛ(id) ON DELETE RESTRICT"
        integer должность_id FK "NOT NULL REFERENCES ДОЛЖНОСТЬ(id) ON DELETE RESTRICT"
    }
    
    ТАБЕЛЬ {
        integer id PK "SERIAL PRIMARY KEY"
        integer сотрудник_id FK "NOT NULL REFERENCES СОТРУДНИК(id) ON DELETE CASCADE"
        integer год "NOT NULL CHECK (год BETWEEN 2020 AND 2100)"
        integer месяц "NOT NULL CHECK (месяц BETWEEN 1 AND 12)"
        decimal часы "NOT NULL CHECK (часы BETWEEN 0 AND 300)"
    }
    
    ОТДЕЛ ||--o{ СОТРУДНИК : "FOREIGN KEY (отдел_id)"
    ДОЛЖНОСТЬ ||--o{ СОТРУДНИК : "FOREIGN KEY (должность_id)"
    ДОЛЖНОСТЬ ||--o{ ТАРИФНАЯ_СЕТКА : "FOREIGN KEY (должность_id)"
    СОТРУДНИК ||--o{ ТАБЕЛЬ : "FOREIGN KEY (сотрудник_id)"
    
    ТАРИФНАЯ_СЕТКА }|--|| СОТРУДНИК : "должность_id + разряд"
```
##2 Лабораторная
Создаем 5 таблиц
<img width="914" height="729" alt="1" src="https://github.com/user-attachments/assets/8665ca11-3bf2-43c3-911d-4ade709b6473" />
<img width="986" height="849" alt="2 jpg" src="https://github.com/user-attachments/assets/e5eafb69-ee52-4cc0-9972-7565aa9daf7e" />

далее записываем данные и выводим их для проверки для всех 5 таблиц
<img width="477" height="228" alt="3" src="https://github.com/user-attachments/assets/f078cf5c-59da-4d37-8b55-0f77efe19eb7" />
<img width="617" height="543" alt="4" src="https://github.com/user-attachments/assets/8f6b3f15-9860-4268-83d9-771f616da6dc" />
<img width="588" height="273" alt="5" src="https://github.com/user-attachments/assets/4b716720-9260-4107-b2ca-046f3e01c535" />
<img width="400" height="902" alt="6 jpg" src="https://github.com/user-attachments/assets/579130a3-8855-4fae-9a5d-3aa845a70338" />
<img width="819" height="577" alt="7" src="https://github.com/user-attachments/assets/6b740186-5d03-497e-ab14-6f06df14c30d" />
<img width="562" height="952" alt="8 jpg" src="https://github.com/user-attachments/assets/3f4efb18-aaf6-4a08-9bcc-d0d4bfe26569" />
<img width="1011" height="402" alt="9" src="https://github.com/user-attachments/assets/187491c6-ecf6-4d72-98d3-741bd9f969cc" />
<img width="947" height="934" alt="10 jpg" src="https://github.com/user-attachments/assets/7a0a80b5-3d6e-422d-8f6a-23c55bbb8019" />
<img width="786" height="928" alt="11 jpg" src="https://github.com/user-attachments/assets/b01ff36a-a853-4fc4-95b7-d57dccc8c0e4" />
<img width="638" height="1057" alt="12 jpg" src="https://github.com/user-attachments/assets/56ce18c4-2255-439b-9e72-b856aa33df5d" />
здесь Выполнены SELECT-запросы c JOIN
<img width="874" height="1080" alt="13 jpg" src="https://github.com/user-attachments/assets/301ec00c-6cc2-44d5-a16d-8bf4fa91b8a5" />

<img width="1210" height="456" alt="14 jpg" src="https://github.com/user-attachments/assets/e92c0e92-5649-4520-b5c4-9ad95aee6186" />
<img width="1434" height="420" alt="15 jpg" src="https://github.com/user-attachments/assets/70475557-ae5f-4e0a-aa2c-7ac36385cad8" />

##3 Лабораторная

создадим 2 представления 
расчетная ведомость
<img width="859" height="452" alt="20 5" src="https://github.com/user-attachments/assets/26d7ca3f-8306-4528-995c-ed170083f84a" />
<img width="1497" height="704" alt="23 jpg" src="https://github.com/user-attachments/assets/9e4def9c-737e-439d-96a2-2ccd06c9c862" />

сводка по разрядам
<img width="1483" height="919" alt="20 jpg" src="https://github.com/user-attachments/assets/0c0cc6ef-c8c7-411f-b5e7-f0b02aaf9ecc" />
<img width="658" height="671" alt="24 jpg" src="https://github.com/user-attachments/assets/54f9c3cf-a8e5-4148-8fbe-d3645178f56e" />
процедура для подсчета зарплаты
<img width="507" height="282" alt="21 jpg" src="https://github.com/user-attachments/assets/e7478099-7eb0-431e-abef-53aeaf2d9a92" />
процедура для добавления табеля
## 4 лаба
генератор отделов
```code
CREATE OR REPLACE PROCEDURE kostuk.генерировать_отделы(количество_отделов INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    названия TEXT[] := ARRAY[
        'Лесозаготовительный', 'Распиловочный', 'Сортировочный', 
        'Сушка древесины', 'Склад готовой продукции', 'Транспортный',
        'Ремонтно-механический', 'Энергетический', 'Лаборатория качества',
        'Охрана труда', 'Бухгалтерия', 'Отдел кадров', 'Снабжение',
        'Сбыт', 'Экологический', 'Склад сырья', 'Упаковочный',
        'Контроль качества', 'Производственный', 'Администрация'
    ];
BEGIN
    INSERT INTO kostuk."Отдел" (название)
    SELECT 
        названия[1 + ((seq-1) % array_length(названия, 1))]  
        ' №'  seq::TEXT
    FROM generate_series(1, количество_отделов) as seq;
END;
$$;
```
![30](https://github.com/user-attachments/assets/fe44c4a5-a9c2-4116-8f40-e1ea53e26679)
генератор должностей
```code
CREATE OR REPLACE PROCEDURE kostuk.генерировать_должности(количество_должностей INTEGER)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO kostuk."Должность" (название)
    SELECT 'Должность ' || seq::TEXT
    FROM generate_series(1, количество_должностей) as seq;
END;
$$;
```
![31](https://github.com/user-attachments/assets/6cc7f074-c66a-4a57-b044-30ff78561f5f)
тарифы
```code
CREATE OR REPLACE PROCEDURE kostuk.генерировать_тарифы(количество_тарифов INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    должности_count INTEGER;
    i INTEGER := 0;
    position_id INTEGER;
    grade INTEGER;
    min_grade INTEGER := 7;
    max_grade INTEGER := 15;
    grade_range INTEGER := max_grade - min_grade + 1; -- 9 разрядов (7-15 включительно)
BEGIN
    SELECT COUNT(*) INTO должности_count FROM kostuk."Должность";
    
    -- Максимально возможное количество уникальных тарифов
    DECLARE
        max_possible_tariffs INTEGER;
    BEGIN
        max_possible_tariffs := должности_count * grade_range;
        
        -- Если запрашиваем больше, чем возможно, ограничиваем
        IF количество_тарифов > max_possible_tariffs THEN
            RAISE NOTICE 'Максимально возможное количество тарифов: %', max_possible_tariffs;
            количество_тарифов := max_possible_tariffs;
        END IF;
    END;
    
    -- Генерируем тарифы пока не достигнем нужного количества
    WHILE i < количество_тарифов LOOP
        -- Случайная должность
        SELECT id INTO position_id 
        FROM kostuk."Должность" 
        ORDER BY RANDOM() 
        LIMIT 1;
        
        -- Случайный разряд от 7 до 15
        grade := min_grade + (RANDOM() * (grade_range - 1))::INTEGER;
        
        -- Пытаемся вставить
        INSERT INTO kostuk."Тарифная_Сетка" (должность_id, разряд, ставка)
        VALUES (position_id, grade, 100 + (RANDOM() * 1900)::NUMERIC(10,2))
        ON CONFLICT (должность_id, разряд) DO NOTHING;
        
        -- Если вставка прошла успешно (не было конфликта), увеличиваем счетчик
        IF FOUND THEN
            i := i + 1;
        END IF;
    END LOOP;
    
    RAISE NOTICE 'Создано тарифов: %', i;
END;
$$;
```
![32](https://github.com/user-attachments/assets/9a4c4ddf-2ddb-4bd9-9573-08fc5cccfe14)
генератор сотрудников
```code
CREATE OR REPLACE PROCEDURE kostuk.генерировать_сотрудников(количество_сотрудников INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    фамилии TEXT[] := ARRAY[
        'Иванов','Петров','Сидоров','Козлов','Смирнов','Михайлов','Новиков',
        'Кузнецов','Попов','Васильев','Фёдоров','Морозов','Волков','Зайцев',
        'Лебедев','Соколов','Орлов','Егоров','Никитин','Соловьёв',
        'Белов','Григорьев','Тихонов','Алексеев','Филиппов','Андреев',
        'Макаров','Николаев','Степанов','Захаров','Павлов','Романов',
        'Владимиров','Сергеев','Герасимов','Борисов','Кириллов','Денисов',
        'Тимофеев','Матвеев','Артемьев','Яковлев','Данилов','Георгиев'
    ];
    имена_мужские TEXT[] := ARRAY[
        'Иван','Алексей','Дмитрий','Сергей','Андрей','Николай','Павел',
        'Виктор','Максим','Артём','Владимир','Михаил','Георгий','Олег',
        'Юрий','Анатолий','Василий','Пётр','Константин','Егор',
        'Александр','Евгений','Вячеслав','Григорий','Леонид','Роман',
        'Тимур','Станислав','Валерий','Борис','Даниил','Тимофей',
        'Фёдор','Семён','Макар','Захар','Кирилл','Денис','Матвей','Ярослав'
    ];
    отчества_мужские TEXT[] := ARRAY[
        'Иванович','Петрович','Сергеевич','Алексеевич','Дмитриевич','Андреевич',
        'Викторович','Михайлович','Владимирович','Олегович','Юрьевич','Геннадьевич',
        'Николаевич','Павлович','Анатольевич','Васильевич','Константинович',
        'Егорович','Александрович','Евгеньевич','Вячеславович','Григорьевич',
        'Леонидович','Романович','Тимурович','Станиславович','Валериевич',
        'Борисович','Даниилович','Тимофеевич','Фёдорович','Семёнович',
        'Макарович','Захарович','Кириллович','Денисович','Матвеевич','Ярославович'
    ];
    max_табельный INTEGER;
BEGIN
    SELECT COALESCE(MAX(табельный_номер), 100000)
    INTO max_табельный
    FROM kostuk."Сотрудник";

    INSERT INTO kostuk."Сотрудник" (табельный_номер, ФИО, разряд, отдел_id, должность_id)
    SELECT
        max_табельный + seq,
        фамилии[1 + ((seq-1) % array_length(фамилии, 1))]  ' ' 
        имена_мужские[1 + ((seq-1) % array_length(имена_мужские, 1))]  ' ' 
        отчества_мужские[1 + ((seq-1) % array_length(отчества_мужские, 1))],
        (SELECT разряд FROM kostuk."Тарифная_Сетка" ORDER BY RANDOM() LIMIT 1),
        (SELECT id FROM kostuk."Отдел" ORDER BY RANDOM() LIMIT 1),
        (SELECT id FROM kostuk."Должность" ORDER BY RANDOM() LIMIT 1)
    FROM generate_series(1, количество_сотрудников) as seq;
END;
$$;
```
![33](https://github.com/user-attachments/assets/d22e0aea-0204-41d3-aa09-8d03d9db578a)
генератор табеля
```code
CREATE OR REPLACE PROCEDURE kostuk.генерировать_табель_уникальный(количество_записей INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    i INTEGER := 0;
BEGIN
    WHILE i < количество_записей LOOP
        INSERT INTO kostuk."Табель" (сотрудник_id, год, месяц, часы)
        SELECT
            (SELECT id FROM kostuk."Сотрудник" ORDER BY RANDOM() LIMIT 1),
            2023 + (RANDOM() * 3)::INTEGER,
            1 + (RANDOM() * 11)::INTEGER,
            80 + (RANDOM() * 120)::NUMERIC(6,2)
        ON CONFLICT (сотрудник_id, год, месяц) DO NOTHING;
        
        -- Считаем сколько реально добавилось
        i := i + 1;
    END LOOP;
END;
$$;
```
![34](https://github.com/user-attachments/assets/d85ca335-b047-417a-8fba-d59121d9e868)
анализ запросов
```code
EXPLAIN ANALYZE
SELECT * FROM kostuk."Отдел" WHERE id = 100;

EXPLAIN ANALYZE
SELECT * FROM kostuk."Отдел" WHERE название LIKE 'Лесозаготовительный';

EXPLAIN ANALYZE
SELECT * FROM kostuk."Должность" WHERE id = 500;

EXPLAIN ANALYZE
SELECT * FROM kostuk."Должность" WHERE название LIKE 'инженер';

EXPLAIN ANALYZE
SELECT * FROM kostuk."Тарифная_Сетка" WHERE разряд BETWEEN 10 AND 12;

EXPLAIN ANALYZE
SELECT 
    ts.*,
    d.название as должность
FROM kostuk."Тарифная_Сетка" ts
JOIN kostuk."Должность" d ON ts.должность_id = d.id
WHERE ts.ставка > 1000;

EXPLAIN ANALYZE
SELECT * FROM kostuk."Сотрудник" WHERE разряд >= 10;

EXPLAIN ANALYZE
SELECT 
    s.*,
    d.название as должность,
    o.название as отдел
FROM kostuk."Сотрудник" s
JOIN kostuk."Должность" d ON s.должность_id = d.id
JOIN kostuk."Отдел" o ON s.отдел_id = o.id
WHERE s.ФИО LIKE 'Иванов';

EXPLAIN ANALYZE
SELECT * FROM kostuk."Табель" WHERE год = 2024 AND месяц = 6;

EXPLAIN ANALYZE
SELECT 
    t.*,
    s.ФИО,
    s.разряд
FROM kostuk."Табель" t
JOIN kostuk."Сотрудник" s ON t.сотрудник_id = s.id
WHERE t.часы > 150;
```
![35](https://github.com/user-attachments/assets/c74a8201-b811-46b3-ad42-2d3e06afa235)
создали индекс
![36](https://github.com/user-attachments/assets/2685a509-3fdd-4233-afc2-60295f41c93a)
принудительно отключили его
![37](https://github.com/user-attachments/assets/d18f6460-0578-4ed5-8843-5e2d03cf2c0c)
без индекса
![38](https://github.com/user-attachments/assets/a5b8bf64-5977-49a6-983e-4f25e65aa282)
с индексом 
![39](https://github.com/user-attachments/assets/30302b3e-3a8a-4055-886f-9ea035155158)
с индексами производительность быстрее
## 5 Лаба
-- 1. ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТАБЕЛЯ ПРИ УДАЛЕНИИ СОТРУДНИКА
```code
CREATE OR REPLACE FUNCTION kostuk.delete_tabel_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем табель сотрудника ID %', OLD.id;
    DELETE FROM kostuk."Табель" WHERE сотрудник_id = OLD.id;
    RETURN OLD;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER delete_tabel_cascade_trigger
    BEFORE DELETE ON kostuk."Сотрудник"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.delete_tabel_cascade_function();
```
-- 2. ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ СОТРУДНИКОВ ПРИ УДАЛЕНИИ ОТДЕЛА
```code
CREATE OR REPLACE FUNCTION kostuk.delete_sotrudniki_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем сотрудников отдела ID %', OLD.id;
    DELETE FROM kostuk."Сотрудник" WHERE отдел_id = OLD.id;
    RETURN OLD;
END;
$$;
```code
CREATE OR REPLACE TRIGGER delete_sotrudniki_cascade_trigger
    BEFORE DELETE ON kostuk."Отдел"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.delete_sotrudniki_cascade_function();
```
-- 3. ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТАРИФОВ И СОТРУДНИКОВ ПРИ УДАЛЕНИИ ДОЛЖНОСТИ
```code
CREATE OR REPLACE FUNCTION kostuk.delete_dolzhnost_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем тарифы и сотрудников должности ID %', OLD.id;
    
    -- Удаляем тарифные ставки для этой должности
    DELETE FROM kostuk."Тарифная_Сетка" WHERE должность_id = OLD.id;
    
    -- Удаляем сотрудников с этой должностью
    DELETE FROM kostuk."Сотрудник" WHERE должность_id = OLD.id;
    
    RETURN OLD;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER delete_dolzhnost_cascade_trigger
    BEFORE DELETE ON kostuk."Должность"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.delete_dolzhnost_cascade_function();
```
-- 4. ТАБЛИЦА АУДИТА ДЛЯ ВСЕХ ТАБЛИЦ
```code

CREATE TABLE IF NOT EXISTS kostuk."Аудит_Общий"(
    аудит_id SERIAL PRIMARY KEY,
    операция CHAR(1) NOT NULL,  -- I=INSERT, U=UPDATE, D=DELETE
    изменено_в TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    изменено_пользователем TEXT DEFAULT CURRENT_USER,
    таблица_имя TEXT NOT NULL,
    
    -- Общие поля для всех таблиц
    запись_id INTEGER,
    
    -- Поля для таблицы Отдел
    отдел_название TEXT,
    
    -- Поля для таблицы Должность
    должность_название TEXT,
    
    -- Поля для таблицы Тарифная_Сетка
    тариф_должность_id INTEGER,
    тариф_разряд INTEGER,
    тариф_ставка NUMERIC(10,2),
    
    -- Поля для таблицы Сотрудник
    сотрудник_табельный_номер INTEGER,
    сотрудник_фио TEXT,
    сотрудник_разряд INTEGER,
    сотрудник_отдел_id INTEGER,
    сотрудник_должность_id INTEGER,
    
    -- Поля для таблицы Табель
    табель_сотрудник_id INTEGER,
    табель_год INTEGER,
    табель_месяц INTEGER,
    табель_часы NUMERIC(6,2)
);
```
-- 5. ФУНКЦИИ И ТРИГГЕРЫ АУДИТА ДЛЯ КАЖДОЙ ТАБЛИЦЫ

-- 5.1. Аудит для таблицы "Отдел"
```code
CREATE OR REPLACE FUNCTION kostuk.audit_otdel_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            отдел_название
        )
        VALUES (
            'U', 'Отдел', NEW.id,
            NEW.название
        );
        RETURN NEW;
    ELSEIF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            отдел_название
        )
        VALUES (
            'D', 'Отдел', OLD.id,
            OLD.название
        );
        RETURN OLD;
    ELSEIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            отдел_название
        )
        VALUES (
            'I', 'Отдел', NEW.id,
            NEW.название
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER audit_otdel_trigger
    AFTER INSERT OR UPDATE OR DELETE ON kostuk."Отдел"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.audit_otdel_function();
```
-- 5.2. Аудит для таблицы "Должность"
```code
CREATE OR REPLACE FUNCTION kostuk.audit_dolzhnost_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            должность_название
        )
        VALUES (
            'U', 'Должность', NEW.id,
            NEW.название
        );
        RETURN NEW;
    ELSEIF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            должность_название
        )
        VALUES (
            'D', 'Должность', OLD.id,
            OLD.название
        );
        RETURN OLD;
    ELSEIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            должность_название
        )
        VALUES (
            'I', 'Должность', NEW.id,
            NEW.название
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER audit_dolzhnost_trigger
    AFTER INSERT OR UPDATE OR DELETE ON kostuk."Должность"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.audit_dolzhnost_function();
```
-- 5.3. Аудит для таблицы "Тарифная_Сетка"
```code
CREATE OR REPLACE FUNCTION kostuk.audit_tarif_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            тариф_должность_id, тариф_разряд, тариф_ставка
        )
        VALUES (
            'U', 'Тарифная_Сетка', NEW.id,
            NEW.должность_id, NEW.разряд, NEW.ставка
        );
        RETURN NEW;
    ELSEIF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            тариф_должность_id, тариф_разряд, тариф_ставка
        )
        VALUES (
            'D', 'Тарифная_Сетка', OLD.id,
            OLD.должность_id, OLD.разряд, OLD.ставка
        );
        RETURN OLD;
    ELSEIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            тариф_должность_id, тариф_разряд, тариф_ставка
        )
        VALUES (
            'I', 'Тарифная_Сетка', NEW.id,
            NEW.должность_id, NEW.разряд, NEW.ставка
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER audit_tarif_trigger
    AFTER INSERT OR UPDATE OR DELETE ON kostuk."Тарифная_Сетка"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.audit_tarif_function();
```

-- 5.4. Аудит для таблицы "Сотрудник"
```code
CREATE OR REPLACE FUNCTION kostuk.audit_sotrudnik_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            сотрудник_табельный_номер, сотрудник_фио, 
            сотрудник_разряд, сотрудник_отдел_id, 
            сотрудник_должность_id
        )
        VALUES (
            'U', 'Сотрудник', NEW.id,
            NEW.табельный_номер, NEW.ФИО, 
            NEW.разряд, NEW.отдел_id, 
            NEW.должность_id
        );
        RETURN NEW;
    ELSEIF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            сотрудник_табельный_номер, сотрудник_фио, 
            сотрудник_разряд, сотрудник_отдел_id, 
            сотрудник_должность_id
        )
        VALUES (
            'D', 'Сотрудник', OLD.id,
            OLD.табельный_номер, OLD.ФИО, 
            OLD.разряд, OLD.отдел_id, 
            OLD.должность_id
        );
        RETURN OLD;
    ELSEIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            сотрудник_табельный_номер, сотрудник_фио, 
            сотрудник_разряд, сотрудник_отдел_id, 
            сотрудник_должность_id
        )
        VALUES (
            'I', 'Сотрудник', NEW.id,
            NEW.табельный_номер, NEW.ФИО, 
            NEW.разряд, NEW.отдел_id, 
            NEW.должность_id
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER audit_sotrudnik_trigger
    AFTER INSERT OR UPDATE OR DELETE ON kostuk."Сотрудник"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.audit_sotrudnik_function();
```
-- 5.5. Аудит для таблицы "Табель"
```code
CREATE OR REPLACE FUNCTION kostuk.audit_tabel_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            табель_сотрудник_id, табель_год, 
            табель_месяц, табель_часы
        )
        VALUES (
            'U', 'Табель', NEW.id,
            NEW.сотрудник_id, NEW.год, 
            NEW.месяц, NEW.часы
        );
        RETURN NEW;
    ELSEIF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            табель_сотрудник_id, табель_год, 
            табель_месяц, табель_часы
        )
        VALUES (
            'D', 'Табель', OLD.id,
            OLD.сотрудник_id, OLD.год, 
            OLD.месяц, OLD.часы
        );
        RETURN OLD;
    ELSEIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Аудит_Общий"(
            операция, таблица_имя, запись_id, 
            табель_сотрудник_id, табель_год, 
            табель_месяц, табель_часы
        )
        VALUES (
            'I', 'Табель', NEW.id,
            NEW.сотрудник_id, NEW.год, 
            NEW.месяц, NEW.часы
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$;
```
```code
CREATE OR REPLACE TRIGGER audit_tabel_trigger
    AFTER INSERT OR UPDATE OR DELETE ON kostuk."Табель"
    FOR EACH ROW
    EXECUTE FUNCTION kostuk.audit_tabel_function();
```
-- 6. ПРОВЕРКА ТРИГГЕРОВ

-- 6.1. Проверка каскадного удаления
```code
DO $$
DECLARE
    v_count INTEGER;
BEGIN
    RAISE NOTICE '=== ПРОВЕРКА КАСКАДНОГО УДАЛЕНИЯ ===';
    
    -- Проверяем количество записей до удаления
    SELECT COUNT(*) INTO v_count FROM kostuk."Табель";
    RAISE NOTICE 'Табель до удаления: % записей', v_count;
    
    -- Удаляем одного сотрудника
    DELETE FROM kostuk."Сотрудник" WHERE id IN (SELECT id FROM kostuk."Сотрудник" LIMIT 1);
    
    -- Проверяем количество записей после удаления
    SELECT COUNT(*) INTO v_count FROM kostuk."Табель";
    RAISE NOTICE 'Табель после удаления: % записей', v_count;
    
    RAISE NOTICE '=== ПРОВЕРКА АУДИТА ===';
    
    -- Проверяем аудит
    SELECT COUNT(*) INTO v_count FROM kostuk."Аудит_Общий";
    RAISE NOTICE 'Записей в аудите: %', v_count;
    
    -- Покажем последние 5 записей аудита
    RAISE NOTICE 'Последние записи аудита:';
END;
$$;
```
-- 6.2. Показать последние записи аудита
```code
SELECT 
    аудит_id,
    операция,
    таблица_имя,
    запись_id,
    изменено_в,
    изменено_пользователем
FROM kostuk."Аудит_Общий" 
ORDER BY изменено_в DESC 
LIMIT 10;
```
-- 6.3. Проверка аудита INSERT/UPDATE/DELETE
```code
DO $$
DECLARE
    v_отдел_id INTEGER;
    v_сотрудник_id INTEGER;
BEGIN
    -- Тест INSERT
    INSERT INTO kostuk."Отдел" (название) 
    VALUES ('Тестовый отдел для аудита') 
    RETURNING id INTO v_отдел_id;
    
    RAISE NOTICE 'Создан отдел ID: %', v_отдел_id;
    
    -- Тест UPDATE
    UPDATE kostuk."Отдел" 
    SET название = 'Тестовый отдел (изменено)' 
    WHERE id = v_отдел_id;
    
    RAISE NOTICE 'Отдел обновлен';
    
    -- Тест DELETE (с каскадом)
    DELETE FROM kostuk."Отдел" WHERE id = v_отдел_id;
    
    RAISE NOTICE 'Отдел удален (с каскадом)';
END;
$$;
```
-- 6.4. Показать подробности аудита по отделам
```code
SELECT 
    аудит_id,
    операция,
    CASE операция 
        WHEN 'I' THEN 'INSERT'
        WHEN 'U' THEN 'UPDATE' 
        WHEN 'D' THEN 'DELETE'
    END as операция_текст,
    отдел_название,
    изменено_в,
    изменено_пользователем
FROM kostuk."Аудит_Общий" 
WHERE таблица_имя = 'Отдел'
ORDER BY изменено_в DESC 
LIMIT 5;
```
-- 7. ДОПОЛНИТЕЛЬНЫЕ ФУНКЦИИ ДЛЯ РАБОТЫ С АУДИТОМ

-- 7.1. Функция для получения истории изменений по таблице и ID
```code
CREATE OR REPLACE FUNCTION kostuk.получить_историю(
    p_таблица TEXT,
    p_запись_id INTEGER DEFAULT NULL
)
RETURNS TABLE (
    аудит_id INTEGER,
    операция TEXT,
    таблица_имя TEXT,
    запись_id INTEGER,
    изменено_в TIMESTAMP,
    изменено_пользователем TEXT,
    данные JSON
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        a.аудит_id,
        CASE a.операция 
            WHEN 'I' THEN 'INSERT'
            WHEN 'U' THEN 'UPDATE' 
            WHEN 'D' THEN 'DELETE'
        END,
        a.таблица_имя,
        a.запись_id,
        a.изменено_в,
        a.изменено_пользователем,
        CASE a.таблица_имя
            WHEN 'Отдел' THEN 
                json_build_object('название', a.отдел_название)
            WHEN 'Должность' THEN 
                json_build_object('название', a.должность_название)
            WHEN 'Тарифная_Сетка' THEN 
                json_build_object(
                    'должность_id', a.тариф_должность_id,
                    'разряд', a.тариф_разряд,
                    'ставка', a.тариф_ставка
                )
            WHEN 'Сотрудник' THEN 
                json_build_object(
                    'табельный_номер', a.сотрудник_табельный_номер,
                    'ФИО', a.сотрудник_фио,
                    'разряд', a.сотрудник_разряд,
                    'отдел_id', a.сотрудник_отдел_id,
                    'должность_id', a.сотрудник_должность_id
                )
            WHEN 'Табель' THEN 
                json_build_object(
                    'сотрудник_id', a.табель_сотрудник_id,
                    'год', a.табель_год,
                    'месяц', a.табель_месяц,
                    'часы', a.табель_часы
                )
        END as данные
    FROM kostuk."Аудит_Общий" a
    WHERE a.таблица_имя = p_таблица
        AND (p_запись_id IS NULL OR a.запись_id = p_запись_id)
    ORDER BY a.изменено_в DESC;
END;
$$ LANGUAGE plpgsql;
```
-- 7.2. Пример использования функции
```code
SELECT * FROM kostuk.получить_историю('Отдел') LIMIT 5;
```
-- 7.3. Функция для очистки старых записей аудита
```code
CREATE OR REPLACE FUNCTION kostuk.очистить_аудит_старше_дней(p_дней INTEGER DEFAULT 90)
RETURNS INTEGER AS $$
DECLARE
    v_удалено INTEGER;
BEGIN
    DELETE FROM kostuk."Аудит_Общий" 
    WHERE изменено_в < CURRENT_TIMESTAMP - (p_дней || ' days')::INTERVAL;
    
    GET DIAGNOSTICS v_удалено = ROW_COUNT;
    RETURN v_удалено;
END;
$$ LANGUAGE plpgsql;
```
-- Пример: очистить записи старше 30 дней
-- SELECT kostuk.очистить_аудит_старше_дней(30);

один ко многим
![40](https://github.com/user-attachments/assets/37982d5c-d4f4-4324-8ec9-5d8795eb62c4)

триггер аудита изменений
```code
CREATE OR REPLACE FUNCTION kostuk.триггер_аудит_изменений()
RETURNS TRIGGER AS $$
BEGIN
    IF (TG_OP = 'DELETE') THEN
        INSERT INTO kostuk."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_до
        )
        VALUES (
            TG_TABLE_NAME, 
            'DELETE', 
            OLD.id, 
            row_to_json(OLD)
        );
        RETURN OLD;
        
    ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO kostuk."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_до, 
            данные_после
        )
        VALUES (
            TG_TABLE_NAME, 
            'UPDATE', 
            NEW.id, 
            row_to_json(OLD), 
            row_to_json(NEW)
        );
        RETURN NEW;
        
    ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO kostuk."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_после
        )
        VALUES (
            TG_TABLE_NAME, 
            'INSERT', 
            NEW.id, 
            row_to_json(NEW)
        );
        RETURN NEW;
    END IF;
    
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```
журнал изменений
![41](https://github.com/user-attachments/assets/160734c4-364d-4dd3-bde1-f1df2ea43ce7)
![42](https://github.com/user-attachments/assets/1337f057-969e-4374-b909-ebfe1ac6f24d)
![43](https://github.com/user-attachments/assets/33bd3bea-dd62-4d41-a4ad-1beb7afde436)
![44](https://github.com/user-attachments/assets/621d3a53-7821-46c2-ba9c-27cecbc7cfa8)
