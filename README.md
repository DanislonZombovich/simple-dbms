# Simple DBMS

Упрощенная реляционная система управления базами данных на языке Java 17 с хранением данных на диске, B-Tree индексированием, поддержкой SQL-запросов, журналом предзаписи (WAL), проверкой внешних ключей и консольным/графическим интерфейсами.

## Основные возможности

- **Типы данных и ограничения**: `INT` (Long), `DOUBLE` (Double), `VARCHAR` (String), `DATE` (ISO-8601 `YYYY-MM-DD`), `BOOLEAN` (true/false); поддержка `PRIMARY KEY`, `NOT NULL`, `REFERENCES <table>(<column>)`.
- **SQL-подобный язык запросов**:
  - DDL: `CREATE TABLE [IF NOT EXISTS]`, `DROP TABLE [IF EXISTS]`, `CREATE INDEX [IF NOT EXISTS] ... ON ...`, `DROP INDEX [IF EXISTS]`.
  - DML: `INSERT INTO` (одиночная и пакетная вставка, явный список столбцов), `SELECT` (`*` или выбор столбцов), `UPDATE ... SET ... WHERE`, `DELETE FROM ... WHERE`.
  - Условия `WHERE`: операторы `=`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `IS NULL`, `IS NOT NULL`, логические связки `AND` / `OR` с учетом приоритета и скобок `(...)`, однострочные комментарии `--`.
  - Диагностика: `SHOW TABLES`, `SHOW INDEXES [FROM|IN <table>]`, `DESCRIBE <table>` / `DESC <table>`, `EXPLAIN SELECT ...`.
- **Персистентность и надежность**:
  - Сериализация схем, строк и индексов в JSON (Jackson) в каталог `data/`.
  - Атомарное обновление файлов через временные `*.tmp.*` файлы.
  - Журнал предзаписи (`wal.log`), удаление незавершенных временных файлов, пропуск оборванных записей журнала и согласование индексов при старте после сбоя.
- **Индексирование**: сбалансированное B-дерево (`BTreeIndex`) для ускорения точечного и диапазонного поиска при наличии индекса по столбцу.
- **Интерфейсы**: интерактивная консоль (REPL) и графическое окно Swing.

## Сборка и запуск

Требуются JDK 17+ и Apache Maven 3.8+.

Сборка исполняемого JAR и запуск тестов:

```bash
mvn clean package
```

Генерация Javadoc:

```bash
mvn javadoc:javadoc
```

Запуск интерактивной консоли (CLI):

```bash
java -jar target/simple-dbms-1.0.0.jar
```

Дополнительные режимы и параметры:

```bash
# Указать каталог хранения данных (по умолчанию ./data)
java -jar target/simple-dbms-1.0.0.jar --data ./my_data

# Запустить графический интерфейс Swing
java -jar target/simple-dbms-1.0.0.jar --gui

# Создать демонстрационные таблицы manufacturers и cars и выйти
java -jar target/simple-dbms-1.0.0.jar --init-demo

# Запустить сравнение производительности SeqScan и B-Tree IndexScan
java -jar target/simple-dbms-1.0.0.jar --benchmark
```

## Примеры работы в CLI

Консольные команды (допускается завершать `;` или вводить без нее):
- `help` - справка по SQL и командам REPL;
- `init-demo` - создать демонстрационные таблицы `manufacturers` и `cars`;
- `tables` - показать все таблицы (`SHOW TABLES;`);
- `indexes` - показать созданные индексы (`SHOW INDEXES;`);
- `schema <таблица>` - показать столбцы и ключи таблицы (`DESCRIBE <таблица>;`);
- `benchmark` - выполнить тест производительности индексов;
- `exit` / `quit` - сохранить контрольную точку WAL и завершить работу.

Примеры SQL-запросов (каждый запрос оканчивается точкой с запятой `;`):

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
