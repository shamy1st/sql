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

Operator | Description
---------|------------
=        | Equal 	
\>       | Greater than 	
<        | Less than 	
\>=      | Greater than or equal 	
<=       | Less than or equal 	
<>       | Not equal. Note: In some versions of SQL this operator may be written as != 	
IN       | To specify multiple possible values for a column
BETWEEN  | Between a certain range 	
LIKE     | Search for a pattern 	

       SELECT * FROM customers
       WHERE birth_date > '1990-01-01';

* **AND, OR, NOT**

       SELECT * FROM customers
       WHERE NOT birth_date > '1990-01-01' OR (points > 1000 AND state = 'VA');

       SELECT * FROM order_items
       WHERE order_id = 6 AND unit_price * quantity > 10;

* **IN**

       SELECT * FROM customers
       WHERE state IN ('VA', 'FL', 'GA');

       SELECT * FROM customers
       WHERE state NOT IN ('VA', 'FL', 'GA');

* **BETWEEN**

       SELECT * FROM customers
       WHERE points >= 1000 AND points <= 3000;
       
       SELECT * FROM customers
       WHERE points BETWEEN 1000 AND 3000;

       SELECT * FROM customers
       WHERE birth_date BETWEEN '1990-01-01' AND '2000-01-01';

* **LIKE**
    * **%** : represents zero, one, or multiple characters.
    * **\_** : represents a single character.

              SELECT * FROM customers
              WHERE last_name LIKE 'b%';

              SELECT * FROM customers
              WHERE last_name LIKE 'b____y';

              SELECT * FROM customers
              WHERE phone NOT LIKE '%9';

* **REQEXP**

       -- last_name contains "field"
       SELECT * FROM customers
       -- WHERE last_name LIKE '%field%';
       WHERE last_name REGEXP 'field';

       -- last name starts with "brush"
       SELECT * FROM customers
       WHERE last_name REGEXP '^brush';

       -- matches last name contains "ge" or "ie" or "me"
       SELECT * FROM customers
       WHERE last_name REGEXP '[gim]e';

       -- last name contains "ae" or "be" or ... "he"
       SELECT * FROM customers
       WHERE last_name REGEXP '[a-h]e';

* **IS NULL**

       SELECT * FROM customers
       WHERE phone IS NULL;

       SELECT * FROM customers
       WHERE phone IS NOT NULL;

* **ORDER BY**

       SELECT * FROM customers
       ORDER BY first_name;

       SELECT * FROM customers
       ORDER BY first_name DESC;

       -- sort by state then first name
       SELECT * FROM customers
       ORDER BY state, first_name;

       -- order items for order=2 sorted by total price desc
       SELECT * , quantity * unit_price AS total_price 
       FROM order_items
       WHERE order_id = 2
       ORDER BY quantity * unit_price DESC;

       SELECT * , quantity * unit_price AS total_price 
       FROM order_items
       WHERE order_id = 2
       ORDER BY total_price DESC;

* **LIMIT**

       -- return only first 3 records
       SELECT * FROM customers
       LIMIT 3;

       -- skip first 6 records then get 3 records
       SELECT * FROM customers
       LIMIT 6, 3;

       -- top 3 customers points
       SELECT * FROM customers
       ORDER BY points DESC
       LIMIT 3;

## Retrieving Data From Multiple Tables

### Joins
![](https://github.com/shamy1st/sql/blob/main/images/joins.png)

* **(INNER) JOIN**: Returns records that have matching values in both tables
* **(OUTER) LEFT JOIN**: Returns all records from the left table, and the matched records from the right table
* **(OUTER) RIGHT JOIN**: Returns all records from the right table, and the matched records from the left table
* **(OUTER) FULL JOIN**: Returns all records when there is a match in either left or right table

### INNER JOIN
![](https://github.com/shamy1st/sql/blob/main/images/inner-join.png)

* INNER keyword is optional, you can write only **JOIN**

       SELECT order_id, o.customer_id, first_name, last_name
       FROM orders o
       JOIN customers c
       ON o.customer_id = c.customer_id;

       SELECT i.order_id, p.name, i.quantity, i.unit_price
       FROM order_items i
       JOIN products p
       ON i.product_id = p.product_id;

### Joining Across Databases

       SELECT *
       FROM sql_store.order_items i
       JOIN sql_inventory.products p
       ON i.product_id = p.product_id;

### Self Join

       SELECT e.employee_id, e.first_name, m.first_name AS manager
       FROM sql_hr.employees e
       JOIN sql_hr.employees m
       ON e.reports_to = m.employee_id;

### Join More Than Two Tables

       SELECT o.order_id, o.order_date, c.first_name, c.last_name, s.name AS status
       FROM orders o
       JOIN customers c ON o.customer_id = c.customer_id
       JOIN order_statuses s ON o.status = s.order_status_id;

### Compound Join Conditions
* tables with two primary keys

       SELECT * 
       FROM order_items i
       JOIN order_item_notes n
       ON i.order_id = n.order_id AND i.product_id = n.product_id;

### Implicit Join Syntax
* it is better not to use implicit syntax, because you may forget the where condition and make **cross join** instead!

       SELECT *
       FROM orders o, customers c
       WHERE o.customer_id = c.customer_id;

       -- same as

       SELECT *
       FROM orders o
       JOIN customers c
       ON o.customer_id = c.customer_id;

### OUTER JOIN

* **LEFT JOIN** return all records from left table, regardless the condition is true or not.

       SELECT c.customer_id, c.first_name, o.order_id
       FROM customers c
       LEFT JOIN orders o
       ON o.customer_id = c.customer_id;

### Outer Join Between More Than Two Tables

       SELECT c.customer_id, c.first_name, o.order_id, s.name AS shipper
       FROM customers c
       LEFT JOIN orders o ON o.customer_id = c.customer_id
       LEFT JOIN shippers s ON o.shipper_id = s.shipper_id;

       SELECT o.order_date, o.order_id, c.first_name AS customer, sh.name AS shipper, s.name AS status
       FROM orders o
       LEFT JOIN customers c ON o.customer_id = c.customer_id
       LEFT JOIN shippers sh ON o.shipper_id = sh.shipper_id
       LEFT JOIN order_statuses s ON o.status = s.order_status_id
       ORDER BY status;

### Self Outer Joins

       -- return the manager with employees and null manager
       SELECT e.employee_id, e.first_name, m.first_name AS manager
       FROM sql_hr.employees e
       LEFT JOIN sql_hr.employees m
       ON e.reports_to = m.employee_id;

### USING Clause
* if the joining column is the same name as primary key column, you can use **USING** clause.

       SELECT o.order_id, c.first_name
       FROM orders o
       JOIN customers c
       USING(customer_id);
       
       -- same as

       SELECT o.order_id, c.first_name
       FROM orders o
       JOIN customers c
       ON o.customer_id = c.customer_id;

* **with two primary keys**

       SELECT * 
       FROM order_items i
       JOIN order_item_notes n
       USING(order_id, product_id);

       -- same as

       SELECT * 
       FROM order_items i
       JOIN order_item_notes n
       ON i.order_id = n.order_id AND i.product_id = n.product_id;

### Natural Joins
* not recommended, sometimes it produce unexpected results.
* the engine expect the join columns.

       SELECT o.order_id, c.first_name
       FROM orders o
       NATURAL JOIN customers c;

       -- same as

       SELECT o.order_id, c.first_name
       FROM orders o
       JOIN customers c
       ON o.customer_id = c.customer_id;

### Cross Joins
* every record from each table are combined with all records from the other table.
* so no need for condition or any columns to join.
* the actual example when you have table of sizes (small, medium, large) and table of colors.

* **explicit syntax**

       SELECT c.first_name AS customer, p.name AS product
       FROM customers c
       CROSS JOIN products p;

* **implicit syntax**

       SELECT c.first_name AS customer, p.name AS product
       FROM customers c, products p;

### Unions
* the columns should be the same

       SELECT first_name FROM customers
       UNION
       SELECT name FROM shippers;


       SELECT order_id, order_date, 'Active' AS status
       FROM sql_store.orders
       WHERE order_date >= '2019-01-01'

       UNION

       SELECT order_id, order_date, 'Archived' AS status
       FROM sql_store.orders
       WHERE order_date < '2019-01-01';

## Inserting, Updating, and Deleting Data

### Column Attributes

* **VARCHAR vs CHAR** varchar is variable length and store only used length, but char is fixed length.

### INSERT

* **all columns**

       INSERT INTO `customers` VALUES (DEFAULT,'John','Smith','1990-01-01',NULL,'address','city','CA',DEFAULT);

* **specify columns**

       INSERT INTO customers (
              first_name,
              last_name,
              birth_date,
              address,
              city,
              state
       )
       VALUES (
              'John', 
              'Smith', 
              '1990-01-01'
              'address',
              'city',
              'CA');

* **multiple rows in single statement**

       -- insert 3 rows
       INSERT INTO shippers (name)
       VALUES ('Shipper1'),
              ('Shipper2'),
              ('Shipper3');

       -- insert 3 rows
       INSERT INTO products (name, quantity_in_stock, unit_price)
       VALUES ('Product1', 10, 1.95),
              ('Product2', 11, 1.95),
              ('Product3', 12, 1.95);

* **hierarchical rows** with LAST_INSERT_ID() function

       INSERT INTO orders (customer_id, order_date, status)
       VALUES (1, '2019-01-02', 1);

       INSERT INTO order_items
       VALUES (LAST_INSERT_ID(), 1, 1, 2.95),
                 (LAST_INSERT_ID(), 2, 1, 3.95);

### Copy Table Data to Another Table

       CREATE TABLE orders_archived AS
       SELECT * FROM orders;

       CREATE TABLE orders_archived
       SELECT * FROM orders
       WHERE order_date < '2019-01-01';

### UPDATE

* **single record**

       UPDATE invoices
       SET payment_total = 10, payment_date = '2019-03-01'
       WHERE invoice_id = 1;
       
       UPDATE invoices
       SET payment_total = invoice_total * 0.5, payment_date = due_date
       WHERE invoice_id = 3;

* **multiple records**

       UPDATE invoices
       SET payment_total = invoice_total * 0.5, payment_date = due_date
       WHERE client_id = 3;

       UPDATE customers
       SET points = points * 1.5
       WHERE birth_date > '1990-01-01';

* **using subqueries in updates**

       UPDATE invoices
       SET payment_total = invoice_total * 0.5, payment_date = due_date
       WHERE client_id = (SELECT client_id FROM clients WHERE name = 'Myworks');

       UPDATE invoices
       SET payment_total = invoice_total * 0.5, payment_date = due_date
       WHERE client_id IN (SELECT client_id FROM clients WHERE state IN ('CA', 'NY'));

### DELETE Rows

       DELETE FROM invoices
       WHERE invoice_id = 1;

       DELETE FROM invoices
       WHERE client_id = (SELECT client_id FROM clients WHERE name = 'Myworks');

### Restoring the Databases
* run the script create-databases.sql which drops all tables and recreate them again.


## Summarizing Data

### Aggregate Functions
operate only on non NULL values, so if you have null value it will not be included in these functions.

**built in functions**:
* **MAX()**

       SELECT MAX(invoice_total) AS maximum FROM invoices;
       SELECT MAX(payment_date) AS latest_date FROM invoices;

* **MIN()**

       SELECT MIN(invoice_total) AS minimum FROM invoices;
       SELECT MIN(payment_date) AS oldest_date FROM invoices;

* **AVG()**

       SELECT AVG(invoice_total) AS average FROM invoices;

* **SUM()**

       SELECT SUM(invoice_total) AS summation FROM invoices;
       -- multiply each value by 1.1 then apply sum
       SELECT SUM(invoice_total * 1.1) AS summation FROM invoices;
       -- sum all values then multiply by 1.1
       SELECT SUM(invoice_total) * 1.1 AS summation FROM invoices;

* **COUNT()**

       -- if null records it will not count
       SELECT COUNT(invoice_total) AS counter FROM invoices;
       -- the actual total records of table
       SELECT COUNT(*) AS total_records FROM invoices;
       -- count distinct number of clients
       SELECT COUNT(DISTINCT client_id) AS client_records FROM invoices;

### GROUP BY

* **single column**

       -- total sales for each client
       SELECT client_id, SUM(invoice_total) AS total_sales 
       FROM invoices
       WHERE invoice_date >= '2019-07-01'
       GROUP BY client_id
       ORDER BY total_sales DESC;

* **multiple columns**

       SELECT c.state, c.city, SUM(i.invoice_total) AS total_sales 
       FROM invoices i
       JOIN clients c USING (client_id)
       GROUP BY c.state, c.city;

### HAVING
you can't use columns that not included in the select clause

       SELECT client_id, SUM(invoice_total) AS total_sales 
       FROM invoices
       GROUP BY client_id
       HAVING total_sales > 500;

       SELECT client_id, SUM(invoice_total) AS total_sales, COUNT(*) AS number_of_invoices 
       FROM invoices
       GROUP BY client_id
       HAVING total_sales > 500 AND number_of_invoices > 5;

       -- customers located in virginia and spent more than $100
       SELECT c.customer_id, c.first_name, c.last_name, c.state, SUM(i.quantity * i.unit_price) AS total_payment
       FROM customers c
       JOIN orders o USING (customer_id)
       JOIN order_items i USING (order_id)
       WHERE c.state = 'VA'
       GROUP BY c.customer_id
       HAVING total_payment > 100;

### ROLLUP
only available in MYSQL

       -- sum the total of sums in a separate row
       SELECT client_id, SUM(invoice_total) AS total_sales 
       FROM invoices
       GROUP BY client_id WITH ROLLUP


## Writing Complex Query

### Subqueries

       -- products more expensive than product 3
       SELECT * FROM products
       WHERE unit_price > (SELECT unit_price FROM products WHERE product_id = 3);

       -- employees earn more than average
       SELECT * FROM employees 
       WHERE salary > (SELECT AVG(salary) FROM employees);

### IN Operator

       -- all products not ordered before
       SELECT * FROM products
       WHERE product_id NOT IN (SELECT DISTINCT product_id FROM order_items);

       -- client without invoices
       SELECT * FROM clients
       WHERE client_id NOT IN (SELECT DISTINCT client_id FROM invoices);

## Subqueries vs Joins

       -- client without invoices using subquery
       SELECT * FROM clients
       WHERE client_id NOT IN (SELECT DISTINCT client_id FROM invoices);

       -- client without invoices using join
       SELECT * 
       FROM clients c
       LEFT JOIN invoices i USING (client_id)
       WHERE invoice_id IS NULL;

### ALL keyword

       -- all invoices larger than all invoices of customer 3
       SELECT * FROM invoices
       WHERE invoice_total > (SELECT MAX(invoice_total) FROM invoices WHERE client_id = 3);

       -- all invoices larger than all invoices of customer 3 - using ALL keyword
       SELECT * FROM invoices
       WHERE invoice_total > ALL (SELECT invoice_total FROM invoices WHERE client_id = 3);

### ANY keyword
'= ANY' same as 'IN' keyword 

       -- clients with at least two invoices
       SELECT * FROM clients
       WHERE client_id IN (SELECT client_id FROM invoices GROUP BY client_id HAVING COUNT(*) > 1);

       -- clients with at least two invoices using ANY keywor
       SELECT * FROM clients
       WHERE client_id = ANY (SELECT client_id FROM invoices GROUP BY client_id HAVING COUNT(*) > 1);

### Correlated Subqueries

       -- employees whose salary is above the average in their offices
       SELECT * 
       FROM employees e
       WHERE salary > (
              SELECT AVG(salary) 
              FROM employees
              WHERE office_id = e.office_id
       );

### EXISTS Operator
more efficient than 'IN' operator

       -- clients that have an invoice
       SELECT * FROM clients
       WHERE client_id IN (SELECT DISTINCT client_id FROM invoices);

       -- clients that have an invoice using EXISTS
       SELECT * FROM clients c
       WHERE EXISTS (SELECT client_id FROM invoices WHERE client_id = c.client_id);

### Subqueries in the SELECT Clause

       SELECT invoice_id, 
              invoice_total,
              (SELECT AVG(invoice_total) FROM invoices) AS invoice_average,
              invoice_total - (SELECT invoice_average) AS difference
       FROM invoices;

### Subqueries in the FROM Clause
should give the query which represent a table an alias like 'invoice_summary'

       SELECT * FROM (
              SELECT invoice_id, 
                     invoice_total,
                     (SELECT AVG(invoice_total) FROM invoices) AS invoice_average,
                     invoice_total - (SELECT invoice_average) AS difference
              FROM invoices
       ) AS invoice_summary;


## Essential MySQL Functions

### Numeric Functions
https://dev.mysql.com/doc/refman/8.0/en/numeric-functions.html

* **ROUND()**

       -- return 5.74
       SELECT ROUND(5.7367, 2);

* **TRUNCATE()**

       -- return 5.73
       SELECT TRUNCATE(5.7367, 2);

* **CEILING()**

       -- return 6
       SELECT CEILING(5.1);

* **FLOOR()**

       -- return 5
       SELECT FLOOR(5.8);

* **ABS()**

       -- absolute value, return 5.3
       SELECT ABS(-5.3);

* **RAND()**

       -- generate random value from 0 to 1
       SELECT RAND();

### String Functions
https://dev.mysql.com/doc/refman/8.0/en/string-functions.html

* **LENGTH()**

       -- return 3
       SELECT LENGTH('sky');

* **UPPER()**

       -- uppercase, return SKY
       SELECT UPPER('sky');

* **LOWER()**

       -- lowercase, return sea
       SELECT LOWER('SEA');

* **LTRIM()**

       -- left trim, return 'hello'
       SELECT LTRIM('  hello  ');

* **RTRIM()**

       -- right trim, return 'hello'
       SELECT RTRIM('hello  ');

* **TRIM()**

       -- return 'hello'
       SELECT TRIM('  hello  ');

* **LEFT()**

       -- return 'fant'
       SELECT LEFT('fantastic', 4);

* **RIGHT()**

       -- return 'stic'
       SELECT RIGHT('fantastic', 4);

* **SUBSTRING()**

       -- start position (count from 1), length of substring (optional, without it then to the end), return 'ant'
       SELECT SUBSTRING('fantastic', 2, 3);

* **LOCATE()**

       -- search about substring and return position (count from 1), return 5
       -- return 0 if not found
       SELECT LOCATE('ast', 'fantastic');

* **REPLACE()**

       -- replace 'ant' with 'and', return 'fandastic'
       SELECT REPLACE('fantastic', 'ant', 'and');

* **CONCAT()**

       -- return 'startend'
       SELECT CONCAT('start', 'end');

### Date Functions
https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html

* **NOW()**

       -- return '2020-11-20 20:25:13'
       SELECT NOW();

* **CURDATE()**

       -- return '2020-11-20'
       SELECT CURDATE();

* **CURTIME()**

       -- return '20:27:10'
       SELECT CURTIME();

* **YEAR() MONTH() DAY() HOUR() MINUTE() SECOND()**

       -- return '2020'
       SELECT YEAR(NOW());

* **DAYNAME() MONTHNAME()**

       -- return 'Friday'
       SELECT DAYNAME(NOW());

* **EXTRACT(... FROM ...) YEAR, MONTH, DAY**

       -- return '2020'
       SELECT EXTRACT(YEAR FROM NOW());

### Formatting Dates and Times
https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_date-format

       -- %y -> two digit year, %Y -> 4 digit year
       -- %m -> two digit month, %M -> month name
       -- %d -> two digit day, %D -> 20th
       SELECT DATE_FORMAT(NOW(), '%M %d, %Y');

       -- return '20:46 PM'
       SELECT TIME_FORMAT(NOW(), '%H:%i %p');

### Calculating Dates and Times

* **ADD, SUB**

       -- add 1 day to now, return '2020-11-21 20:48:17'
       SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);
       
       -- subtract 1 day to now, return '2020-11-19 20:50:12'
       SELECT DATE_SUB(NOW(), INTERVAL 1 DAY);

       -- add 1 year to now, return '2021-11-20 20:49:16'
       SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR);

       -- subtract 1 year to now, return '2019-11-20 20:50:46'
       SELECT DATE_SUB(NOW(), INTERVAL 1 YEAR);

* **DIFF**

       -- return 4
       SELECT DATEDIFF('2020-01-05', '2020-01-01');

       -- TIME_TO_SEC(): number of sec since mid-night
       SELECT TIME_TO_SEC('09:00') - TIME_TO_SEC('08:59');

### IFNULL and COALESCE Functions

































































