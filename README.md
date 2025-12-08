
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
