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

* MAX()

       SELECT MAX(invoice_total) AS maximum FROM invoices;
       SELECT MAX(payment_date) AS latest_date FROM invoices;

* MIN()

       SELECT MIN(invoice_total) AS minimum FROM invoices;
       SELECT MIN(payment_date) AS oldest_date FROM invoices;

* AVG()

       SELECT AVG(invoice_total) AS average FROM invoices;

* SUM()

       SELECT SUM(invoice_total) AS summation FROM invoices;
       -- multiply each value by 1.1 then apply sum
       SELECT SUM(invoice_total * 1.1) AS summation FROM invoices;
       -- sum all values then multiply by 1.1
       SELECT SUM(invoice_total) * 1.1 AS summation FROM invoices;

* COUNT()

       -- if null records it will not count
       SELECT COUNT(invoice_total) AS counter FROM invoices;
       -- the actual total records of table
       SELECT COUNT(*) AS total_records FROM invoices;
       -- count distinct number of clients
       SELECT COUNT(DISTINCT client_id) AS client_records FROM invoices;





