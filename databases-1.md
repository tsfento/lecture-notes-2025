# Database Fundamentals - Part 1

## Introduction

-   We're going to explore databases and how we can use them to store and retrieve data in our applications

## Key Concepts

### Can anyone define what a database is?

-   A centralized system for storing, organizing, and managing data efficiently

### What are some use cases for databases?

-   Used in various applications for data storage and retrieval, essential in multiuser environments and for individual applications.

### Database Management Systems (DBMS)

-   Definition: Software that facilitates creating, managing, and manipulating data.
-   Functionality: Allows for data addition, deletion, updating, and querying.
-   Flexibility: Simplifies adapting to changing information requirements.
-   Examples: SQLite, MySQL, PostgreSQL, and Oracle.

### Types of Databases

-   Relational (SQL) Databases: Store data in tables, ideal for structured data. SQL used for manipulation.
-   Non-relational (NoSQL) Databases: Use various models like document, key-value for unstructured data. Flexible in handling changes and large-scale data.
-   Others include graph databases, object-oriented databases, and XML databases.

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Ask the assistant: What is the difference between a relational and a non-relational database?
> -   Use the assistant's answer to clarify the distinction for students.

### Choosing Database Type

-   Relational Databases: Suitable for structured data and multi-row transactions. (e.g., Banking systems)
-   Non-relational Databases: Best for large-scale, unstructured data with evolving schemas. (e.g., Social media platforms)
-   Hybrid Approach: Combining both types for different needs within an application.

# Introduction to SQL

### What is SQL?

-   SQL (Structured Query Language): Standard language for interacting with relational databases.
-   Basic Operations: Data retrieval, insertion, updating, and deletion.

### Creating Tables in SQL

-   About Tables: Store data in rows and columns, each row represents a record, and each column represents a field.
-   CREATE TABLE: Defines a new table structure with specified columns and data types.
-   Data Types: Include int, varchar, date, text, each serving different data storage needs.
-   Performance Considerations: Choosing appropriate data types and sizes impacts database performance.

Create a table named Customers with columns for customer ID, name, and email.

```sql
CREATE TABLE Customers (
  CustomerID int PRIMARY KEY,
  Name varchar(255),
  Email varchar(255)
);
```

This SQL statement creates a new table called Customers with three columns. The CustomerID column is an integer and is the primary key, while Name and Email are variable-length strings.

### Inserting Data

-   INSERT INTO: Adds new rows to a table.
-   Data Integrity: Ensures the correct and consistent entry of data.

Add three records to the Customers table.

```sql
INSERT INTO Customers (CustomerID, Name, Email)
VALUES (1, 'John Doe', 'john.doe@email.com'),
       (2, 'Jane Smith', 'jane.smith@email.com'),
       (3, 'Luke James', 'luke.james@email.com');
```

This query inserts three new rows into the Customers table. Each row includes a customer ID, a name, and an email address.

### Selecting Data

-   SELECT: Retrieves data from the database.
-   Filtering: Using conditions to refine data retrieval.

Retrieve all records from the Customers table.

```sql
SELECT * FROM Customers;
```

The SELECT \* statement is used to retrieve all columns for every row in the Customers table.

### Where Clause

-   Purpose: Filters records based on specified conditions.
-   Usage: Commonly combined with logical operators like AND, OR.

Find the customer with the name 'John Doe'.

```sql
SELECT * FROM Customers WHERE Name = 'John Doe';
```

This query retrieves all columns from the Customers table where the Name column matches 'John Doe'.

### Order By and Limit Clauses

-   Order By: Sorts the result set based on one or more columns.
-   Limit: Restricts the number of rows returned by a query.

Get the first 2 customers sorted by name.

```sql
SELECT * FROM Customers ORDER BY Name ASC LIMIT 2;
```

This command sorts the results by the Name column in ascending order and limits the output to the first 2 rows.

### Aggregate Functions and GROUP BY Clause

-   Aggregate Functions: Perform calculations on sets of data and return single values (e.g., COUNT, SUM).
-   GROUP BY: Groups rows sharing a particular property, often used with aggregate functions.

### Introduction to Joins

-   Joins: Combine rows from two or more tables based on related columns.
-   Types of Joins: Includes INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN, and SELF JOIN.
-   Practical Use: Essential for querying related data across multiple tables.

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Ask the assistant: Show an example SQL query that joins Customers and Orders to list all orders for each customer.
> -   Use the assistant's answer to walk through join logic live.

## E-commerce Database

An e-commerce database is a database that stores information about products, customers, and orders. It is used to power e-commerce websites and applications.

Let's create the following tables:

```sql
CREATE TABLE Customers (
    CustomerID int,
    Name varchar(255),
    Email varchar(255)
);
```

-   Customers
    -   CustomerID (int)
    -   Name (varchar)
    -   Email (varchar)

```sql
CREATE TABLE Orders (
    OrderID int,
    CustomerID int,
    OrderDate date
);
```

-   Orders
    -   OrderID (int)
    -   CustomerID (int)
    -   OrderDate (date)

The orders table has a foreign key column called CustomerID that references the CustomerID column in the Customers table. This is how we can link the two tables together.

```sql
CREATE TABLE Products (
    ProductID int,
    ProductName varchar(255)
);
```

-   Products
    -   ProductId (int)
    -   ProductName (varchar)

```sql
CREATE TABLE OrderDetails (
    OrderDetailID int,
    OrderID int,
    ProductID int,
    Quantity int,
    PricePerItem float
);
```

-   OrderDetails
    -   OrderDetailID (int)
    -   OrderID (int)
    -   ProductID (int)
    -   Quantity (int)
    -   PricePerItem (float)

The OrderDetails table has a foreign key column called OrderID that references the OrderID column in the Orders table. It also has a foreign key column called ProductID that references the ProductID column in the Products table. This allows us to link the OrderDetails table to the Orders and Products tables.

Now that we have created the tables, let's insert some data into them.

Create two customers:

```sql
INSERT INTO Customers (CustomerID, Name, Email) VALUES (1, 'Alice Smith', 'alice@example.com');
INSERT INTO Customers (CustomerID, Name, Email) VALUES (2, 'Bob Johnson', 'bob@example.com');
```

Create two orders for the customers:

```sql
INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (1, 1, '2023-01-15');
INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (2, 2, '2023-01-17');
```

Create two products for the orders:

```sql
INSERT INTO Products (ProductID, ProductName) VALUES (1, 'Laptop');
INSERT INTO Products (ProductID, ProductName) VALUES (2, 'Smartphone');
```

Create four order details for the orders:

```sql
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (1, 1, 1, 1, 1000.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (2, 1, 2, 2, 500.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (3, 2, 1, 1, 1200.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (4, 2, 2, 1, 800.00);
```

Now that we have some data in our tables, let's write some queries to retrieve it.

Let's select the customers name, order date, product name, quantity and price per item:

```sql
SELECT
    Customers.Name AS CustomerName,
    Orders.OrderDate,
    Products.ProductName,
    OrderDetails.Quantity,
    OrderDetails.PricePerItem
```

from customers, orders, order details and products tables:

```sql
FROM Customers
JOIN Orders ON Customers.CustomerID = Orders.CustomerID
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
JOIN Products ON OrderDetails.ProductID = Products.ProductID
```

Where the customer id is 1:

```sql
WHERE
    Customers.CustomerID = 1;
```

Outcome:

```bash
Alice Smith|2023-01-15|Laptop|1|1000.0
Alice Smith|2023-01-15|Smartphone|2|500.0
```

### Result:

```sql
CREATE TABLE Customers (
    CustomerID int,
    Name varchar(255),
    Email varchar(255)
);

CREATE TABLE Orders (
    OrderID int,
    CustomerID int,
    OrderDate date
);

CREATE TABLE OrderDetails (
    OrderDetailID int,
    OrderID int,
    ProductID int,
    Quantity int,
    PricePerItem float
);

CREATE TABLE Products (
    ProductID int,
    ProductName varchar(255)
);

INSERT INTO Customers (CustomerID, Name, Email) VALUES (1, 'Alice Smith', 'alice@example.com');
INSERT INTO Customers (CustomerID, Name, Email) VALUES (2, 'Bob Johnson', 'bob@example.com');

INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (1, 1, '2023-01-15');
INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (2, 2, '2023-01-17');

INSERT INTO Products (ProductID, ProductName) VALUES (1, 'Laptop');
INSERT INTO Products (ProductID, ProductName) VALUES (2, 'Smartphone');

INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (1, 1, 1, 1, 1000.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (2, 1, 2, 2, 500.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (3, 2, 1, 1, 1200.00);
INSERT INTO OrderDetails (OrderDetailID, OrderID, ProductID, Quantity, PricePerItem) VALUES (4, 2, 2, 1, 800.00);

SELECT
    Customers.Name AS CustomerName,
    Orders.OrderDate,
    Products.ProductName,
    OrderDetails.Quantity,
    OrderDetails.PricePerItem
FROM
    Customers
JOIN Orders ON Customers.CustomerID = Orders.CustomerID
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
JOIN Products ON OrderDetails.ProductID = Products.ProductID
WHERE
    Customers.CustomerID = 1;
```

# Any questions?
