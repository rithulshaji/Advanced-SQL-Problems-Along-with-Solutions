# Advanced-SQL-Problems-Along-with-Solutions
This repository contains a collection of advanced SQL problems along with their detailed solutions. Each query is designed to solve real-world business scenarios and includes various SQL techniques such as recursive queries, window functions, complex joins, CTEs (Common Table Expressions), and pivot tables.

## 1. Find Top-N Employees by Department
**Question:** Write a query to find the top 3 highest-paid employees from each department.

**Tables:**
- Employees (EmployeeID, Name, Salary, DepartmentID)
- Departments (DepartmentID, DepartmentName)

```sql
WITH RankedSalary AS (
    SELECT d.DepartmentName, e.Name, e.Salary,
           DENSE_RANK() OVER (PARTITION BY d.DepartmentName ORDER BY e.Salary DESC) AS Drnk
    FROM Employees e
    INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
)
SELECT DepartmentName, Name, Salary
FROM RankedSalary
WHERE Drnk <= 3;
```

## 2. Recursive Query for Organizational Hierarchy
**Question:** Given an organizational hierarchy, where each employee reports to another employee, write a recursive query to find all direct and indirect reports of a given employee.

**Tables:**
- Employees (EmployeeID, Name, ManagerID)

```sql
WITH RECURSIVE OrgHierarchy AS (
    SELECT EmployeeID, Name, ManagerID
    FROM Employees
    WHERE ManagerID = <EmployeeID>  -- Replace <EmployeeID> with the specific ID
    UNION ALL
    SELECT e.EmployeeID, e.Name, e.ManagerID
    FROM Employees e
    INNER JOIN OrgHierarchy oh ON e.ManagerID = oh.EmployeeID
)
SELECT * FROM OrgHierarchy;
```

## 3. Calculate Year-over-Year Sales Growth
**Question:** Write a query to calculate the year-over-year percentage growth of total sales for each year.

**Tables:**
- Sales (SaleID, SaleAmount, SaleDate)

```sql
WITH YearWiseSales AS (
    SELECT YEAR(SaleDate) AS Year, SUM(SaleAmount) AS TotalSales
    FROM Sales
    GROUP BY YEAR(SaleDate)
)
SELECT Year,
       ROUND((TotalSales - LAG(TotalSales, 1, TotalSales) OVER (ORDER BY Year)) / LAG(TotalSales, 1, TotalSales) OVER (ORDER BY Year) * 100, 2) AS YoYGrowth
FROM YearWiseSales;
```

## 4. Find Customers with Orders in Every Month
**Question:** Write a query to find customers who have placed at least one order in every month of 2023.

**Tables:**
- Orders (OrderID, CustomerID, OrderDate)
- Customers (CustomerID, CustomerName)

```sql
SELECT c.CustomerName
FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.CustomerID
WHERE YEAR(o.OrderDate) = 2023
GROUP BY c.CustomerName
HAVING COUNT(DISTINCT MONTH(o.OrderDate)) = 12;
```

## 5. Filter Duplicates Based on Criteria
**Question:** In a dataset of products, find the most recent record for each product where duplicate entries exist.

**Tables:**
- Products (ProductID, ProductName, UpdatedAt, Price)

```sql
SELECT ProductID, ProductName, MAX(UpdatedAt) AS UpdatedAt, MAX(Price) AS Price
FROM Products
WHERE ProductID IN (
    SELECT ProductID FROM Products 
    GROUP BY ProductID
    HAVING COUNT(*) > 1
)
GROUP BY ProductID, ProductName;

SELECT ProductID, ProductName, MAX(UpdatedAt) AS UpdatedAt, Price
FROM Products p
WHERE UpdatedAt = (SELECT MAX(UpdatedAt) FROM Products WHERE ProductID = p.ProductID)
GROUP BY ProductID, ProductName, Price;
```

## 6. Find Top-Performing Products per Region
**Question:** Write a query to find the top 5 best-selling products in each region based on total sales.

**Tables:**
- Sales (SaleID, ProductID, Region, SaleAmount)
- Products (ProductID, ProductName)

```sql
WITH ProductSales AS (
    SELECT s.Region, p.ProductName, SUM(s.SaleAmount) AS TotalSales
    FROM Sales s
    INNER JOIN Products p ON s.ProductID = p.ProductID
    GROUP BY s.Region, p.ProductName
), RankedSales AS (
    SELECT Region, ProductName, TotalSales,
           DENSE_RANK() OVER (PARTITION BY Region ORDER BY TotalSales DESC) AS Drnk
    FROM ProductSales
)
SELECT Region, ProductName, TotalSales
FROM RankedSales
WHERE Drnk <= 5;
```

## 7. Detect Missing Dates in a Series
**Question:**  Write a query to find all dates where no sales were made, given a range of dates (between '2023-01-01' and '2023-12-31').

**Tables:**
- Sales (SaleID, SaleDate, SaleAmount)


```sql
WITH RECURSIVE DateRange AS (
    SELECT '2023-01-01' AS SaleDate
    UNION ALL
    SELECT DATE_ADD(SaleDate, INTERVAL 1 DAY)
    FROM DateRange
    WHERE SaleDate < '2023-12-31'
)
SELECT SaleDate
FROM DateRange
WHERE SaleDate NOT IN (SELECT SaleDate FROM Sales);
```

## 8. Complex Join with Multiple Conditions
**Question:**  Write a query to find all orders where the customer ordered both 'Product A' and 'Product B' in the same transaction.

**Tables:**
- Orders (OrderID, CustomerID, OrderDate)
- OrderDetails (OrderID, ProductID, Quantity)
- Products (ProductID, ProductName)


```sql
SELECT o.OrderID, CustomerID, OrderDate
FROM Orders o
INNER JOIN OrderDetails od ON o.OrderID = od.OrderID
INNER JOIN Products p ON p.ProductID = od.ProductID
WHERE p.ProductName IN ('Product A', 'Product B')
GROUP BY o.OrderID, CustomerID, OrderDate
HAVING COUNT(DISTINCT p.ProductName) = 2;
```

## 9. Find Employees with No Subordinates
**Question:**  Write a query to find all employees who do not have any subordinates reporting to them.

**Tables:**
- Employees (EmployeeID, Name, ManagerID)


```sql
SELECT e.EmployeeID, e.Name
FROM Employees e
LEFT JOIN Employees m ON e.EmployeeID = m.ManagerID
WHERE m.ManagerID IS NULL;
```

## 10. Dynamic Pivot Table
**Question:**  Write a query to pivot the sales data such that each row represents a product and each column represents the total sales amount for each region.

**Tables:**
- Sales (SaleID, ProductID, Region, SaleAmount)


```sql
SELECT ProductID,
       SUM(CASE WHEN Region = 'Region_A' THEN SaleAmount ELSE 0 END) AS Region_A_Sales,
       SUM(CASE WHEN Region = 'Region_B' THEN SaleAmount ELSE 0 END) AS Region_B_Sales,
       SUM(CASE WHEN Region = 'Region_C' THEN SaleAmount ELSE 0 END) AS Region_C_Sales
FROM Sales
GROUP BY ProductID;
```

## By Rithul Shaji
