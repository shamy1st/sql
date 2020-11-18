# SQL
**Database** is a collection of data stored in a format that can easily be accessed.

**SEQUEL**: **S**tructured **E**nglish **Que**ry **L**anguage (intially by ibm in 1970's)

**SQL**: **S**tructured **Q**uery **L**anguage (changed, trade mark for an aeroplane company)

![](https://github.com/shamy1st/sql/blob/main/images/dbms.png)

## Retrieving Data From a Single Table

### SELECT

SELECT * FROM customers;

SELECT first_name, 
       last_name, 
       points, 
       (points + 10) * 100 AS 'discount factor'
FROM customers;

SELECT DISTINCT state FROM customers;

### WHERE

SELECT * FROM customers
WHERE birth_date > '1990-01-01';

* **AND, OR, NOT**

SELECT * FROM customers
WHERE NOT birth_date > '1990-01-01' OR (points > 1000 AND state = 'VA');

SELECT * , unit_price * quantity AS total_price FROM order_items
WHERE order_id = 6 AND unit_price * quantity > 10;

* **IN**






