# Simple DBMS

Упрощенная реляционная СУБД на Java 17 с хранением данных на диске, B-Tree индексами, журналом предзаписи (WAL), проверкой внешних ключей и двумя интерфейсами - консольным и Swing.

## Возможности

Система поддерживает пять типов столбцов: `INT` (`Long`), `DOUBLE` (`Double`), `VARCHAR` (`String`), `DATE` (ISO-8601 `YYYY-MM-DD`) и `BOOLEAN` (`true`/`false`). При создании таблиц можно задавать ограничения `PRIMARY KEY`, `NOT NULL` и внешние ключи `REFERENCES <table>(<column>)`.

Из SQL-команд доступны:
- DDL: `CREATE TABLE [IF NOT EXISTS]`, `DROP TABLE [IF EXISTS]`, `CREATE INDEX [IF NOT EXISTS] ... ON ...`, `DROP INDEX [IF EXISTS]`;
- DML: `INSERT INTO` (по одной строке, с явным списком столбцов или пачкой через запятую), `SELECT` (`*` или выбранные столбцы), `UPDATE ... SET ... WHERE`, `DELETE FROM ... WHERE`;
- фильтрация `WHERE`: операторы `=`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `IS NULL`, `IS NOT NULL`, логические связки `AND` и `OR` с учетом приоритета и скобок `(...)`, а также однострочные комментарии `--`;
- служебные запросы: `SHOW TABLES`, `SHOW INDEXES [FROM|IN <table>]`, `DESCRIBE <table>` / `DESC <table>` и `EXPLAIN SELECT ...`.

Таблицы, схемы и индексы сериализуются через Jackson в каталог `data/` с записью через временные файлы `*.tmp.*` и атомарным переименованием. Операции изменения фиксируются в журнале `wal.log`. При запуске после сбоя система удаляет оборванные временные файлы, читает журнал (пропуская поврежденный хвост) и сверяет индексы с таблицами. Для ускорения запросов по индексированным столбцам используется B-дерево (`BTreeIndex`).

## Сборка и запуск

Для работы требуются JDK 17+ и Apache Maven 3.8+.

Сборка исполняемого JAR с зависимостями и прогон тестов:

```bash
mvn clean package
```

Генерация документации Javadoc:

```bash
mvn javadoc:javadoc
```

Запуск интерактивной консоли (данные по умолчанию сохраняются в `./data`):

```bash
java -jar target/simple-dbms-1.0.0.jar
```

Дополнительные флаги запуска:

```bash
# Указать свой каталог для хранения базы
java -jar target/simple-dbms-1.0.0.jar --data ./my_data

# Открыть графический интерфейс Swing
java -jar target/simple-dbms-1.0.0.jar --gui

# Создать демонстрационные таблицы manufacturers и cars и сразу выйти
java -jar target/simple-dbms-1.0.0.jar --init-demo

# Запустить сравнение скорости SeqScan и B-Tree IndexScan
java -jar target/simple-dbms-1.0.0.jar --benchmark
```

## Работа в консоли и примеры SQL

Помимо SQL, в консоли есть служебные команды (точку с запятой после них ставить не обязательно):
- `help` показывает справку по SQL и командам REPL;
- `init-demo` создает демонстрационные таблицы `manufacturers` и `cars`;
- `tables` выводит список таблиц (`SHOW TABLES;`);
- `indexes` выводит список индексов (`SHOW INDEXES;`);
- `schema <таблица>` показывает столбцы и ключи таблицы (`DESCRIBE <таблица>;`);
- `benchmark` запускает тест производительности индексов;
- `exit` или `quit` сбрасывает контрольную точку WAL и завершает программу.

SQL-запросы завершаются точкой с запятой `;`:

```sql
CREATE TABLE manufacturers (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR
);

CREATE TABLE cars (
    id INT PRIMARY KEY,
    brand VARCHAR NOT NULL,
    price DOUBLE,
    is_available BOOLEAN,
    created_at DATE,
    manufacturer_id INT REFERENCES manufacturers(id)
);

CREATE INDEX idx_cars_brand ON cars (brand);

INSERT INTO manufacturers VALUES (1, 'Toyota Motor Corp', 'Japan');
INSERT INTO cars VALUES (1, 'Toyota', 25000.0, TRUE, '2024-01-15', 1);
INSERT INTO cars (id, brand) VALUES (2, 'BMW');

SELECT * FROM cars;
SELECT brand, price FROM cars WHERE price > 20000;
EXPLAIN SELECT * FROM cars WHERE brand = 'Toyota';

UPDATE cars SET price = 30000 WHERE id = 2;
DELETE FROM cars WHERE brand = 'Toyota';
```
