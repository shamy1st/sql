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

* **IFNULL()**

       SELECT 
              order_id, 
              IFNULL(shipper_id, 'Not assigned') AS shipper
       FROM orders;

* **COALESCE()**

       -- COALESCE: if shipper_id is null return comments and if comments null return 'Not assigned'
       SELECT 
              order_id, 
              COALESCE(shipper_id, comments, 'Not assigned') AS shipper
       FROM orders;

### IF Function

       SELECT
           order_id,
           order_date,
           IF(YEAR(order_date) = YEAR(NOW()), 'Active', 'Archived') AS status
       FROM orders;
       
       SELECT 
              p.product_id, 
              p.name, 
              COUNT(*) AS orders, 
              IF(COUNT(*) = 1, 'Once', 'Many Times') AS frequency
       FROM products p
       JOIN order_items i USING (product_id)
       GROUP BY product_id;       

### CASE Operator

       SELECT
           order_id,
           order_date,
           CASE
               WHEN YEAR(order_date) = YEAR(NOW()) THEN 'Active'
               WHEN YEAR(order_date) = YEAR(NOW()) - 1 THEN 'Last Year'
               WHEN YEAR(order_date) < YEAR(NOW()) - 1 THEN 'Archived'
               ELSE 'Future'
           END AS category
       FROM orders;

       SELECT
           CONCAT(first_name, ' ', last_name) AS customer,
           points,
           CASE
               WHEN points > 3000 THEN 'Gold'
               WHEN points >= 2000 THEN 'Silver'
               ELSE 'Bronze'
           END AS category
       FROM customers;


## Views

### Creating Views

       CREATE OR REPLACE VIEW sales_by_client AS
       SELECT c.client_id, c.name, SUM(i.invoice_total) AS total_sales
       FROM clients c
       JOIN invoices i USING (client_id)
       GROUP BY client_id
       WITH CHECK OPTION;

       SELECT * FROM sales_by_client;

### Altering or Dropping Views

       DROP VIEW sales_by_client;

       -- edit
       CREATE OR REPLACE VIEW sales_by_client AS
       SELECT c.client_id, c.name, SUM(i.invoice_total) AS total_sales
       FROM clients c
       JOIN invoices i USING (client_id)
       GROUP BY client_id
       WITH CHECK OPTION;

### Updatable Views
if you don't have the following keywords in your view query then you can delete, update, insert
* DISTINCT
* Aggregate Function (MIN, MAX, SUM, AVG, ...)
* GROUP BY / HAVING
* UNION

       DELETE FROM invoices_with_balance WHERE invoice_id = 1;
       
       UPDATE invoices_with_balance SET due_date = DATE_ADD(due_date, INTERVAL 2 DAY) WHERE invoice_id = 2;

       INSERT (little bit tricky)

### WITH OPTION CHECK Clause

       -- the row with invoice_id = 2, will be discarded after run this update
       UPDATE invoices_with_balance SET payment_total = invoice_total WHERE invoice_id = 2;

       -- add 'WITH CHECK OPTION' at the end of the query
       -- if something happen as previous update, error will be shown in console without discard row.
       CREATE OR REPLACE VIEW invoices_with_balance AS
       SELECT 
           invoice_id,
           number,
           client_id,
           invoice_total,
           payment_total,
           invoice_total - payment_total AS balance,
           invoice_date,
           due_date,
           payment_date
       FROM invoices
       WHERE (invoice_total - payment_total) > 0
       WITH CHECK OPTION;


## Stored Procedures

### Creating a Stored Procedure

       DELIMITER $$
       CREATE PROCEDURE get_clients()
       BEGIN
              SELECT * FROM clients;
       END$$
       DELIMITER ;

       CALL sql_invoicing.get_clients();

### Dropping Stored Procedures

       DROP PROCEDURE IF EXISTS get_clients;

### Template for Git

       DROP PROCEDURE IF EXISTS get_clients;

       DELIMITER $$
       CREATE PROCEDURE get_clients()
       BEGIN
              SELECT * FROM clients;
       END$$
       DELIMITER ;

### Parameters

       DROP PROCEDURE IF EXISTS get_clients_by_state;

       DELIMITER $$
       CREATE PROCEDURE get_clients_by_state(state CHAR(2))
       BEGIN
              SELECT * FROM clients c WHERE c.state = state;
       END$$
       DELIMITER ;

       CALL sql_invoicing.get_clients_by_state('CA');

       --

       DROP PROCEDURE IF EXISTS get_invoices_by_client;

       DELIMITER $$
       CREATE PROCEDURE get_invoices_by_client(client_id INT)
       BEGIN
              SELECT * FROM invoices i WHERE i.client_id = client_id;
       END$$
       DELIMITER ;

       CALL sql_invoicing.get_invoices_by_client(5);

### Parameters with Default Value

       DROP PROCEDURE IF EXISTS get_clients_by_state;

       DELIMITER $$
       CREATE PROCEDURE get_clients_by_state(state CHAR(2))
       BEGIN
              SELECT * FROM clients c WHERE c.state = IFNULL(state, c.state);
       END$$
       DELIMITER ;

       CALL sql_invoicing.get_clients_by_state(NULL);

       --

       DROP PROCEDURE IF EXISTS get_payments;

       DELIMITER $$
       CREATE PROCEDURE get_payments(
       client_id INT,
       payment_method_id TINYINT
       )
       BEGIN
              SELECT * 
           FROM payments p 
           WHERE p.client_id = IFNULL(client_id, p.client_id) AND 
                       p.payment_method = IFNULL(payment_method_id, p.payment_method);
       END$$
       DELIMITER ;

       CALL sql_invoicing.get_payments(5, 2);

### Parameter Validation

[SQLSTATE VALUES](https://www.ibm.com/support/knowledgecenter/SSEPEK_11.0.0/codes/src/tpc/db2z_sqlstatevalues.html)

* **22003**: A numeric value is out of range.

       DROP PROCEDURE IF EXISTS update_invoice;

       DELIMITER $$
       CREATE PROCEDURE update_invoice(
              invoice_id INT,
              payment_total DECIMAL(9,2),
              payment_date DATE
       )
       BEGIN
              IF payment_total <= 0 THEN
                     SIGNAL SQLSTATE '22003' SET MESSAGE_TEXT = 'Invalid payment_total!';
              END IF;

              UPDATE invoices i 
              SET i.payment_total = payment_total, i.payment_date = payment_date
              WHERE i.invoice_id = invoice_id;
       END$$
       DELIMITER ;

       CALL sql_invoicing.update_invoice(2, -100, '2019-01-01');

### Output Parameters

       DROP PROCEDURE IF EXISTS get_unpain_invoices_for_client;

       DELIMITER $$
       CREATE PROCEDURE get_unpain_invoices_for_client(
           client_id INT,
           OUT invoices_count INT,
           OUT invoices_total DECIMAL(9,2)
       )
       BEGIN
           SELECT COUNT(*), SUM(invoice_total)
           INTO invoices_count, invoices_total
           FROM invoices i 
           WHERE i.client_id = client_id AND i.payment_total = 0;
       END$$
       DELIMITER ;

       SET @invoices_count = 0;
       SET @invoices_total = 0;
       CALL sql_invoicing.get_unpain_invoices_for_client(3, @invoices_count, @invoices_total);
       SELECT @invoices_count, @invoices_total;

### Variables

       -- output parameter
       -- user or session variables
       -- will be in memory during the user session, when user disconnect from mysql it will be free
       SET @invoices_count = 0;

       -- local variable (defined inside stored procedure).
       -- go out of memory after procedure finished.
       DECLARE risk_factor DECIMAL(9,2) DEFAULT 0;


       DROP PROCEDURE IF EXISTS get_risk_factor;

       DELIMITER $$
       CREATE PROCEDURE get_risk_factor()
       BEGIN
           DECLARE risk_factor DECIMAL(9,2) DEFAULT 0;
           DECLARE invoices_total DECIMAL(9,2);
           DECLARE invoices_count INT;

           SELECT COUNT(*), SUM(invoice_total)
           INTO invoices_count, invoices_total
           FROM invoices;

           SET risk_factor = invoices_total / invoices_count * 5;

           SELECT risk_factor;
       END$$
       DELIMITER ;

       CALL get_risk_factor();


## Functions
custom function with single return value like MIN(), MAX(), SUM(), ...

### CREATE Function

* **Attributes**: the function at least have one attribute

    * **DETERMINISTIC**: for the same input return the same output.
    * **READS SQL DATA**: use select statements.
    * **MODIFIES SQL DATA**: use modify statements (update, delete, drop, ...).

              DROP FUNCTION IF EXISTS get_risk_factor_for_client;

              DELIMITER $$
              CREATE FUNCTION get_risk_factor_for_client(client_id INT)
              RETURNS INTEGER
              READS SQL DATA
              BEGIN
                     DECLARE risk_factor DECIMAL(9,2) DEFAULT 0;
                     DECLARE invoices_total DECIMAL(9,2);
                     DECLARE invoices_count INT;

                     SELECT COUNT(*), SUM(invoice_total)
                     INTO invoices_count, invoices_total
                     FROM invoices i
                     WHERE i.client_id = client_id;

                     SET risk_factor = invoices_total / invoices_count * 5;

                     RETURN IFNULL(risk_factor, 0);
              END$$
              DELIMITER ;

              SELECT client_id, name, get_risk_factor_for_client(client_id) AS risk_factor FROM clients;


## Triggers and Events

### Trigger

* **Trigger** is a block of code that automatically executed before or after an insert, update or delete statement.
* you can trigger any table except the changed table.

       DROP TRIGGER IF EXISTS payments_after_insert;

       DELIMITER $$
       CREATE TRIGGER payments_after_insert
       AFTER INSERT ON payments
       FOR EACh ROW
       BEGIN
           UPDATE invoices
           SET payment_total = payment_total + NEW.amount
           WHERE invoice_id = NEW.invoice_id;
       END$$
       DELIMITER ;

       INSERT INTO payments VALUES (DEFAULT,5,3,'2019-01-01',10,1);
       SELECT * FROM invoices WHERE invoice_id = 3;

       --

       DROP TRIGGER IF EXISTS payments_after_delete;

       DELIMITER $$
       CREATE TRIGGER payments_after_delete
       AFTER DELETE ON payments
       FOR EACh ROW
       BEGIN
              UPDATE invoices
           SET payment_total = payment_total - OLD.amount
           WHERE invoice_id = OLD.invoice_id;
       END$$
       DELIMITER ;

       DELETE FROM payments WHERE payment_id = 9;
       SELECT * FROM invoices WHERE invoice_id = 3;

* **View Triggers**

       SHOW TRIGGERS;

* **Dropping Trigger**

       DROP TRIGGER IF EXISTS payments_after_delete;

* **Using Triggers for Auditing** create payments_audit table and each trigger insert row (insert, delete, update, ...)

### Event

* **Event**: is a task or (block of SQL code) executed according to a schedule.

       -- all mysql system variables
       SHOW VARIABLES;

       -- for event_scheduler system variable
       SHOW VARIABLES LIKE 'event%';
       -- set event_scheduler = ON
       SET GLOBAL event_scheduler = ON;

* **Create Event**

       DROP EVENT IF EXISTS yearly_delete_audit_rows;

       DELIMITER $$
       CREATE EVENT yearly_delete_audit_rows
       ON SCHEDULE
              -- AT '2019-05-01'
           EVERY 1 YEAR STARTS '2021-01-01' ENDS '2029-01-01'
       DO BEGIN
              DELETE FROM payments_audit
           WHERE action_date < Now() - INTERVAL 1 YEAR;
       END$$
       DELIMITER ;

* **View Events**

       SHOW EVENTS;

* **Enable/Disable Event**

       ALTER EVENT yearly_delete_audit_rows DISABLE;
       ALTER EVENT yearly_delete_audit_rows ENABLE;


## Transactions and Concurrency

### Transaction

* **Transaction** is a group of SQL statements that represent a single unit of work.

* **ACID Properties**
  * **ATOMICITY** like atom is unbreakable.
  * **CONSISTENCY** data must be at consistent state.
  * **ISOLATION** each transaction isolated from each other.
  * **Durability** once it is committed, changes will be permanent.
  
* **Create Transaction**

       START TRANSACTION;

       INSERT INTO orders VALUES (DEFAULT,1,'2019-01-01',1,NULL,NULL,NULL);

       INSERT INTO `order_items` VALUES (LAST_INSERT_ID(),1,1,1);

       COMMIT;

* **autocommit**
mysql wrap (insert, update, delete) statements in transaction and autocommit it. autocommit variable is ON by default.

       SHOW VARIABLES LIKE 'autocommit%';

### Concurrency and Lockign
If two transactions update the same row, mysql lock the row for the second one and wait unitl first one finish.

### Concurrency Problems

1. **LOST UPDATES** (two transactions update the same row)
![](https://github.com/shamy1st/sql/blob/main/images/lost-updates.png)

2. **DIRTY READS** (read uncommitted data)
![](https://github.com/shamy1st/sql/blob/main/images/dirty-reads-1.png)
![](https://github.com/shamy1st/sql/blob/main/images/dirty-reads-2.png)

3. **NON-REPEATING READS** (read the same data in the same transaction with different results)
![](https://github.com/shamy1st/sql/blob/main/images/non-repeating-1.png)
![](https://github.com/shamy1st/sql/blob/main/images/non-repeating-2.png)

4. **PHANTOM READS** (miss one or more row in our query, because another transaction update it [ghost])
![](https://github.com/shamy1st/sql/blob/main/images/phantom-reads-1.png)
![](https://github.com/shamy1st/sql/blob/main/images/phantom-reads-2.png)

### Transaction Isolation Levels
mysql by default is set to **REPEATABLE READ**
![](https://github.com/shamy1st/sql/blob/main/images/transaction-isolation-levels.png)

       -- by default transaction_isolation = REPEATABLE-READ
       SHOW VARIABLES LIKE 'transaction_isolation';
       
       SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
       SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
       SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
       SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

       -- SESSION: only for this session
       SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
       
       -- GLOBAL: for all upcoming transaction in the future
       SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

* **Isolation Levels**
1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE

### DEADLOCK
two transactions waiting each others to release the lock.

## Data Types

[SQL Data Types for MySQL, SQL Server, MS Access](https://www.w3schools.com/sql/sql_datatypes.asp)

### String Type
![](https://github.com/shamy1st/sql/blob/main/images/character-size.png)

* **CHAR(x)**: fixed length
* **VARCHAR(x)**: variable length, max length 65,535 characters (~64KB) [can be indexed]
  * VARCHAR(50): for short strings, like username, passwords.
  * VARCHAR(255): for medium-length strings, like address
* **MEDIUMTEXT**: max (16MB), good for JSON objects, medium length books
* **LONGTEXT**: max (4GB), good for text books, log files
* **TINTTEXT**: max 255 bytes
* **TEXT**: max 64KB

### Integer Type
https://dev.mysql.com/doc/refman/8.0/en/integer-types.html

* **TINYINT**: 1 Byte [-128, 127]
* **UNSIGNED TINYINT**: [0, 255]
* **SMALLINT**: 2 Bytes [-32K, 32K]
* **MEDIUMINT**: 3 Bytes [-8M, 8M]
* **INT**: 4 Bytes [-2B, 2B]
* **BIGINT**: 8 Bytes [-9Z, 9Z]

### Fixed-point and Floating-point Types

* **DECIMAL(p, s)**: e.g. 1234567.89
* **DEC**: synonym, exactly the same as DECIMAL(p, s)
* **NUMERIC**: synonym, exactly the same as DECIMAL(p, s)
* **FIXED**: synonym, exactly the same as DECIMAL(p, s)
* **FLOAT**: 4 Bytes
* **DOUBLE**: 8 Bytes

### Boolean Type

* **BOOL**: synonym to TINYINT
* **BOOLEAN**: synonym to TINYINT

### Enum and Set Types
Bad feature don't use it!

Enum('small', 'medium', 'large')

SET(...)

### Date and Time Types

* **DATE**:
* **TIME**:
* **DATETIME**: 8 Bytes
* **TIMESTAMP**: 4 Bytes (up to 2038)
* **YEAR**:

### Blob Type
large amount of binary data like images, videos, pdf, ...

* **TINYBLOB**: max 255 Bytes
* **BLOB**: max 65 KB
* **MEDIUMBLOB**: 16 MB
* **LONGBLOB**: 4 GB

### JSON Type
with JSON type you can save, edit properties, remove properties, show specific property in JSON object.


## Designing Databases
https://github.com/shamy1st/database-modeling

## Indexing for High Performance

indexes speed up our queries.

design indexes based on your queries, not your table.

![](https://github.com/shamy1st/sql/blob/main/images/index-1.png)
![](https://github.com/shamy1st/sql/blob/main/images/index-2.png)
![](https://github.com/shamy1st/sql/blob/main/images/index-3.png)
![](https://github.com/shamy1st/sql/blob/main/images/index-cost.png)

## Securing Databases

