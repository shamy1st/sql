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

SELECT name, unit_price, unit_price * 1.1 AS new_price
FROM products;

### WHERE









