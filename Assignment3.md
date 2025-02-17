# CS338A3

# Q.1.
```{sql}
DROP TABLE IF EXISTS demo;
DROP TABLE IF EXISTS CustomerCustomerDemo;
DROP TABLE IF EXISTS CustomerDemographics;
```
# Q.2.
```{sql}
CREATE TABLE IF NOT EXISTS ArchivedOrders (
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    ArchivedDate DATETIME DEFAULT CURRENT_TIMESTAMP,
    Freight DECIMAL(10, 2) DEFAULT 0,
    UnitPrice DECIMAL(10, 2) DEFAULT 0 CHECK (UnitPrice >= 0),
    Quantity INT DEFAULT 1 CHECK (Quantity > 0),
    Discount DECIMAL(4, 2) DEFAULT 0 CHECK (Discount BETWEEN 0 AND 1),
    EmployeeID INT,
    CustomerID TEXt,
    OrderDate DATETIME,
    RequiredDate DATETIME,
    ShippedDate DATETIME,
    ShipName TEXT,
    ShipAddress TEXT,
    ShipCity TEXT,
    ShipRegion TEXT,
    ShipPostalCode TEXT,
    ShipCountry TEXT,
    ShipVia INT,
    PRIMARY KEY (OrderID, ProductID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID),
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
    FOREIGN KEY (ShipVia) REFERENCES Shippers(ShipperID)
);

INSERT INTO ArchivedOrders (
    OrderID, ProductID, ArchivedDate, Freight, UnitPrice, Quantity, Discount,
    CustomerID, EmployeeID, OrderDate, RequiredDate, ShippedDate, ShipVia,
    ShipName, ShipAddress, ShipCity, ShipRegion, ShipPostalCode, ShipCountry
)
SELECT
    o.OrderID,
    od.ProductID,
    CURRENT_TIMESTAMP,
    o.Freight,
    od.UnitPrice,
    od.Quantity,
    od.Discount,
    o.CustomerID,
    o.EmployeeID,
    o.OrderDate,
    o.RequiredDate,
    o.ShippedDate,
    o.ShipVia,
    o.ShipName,
    o.ShipAddress,
    o.ShipCity,
    o.ShipRegion,
    o.ShipPostalCode,
    o.ShipCountry
FROM Orders o
JOIN OrderDetails od ON o.OrderID = od.OrderID;

```

# Q.3.
```{sql}

INSERT INTO ArchivedOrders (
    OrderID, ProductID, ArchivedDate, Freight, UnitPrice, Quantity, Discount,
    CustomerID, EmployeeID, OrderDate, RequiredDate, ShippedDate, ShipVia,
    ShipName, ShipAddress, ShipCity, ShipRegion, ShipPostalCode, ShipCountry
)
SELECT
    o.OrderID,
    od.ProductID,
    CURRENT_TIMESTAMP,
    o.Freight,
    od.UnitPrice,
    od.Quantity,
    od.Discount,
    o.CustomerID,
    o.EmployeeID,
    o.OrderDate,
    o.RequiredDate,
    o.ShippedDate,
    o.ShipVia,
    o.ShipName,
    o.ShipAddress,
    o.ShipCity,
    o.ShipRegion,
    o.ShipPostalCode,
    o.ShipCountry
FROM Orders o
JOIN OrderDetails od ON o.OrderID = od.OrderID
WHERE o.OrderDate < '2016-08-01';


DELETE FROM OrderDetails
WHERE OrderID IN (
    SELECT OrderID
    FROM Orders
    WHERE OrderDate < '2016-08-01'
);

DELETE FROM Orders
WHERE OrderDate < '2016-08-01';


SELECT COUNT(*) AS NumberOfOrders FROM Orders;

SELECT COUNT(*) AS NumberOfOrderDetails FROM OrderDetails;

SELECT COUNT(*) AS NumberOfArchivedOrders FROM ArchivedOrders;

```
# Q.4.
```{sql}
SELECT COUNT(*)
FROM Products p
JOIN Suppliers s ON p.SupplierID = s.SupplierID
WHERE s.CompanyName = 'Ma Maison';
```

# Q.5.
```{sql}
SELECT AVG(ProductCount) AS AverageProductsPerCategory
FROM (
    SELECT CategoryID, COUNT(*) AS ProductCount
    FROM Products
    GROUP BY CategoryID
) AS CategoryProductCounts;
```

# Q.6.
```{sql}
CREATE VIEW RegionEmployeeCounts AS
SELECT r.RegionID, r.RegionDescription, COUNT(e.EmployeeID) AS NumberOfEmployees
FROM Regions r
JOIN Territories t ON r.RegionID = t.RegionID
JOIN EmployeeTerritories et ON t.TerritoryID = et.TerritoryID
JOIN Employees e ON et.EmployeeID = e.EmployeeID
GROUP BY r.RegionID, r.RegionDescription;

WITH RankedRegions AS (
    SELECT RegionID, RegionDescription, NumberOfEmployees,
           ROW_NUMBER() OVER (ORDER BY NumberOfEmployees DESC) AS Rank
    FROM RegionEmployeeCounts
)

SELECT RegionID, RegionDescription, NumberOfEmployees
FROM RankedRegions
WHERE Rank = 1;
```

# Q.7.
```{sql}
SELECT COUNT(*)
FROM (
    SELECT CustomerID
    FROM Orders
    GROUP BY CustomerID
    HAVING COUNT(OrderID) > 10
) AS CustomersWithMoreThan10Orders;

```

# Q.8.
```{sql}
SELECT c.CompanyName, c.City, c.Country
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN Shippers s ON o.ShipVia = s.ShipperID
WHERE s.CompanyName = 'United Package';
```

# Q.9.
```{sql}
SELECT e.EmployeeID, e.ReportsTo AS ManagerID, COUNT(et.TerritoryID) AS NumberOfTerritories
FROM Employees e
JOIN Employees m ON e.ReportsTo = m.EmployeeID
JOIN EmployeeTerritories et ON m.EmployeeID = et.EmployeeID
GROUP BY e.EmployeeID, e.ReportsTo;

```


# Q.10.
```{sql}
INSERT INTO Shippers (ShipperID, CompanyName, Phone)
VALUES (4, 'Waterloo Shipping', '11111111');

SELECT s.CompanyName, 
       COUNT(o.OrderID) AS NumberOfOrders, 
       AVG(OrderTotal.TotalCost) AS AverageTotalCost
FROM Shippers s
JOIN Orders o ON s.ShipperID = o.ShipVia
JOIN (
    SELECT OrderID, SUM(UnitPrice * Quantity - Discount) AS TotalCost
    FROM OrderDetails
    GROUP BY OrderID
    HAVING COUNT(ProductID) > 5
) AS OrderTotal ON o.OrderID = OrderTotal.OrderID
GROUP BY s.CompanyName;

```


