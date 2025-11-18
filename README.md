# E-Commerce System Mentorship Tasks

This repository contains solutions to a series of tasks related to designing and querying an e-commerce system database, as part of a mentorship program.

---

## 1. Create the DB schema script with the following entities

**Answer:**

```sql
CREATE DATABASE e_commerce;
USE e_commerce;

CREATE TABLE category (
    Category_Id INT NOT NULL AUTO_INCREMENT,
    Category_Name VARCHAR(255) NOT NULL,
    PRIMARY KEY (Category_Id)
);

CREATE TABLE product (
    Product_Id INT NOT NULL AUTO_INCREMENT,
    Category_Id INT NOT NULL,
    Name VARCHAR(255) NOT NULL,
    Description TEXT,
    Price DECIMAL NOT NULL,
    Stock_Quantity FLOAT NOT NULL,
    PRIMARY KEY (Product_Id),
    FOREIGN KEY (Category_Id) REFERENCES category(Category_Id)
);

CREATE TABLE customer (
    Customer_Id INT NOT NULL AUTO_INCREMENT,
    First_Name VARCHAR(255) NOT NULL,
    Last_Name VARCHAR(255) NOT NULL,
    Email VARCHAR(255) NOT NULL,
    User_Password VARCHAR(256) NOT NULL,
    PRIMARY KEY (Customer_Id)
);

CREATE TABLE customer_order (
    Order_Id INT NOT NULL AUTO_INCREMENT,
    Customer_Id INT NOT NULL,
    Order_Date DATETIME NOT NULL,
    Total_Amount DECIMAL NOT NULL,
    PRIMARY KEY (Order_Id),
    FOREIGN KEY (Customer_Id) REFERENCES customer(Customer_Id)
);

CREATE TABLE order_details (
    Order_Detail_Id INT NOT NULL AUTO_INCREMENT,
    Order_Id INT NOT NULL,
    Product_Id INT NOT NULL,
    Quantity INT NOT NULL,
    Unit_Price DECIMAL NOT NULL,
    PRIMARY KEY (Order_Detail_Id),
    FOREIGN KEY (Order_Id) REFERENCES customer_order(Order_Id),
    FOREIGN KEY (Product_Id) REFERENCES product(Product_Id)
);
```

---

## 2. Identify the relationships between entities

**Answer:**

The relationships between the entities are illustrated in the ERD diagram provided in this repository. Please refer to the `ERD.svg` file for a visual representation of all entity relationships, including primary and foreign key associations.

---

## 3. Draw the ERD diagram of this sample schema

**Answer:**

The Entity-Relationship Diagram (ERD) for the e-commerce system is available in the file [`ERD.svg`](ERD.svg). This diagram visually represents the structure and relationships of all entities in the database schema.

---

## 4. Write an SQL query to generate a daily report of the total revenue for a specific date.

**Answer:**

The following SQL query generates the total revenue for a specific date by summing the `Total_Amount` from the `customer_order` table for orders placed within the specified date range:

```sql
SELECT SUM(co.Total_Amount)
FROM customer_order co
WHERE order_date BETWEEN '2025-11-04 00:00:00' AND '2025-11-04 23:59:59';
```

Replace the date values as needed to generate the report for a different day.

---

## 5. Write an SQL query to generate a monthly report of the top-selling products in a given month.

**Answer:**

The following SQL query generates a monthly report of the top-selling products by quantity for each month. It uses window functions to rank products by total quantity sold per month and selects the top product(s) for each month:

```sql
SELECT * FROM (
    SELECT p.Product_Id,
           p.Name,
           YEAR(Order_Date) AS Order_Year,
           MONTH(Order_Date) AS Order_Month,
           SUM(od.Quantity) AS Total_Quantity,
           RANK() OVER (
               PARTITION BY YEAR(Order_Date), MONTH(Order_Date)
               ORDER BY SUM(od.Quantity) DESC
           ) AS Qty_Rank
      FROM product p
      JOIN order_details od ON p.Product_Id = od.Product_Id
      JOIN customer_order co ON od.Order_Id = co.Order_Id
     GROUP BY p.Product_Id, p.Name, YEAR(Order_Date), MONTH(Order_Date)
) t
WHERE Qty_Rank = 1
ORDER BY Order_Year, Order_Month, Total_Quantity, Qty_Rank;
```

This query returns the top-selling product(s) for each month, including the product name, year, month, and total quantity sold.

---

## 6. Write a SQL query to retrieve a list of customers who have placed orders totaling more than $500 in the past month. Include customer names and their total order amounts. [Complex query]

**Answer:**

The following SQL query retrieves customers who have placed orders totaling more than $500 in the previous month. It includes the customer's full name and their total order amount for that month:

```sql
SELECT Full_Name, TotalAmount FROM (
    SELECT CONCAT(c.First_Name, ' ', c.Last_Name) AS Full_Name,
           SUM(co.Total_Amount) AS TotalAmount,
           YEAR(co.Order_Date) AS Order_Year,
           MONTH(co.Order_Date) AS Order_Month
      FROM Customer c
      JOIN Customer_Order co ON c.Customer_Id = co.Customer_Id
     GROUP BY CONCAT(c.First_Name, ' ', c.Last_Name), YEAR(co.Order_Date), MONTH(co.Order_Date)
) t
WHERE t.Order_Year = YEAR(CURDATE())
  AND t.Order_Month = MONTH(CURDATE() - INTERVAL 1 MONTH)
  AND TotalAmount > 500;
```

This query returns the full names and total order amounts of customers who meet the criteria for the previous month.

---

## 7. How can we apply a denormalization mechanism on customer and order entities?

**Answer:**

Denormalization can be applied to the `customer` and `order` entities to improve query performance and reduce the need for complex joins, at the cost of some data redundancy.

For the `order` entity, denormalization can be achieved by storing customer information directly within the order record. For example, adding columns such as `Customer_First_Name`, `Customer_Last_Name`, and `Customer_Email` to the `order` table allows direct access to customer details without joining the `customer` table. This is especially useful for reporting and historical data, ensuring that customer information at the time of the order is preserved even if the customer's details change later.

On the other hand, the `customer` entity can be denormalized by maintaining summary information about the customer's order history. This can include columns like `Total_Orders` (the total number of orders placed), `Total_Spent` (the cumulative amount spent by the customer), and `Last_Order_Date` (the date of the most recent order). Storing these aggregates directly in the `customer` table enables faster access to frequently needed summary data, reducing the need for real-time calculations across large datasets.

While denormalization can improve performance for certain queries, it also introduces data redundancy and the need for additional logic to keep the data consistent during updates.

---
