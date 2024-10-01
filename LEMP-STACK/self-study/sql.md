## Structured Query Language (SQL) Documentation

### Introduction
### SQL (Structured Query Language) is a powerful, standardized language for interacting with relational databases. It allows users to create, retrieve, update, and delete data within these databases. This guide offers an introduction to SQL syntax and covers commonly used commands.

## Basic Syntax
SQL commands follow a particular format that specifies the action to be executed. Here's an overview of key statements:

- SELECT Statement: Retrieves data from one or more tables.
```
    SELECT column1, column2, ... FROM table_name;
```

- INSERT Statement: Adds new rows to a table.
```
    INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);
```
UPDATE Statement: Modifies existing rows in a table.

```
    UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition;
```
- DELETE Statement: Removes rows from a table.

```
    DELETE FROM table_name WHERE condition;
```

- CREATE TABLE Statement: Defines and creates a new table in the database.

```
    CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
    );
```
- ALTER TABLE Statement: Alters the structure of an existing table by adding, removing, or modifying columns.

```
    ALTER TABLE table_name ADD column_name datatype;
```
- DROP TABLE Statement: Deletes a table permanently from the database.

```
    DROP TABLE table_name;
```

## Commonly Used Commands
Beyond the basic commands, SQL provides a rich set of tools for data manipulation and retrieval. Below are some key ones:

- SELECT: Retrieves data from a table. You can specify the columns and use a filter with the WHERE clause.
```
    SELECT * FROM employees;  -- Retrieve all columns from the 'employees' table.
```

- WHERE: Filters data based on a condition.
```
    SELECT * FROM employees WHERE department = 'HR';  -- Retrieve employees from the HR department.
```

- ORDER BY: Sorts the results in ascending or descending order based on a column.
```
    SELECT * FROM employees ORDER BY salary DESC;  -- Order employees by salary in descending order.
```
- GROUP BY: Groups rows by common values in a specific column, useful for performing summary calculations.
```
    SELECT department, COUNT(*) FROM employees GROUP BY department;  -- Count the number of employees in each department.
```
- JOIN: Combines rows from two or more tables based on a common column, enabling relational data queries.
```
    SELECT employees.name, departments.name FROM employees JOIN departments ON employees.department_id = departments.id;  -- Join employee names with department names.
```
- COUNT: An aggregate function that counts the total number of rows in the result set.
```
    SELECT COUNT(*) FROM employees;  -- Count the total number of employees.
```
- SUM, AVG, MIN, MAX: Aggregate functions that perform operations like calculating the total (SUM), average (AVG), minimum (MIN), and maximum (MAX) values.
```
    SELECT SUM(salary) FROM employees;  -- Calculate the total salary of all employees.
```

## Conclusion
This documentation covers the foundational syntax and commonly used SQL commands. By exploring more complex queries and practicing, you can gain a deeper understanding of SQL’s capabilities and unlock its full potential for database management and analysis.









