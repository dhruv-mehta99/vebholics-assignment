# vebholics-assignment

## Database Queries

#### 1.Which shipper do we use the most to ship our orders out through?
##### SQL Query
```
SELECT s.companyName
FROM orders o
JOIN shippers s ON o.shipVia = s.shipperID
GROUP BY s.companyName
ORDER BY COUNT(*) DESC
LIMIT 1;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/5b793fc4-5646-401e-a9a0-e324bc41d58d)

##### MongoDB Query
```
db.orders.aggregate([
  {
    $group:
      {
        _id: "$shipVia",
        count: {
          $count: {},
        },
      },
  },
  {
    $sort:
      {
        count: -1,
      },
  },
  {
    $limit:
      1,
  },
  {
    $lookup:
      {
        from: "shippers",
        localField: "_id",
        foreignField: "shipperID",
        as: "shipper",
      },
  },
  {
    $unwind:
      {
        path: "$shipper",
      },
  },
  {
    $project: {
      _id: 0,
      shipperCompanyName: "$shipper.companyName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/b155d21a-a04c-4cb0-9629-2805c4364d17)


#### 2.List the following employee information (EmployeeID, LastName, FirstName, ManagerLastName, ManagerFirstName)
##### SQL Query
```
SELECT e.employeeID, e.lastName, e.firstName, m.lastName AS managerLastName, m.firstName AS managerFirstName
FROM employees e
LEFT JOIN employees m ON e.reportsTo = m.employeeID;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/0c81dca6-53aa-4ee6-a7f0-7a88a5f44d5c)

##### MongoDB Query
```
db.employees.aggregate([
  {
    $lookup: {
      from: "employees",
      localField: "reportsTo",
      foreignField: "employeeID",
      as: "manager",
    },
  },
  {
    $unwind: {
      path: "$manager",
      preserveNullAndEmptyArrays: true,
    },
  },
  {
    $project: {
      _id: 0,
      employeeID: 1,
      lastName: 1,
      firstName: 1,
      managerLastName: "$manager.lastName",
      managerFirstName: "$manager.firstName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/d240ce14-81bb-4cfd-9e56-d13d7cfe1020)


#### 3.What are the last names of all employees who were born in the month of November?
##### SQL Query
```
SELECT lastName
FROM employees
WHERE MONTH(birthDate) = 11;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/1456374e-06dc-43ef-a5d1-ab87435be82b)

last name is null as there is no employee with birthdate in november.

##### MongoDB Query
```
db.employees.aggregate([
  {
    $addFields: {
      birthDate: {
        $toDate: "$birthDate",
      },
    },
  },
  {
    $project: {
      _id: 0,
      lastName: 1,
      month: {
        $month: "$birthDate",
      },
    },
  },
  {
    $match: {
      month: 11,
    },
  },
  {
    $project: {
      _id: 0,
      lastName: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/a7c98b34-c6c0-4c63-90ff-98f1a71b8a20)

#### 4.List each employee (lastname, firstname, territory) and sort the list by territory and then by employee's last name. Remember employees may work for more than one territory
##### SQL Query
```
SELECT e.lastName, e.firstName, et.territoryID
FROM employees e
INNER JOIN employee_territories et ON e.employeeID = et.employeeID
ORDER BY et.territoryID, e.lastName;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/6d6a84e4-406f-4015-a83b-244f077b23c0)

##### MongoDB Query
```
db.employees.aggregate([
  {
    $lookup: {
      from: "employee_territories",
      localField: "employeeID",
      foreignField: "employeeID",
      as: "employee_territories",
    },
  },
  {
    $project: {
      _id: 0,
      lastName: 1,
      firstName: 1,
      employee_territories: 1,
    },
  },
  {
    $lookup: {
      from: "territories",
      localField:
        "employee_territories.territoryID",
      foreignField: "territoryID",
      as: "territories",
    },
  },
  {
    $unset: "employee_territories",
  },
  {
    $sort: {
      "territories.territoryDescription": 1,
      lastName: 1,
    },
  },
  {
    $addFields: {
      territory:
        "$territories.territoryDescription",
    },
  },
  {
    $unset: "territories",
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/3f85e84e-6cad-4194-bfef-ace80d3bd10c)


#### 5.Regarding sales value, what has been our best-selling product of all time?
##### SQL Query
```
SELECT p.productName
FROM order_details od 
INNER JOIN products p ON p.productID=od.productID 
GROUP BY od.productID ORDER BY SUM(od.unitPrice * od.quantity*(1-od.discount)) DESC
LIMIT 1;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/523a5ee3-58e1-4ac8-a822-3c71cdf1bef5)

##### MongoDB Query
```
db.order_details.aggregate([
  {
    $group: {
      _id: "$productID",
      totalSales: {
        $sum: {
          $multiply: [
            "$unitPrice",
            "$quantity",
            {
              $subtract: [1, "$discount"],
            },
          ],
        },
      },
    },
  },
  {
    $sort: {
      totalSales: -1,
    },
  },
  {
    $limit: 1,
  },
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "productID",
      as: "product",
    },
  },
  {
    $unwind: "$product",
  },
  {
    $project: {
      _id: 0,
      productName: "$product.productName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/012bb983-2a6d-45fd-ae77-07435e6b8c5e)


#### 6.Regarding sales value, only include products that have at least been sold once, which has been our worst-selling product of all time
##### SQL Query
```
SELECT p.productName
FROM order_details od 
INNER JOIN products p ON p.productID=od.productID 
GROUP BY od.productID ORDER BY SUM(od.unitPrice * od.quantity*(1-od.discount)) ASC
LIMIT 1;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/7e6eac88-9e0c-437c-b446-359fb3191d78)

##### MongoDB Query
```
db.order_details.aggregate([
  {
    $group: {
      _id: "$productID",
      totalSales: {
        $sum: {
          $multiply: [
            "$unitPrice",
            "$quantity",
            {
              $subtract: [1, "$discount"],
            },
          ],
        },
      },
    },
  },
  {
    $sort: {
      totalSales: 1,
    },
  },
  {
    $limit: 1,
  },
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "productID",
      as: "product",
    },
  },
  {
    $unwind: "$product",
  },
  {
    $project: {
      _id: 0,
      productName: "$product.productName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/7e9bd3c2-fb21-40f1-b6df-b36c109156a2)



#### 7.Regarding sales value, which month has been traditionally best for sales
##### SQL Query
```
SELECT DATE_FORMAT(o.orderDate, '%m') AS month
FROM orders AS o
INNER JOIN order_details od ON od.orderId=o.orderID
GROUP BY month ORDER BY SUM(od.unitPrice * od.quantity*(1-od.discount)) DESC
LIMIT 1;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/1090d4ee-e363-4e98-b65e-be6f10ded6f5)

##### MongoDB Query
```
db.order_details.aggregate([
  {
    $group: {
      _id: "$orderID",
      totalSales: {
        $sum: {
          $multiply: [
            "$unitPrice",
            "$quantity",
            {
              $subtract: [1, "$discount"],
            },
          ],
        },
      },
    },
  },
  {
    $sort: {
      totalSales: -1,
    },
  },
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "orderID",
      as: "order",
    },
  },
  {
    $unwind: {
      path: "$order",
    },
  },
  {
    $addFields:
      /**
       * newField: The new field name.
       * expression: The new field expression.
       */
      {
        orderDate: {
          $toDate: "$order.orderDate",
        },
        month: {
          $month: {
            date: {
              $toDate: "$order.orderDate",
            },
          },
        },
      },
  },
  {
    $group: {
      _id: "$month",
      totalSalesMonthwise: {
        $sum: "$totalSales",
      },
    },
  },
  {
    $sort: {
      totalSalesMonthwise: -1,
    },
  },
  {
    $limit: 1,
  },
  {
    $project: {
      _id: 0,
      month: "$_id",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/09345420-53b2-40de-a77d-81fd30cec829)


#### 8.What is the name of our best sales person
##### SQL Query
```
SELECT CONCAT(emp.firstName, ' ', emp.lastName) as employeeName
FROM orders AS o
INNER JOIN employees emp ON emp.employeeID=o.employeeID
INNER JOIN order_details od ON od.orderId=o.orderID
GROUP BY o.employeeID 
ORDER BY SUM(od.unitPrice * od.quantity*(1-od.discount)) DESC
LIMIT 1;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/dfd67847-dc51-4524-b381-44195a537e3e)

##### MongoDB Query
```
db.orders.aggregate([
  {
    $lookup: {
      from: "employees",
      localField: "employeeID",
      foreignField: "employeeID",
      as: "employee",
    },
  },
  {
    $unwind: "$employee",
  },
  {
    $lookup: {
      from: "order_details",
      localField: "orderID",
      foreignField: "orderID",
      as: "orderDetails",
    },
  },
  {
    $unwind: "$orderDetails",
  },
  {
    $group: {
      _id: "$employee.employeeID",
      employeeName: {
        $first: {
          $concat: [
            "$employee.firstName",
            " ",
            "$employee.lastName",
          ],
        },
      },
      totalSales: {
        $sum: {
          $multiply: [
            "$orderDetails.unitPrice",
            "$orderDetails.quantity",
            {
              $subtract: [
                1,
                "$orderDetails.discount",
              ],
            },
          ],
        },
      },
    },
  },
  {
    $sort: {
      totalSales: -1,
    },
  },
  {
    $limit: 1,
  },
  {
    $project: {
      _id: 0,
      employeeName: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/ded4f59b-a751-4202-bcca-0d07c7208979)



#### 9.Product report (productID, ProductName, Supplier Name, Product Category). Order the list by product category
##### SQL Query
```
SELECT p.productID, p.productName ,s.companyName AS supplierName, c.categoryName AS productCategory
FROM products p
INNER JOIN suppliers s ON p.supplierID=s.supplierID
INNER JOIN categories c ON p.categoryID=c.categoryID
ORDER BY c.categoryName;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/6f9066a7-1376-4df3-8d8a-c67360e694d0)

##### MongoDB Query
```
db.products.aggregate([
  {
    $lookup: {
      from: "suppliers",
      localField: "supplierID",
      foreignField: "supplierID",
      as: "supplier",
    },
  },
  {
    $unwind: "$supplier",
  },
  {
    $lookup: {
      from: "categories",
      localField: "categoryID",
      foreignField: "categoryID",
      as: "category",
    },
  },
  {
    $unwind: "$category",
  },
  {
    $sort: {
      "category.categoryName": 1,
    },
  },
  {
    $project: {
      _id: 0,
      productID: "$productID",
      productName: "$productName",
      supplierName: "$supplier.companyName",
      productCategory: "$category.categoryName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/81f4e314-ace6-450d-83f1-7ffcbf5d488b)



#### 10.Produce a count of the employees by each sales region
##### SQL Query
```
SELECT region, COUNT(*) AS employeeCount
FROM employees
WHERE region IS NOT NULL
GROUP BY region;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/41523a99-2446-468b-89cb-087895bc2f9a)

##### MongoDB Query
```
db.employees.aggregate([
  {
    $match: {
      region: {
        $ne: null,
      },
    },
  },
  {
    $group: {
      _id: "$region",
      employeeCount: {
        $sum: 1,
      },
    },
  },
  {
    $project: {
      _id: 0,
      region: "$_id",
      employeeCount: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/9e9736a7-04a2-4c34-8e42-b0e9c765aa05)




#### 11. List the dollar values for sales by region
##### SQL Query
```
SELECT o.shipRegion as salesRegion, SUM(od.unitPrice * od.quantity * (1 - od.discount)) AS totalSales
FROM orders o
JOIN order_details od ON o.orderID = od.orderID
WHERE o.shipRegion IS NOT NULL
GROUP BY o.shipRegion;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/9b58f118-111b-4077-831a-eca10775f29f)

##### MongoDB Query
```
db.employees.aggregate([
  {
    $match: {
      shipRegion: {
        $ne: null,
      },
    },
  },
  {
    $lookup: {
      from: "order_details",
      localField: "orderID",
      foreignField: "orderID",
      as: "orderDetails",
    },
  },
  {
    $unwind: "$orderDetails",
  },
  {
    $group: {
      _id: "$shipRegion",
      totalSales: {
        $sum: {
          $multiply: [
            "$orderDetails.unitPrice",
            "$orderDetails.quantity",
            {
              $subtract: [
                1,
                "$orderDetails.discount",
              ],
            },
          ],
        },
      },
    },
  },
  {
    $project: {
      _id: 0,
      salesRegion: "$_id",
      totalSales: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/ba818944-7c29-4a41-86f8-779265104f3f)



#### 12. What is the average value of a sales order?
##### SQL Query
```
SELECT AVG(orderValue) AS averageOrderValue
FROM (
    SELECT orderID, SUM(unitPrice * quantity * (1 - discount)) AS orderValue
    FROM order_details
    GROUP BY orderID
) AS subquery;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/940d260a-a7af-4bb7-af24-a27a9f932118)

##### MongoDB Query
```
db.order_details.aggregate([
  {
    $group: {
      _id: "$orderID",
      orderValue: {
        $sum: {
          $multiply: [
            "$unitPrice",
            "$quantity",
            {
              $subtract: [1, "$discount"],
            },
          ],
        },
      },
    },
  },
  {
    $group: {
      _id: null,
      averageOrderValue: {
        $avg: "$orderValue",
      },
    },
  },
  {
    $project: {
      _id: 0,
      averageOrderValue: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/ff01752a-7179-4d6a-b7fb-6ca1e7cef717)



#### 13. List orders (OrderID, OrderDate, Customer Name) where the total order value exceeds a sales order's average value.
##### SQL Query
```
SELECT o.OrderID, o.OrderDate, c.CompanyName as CustomerName
FROM orders o
JOIN (
    SELECT od.OrderID, SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalOrderValue
    FROM order_details od
    GROUP BY od.OrderID
) AS order_summary ON o.OrderID = order_summary.OrderID
INNER JOIN customers c ON o.CustomerID = c.CustomerID
JOIN (
    SELECT AVG(orderValue) AS averageOrderValue
FROM (
    SELECT orderID, SUM(unitPrice * quantity * (1 - discount)) AS orderValue
    FROM order_details
    GROUP BY orderID
) AS subquery
) AS average_summary
WHERE order_summary.TotalOrderValue > average_summary.AverageOrderValue;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/2549cf9e-5e2f-4b28-b4cc-b96c5d047496)

##### MongoDB Query
```
db.orders.aggregate([
  {
    $lookup: {
      from: "order_details",
      localField: "orderID",
      foreignField: "orderID",
      as: "order_details",
    },
  },
  {
    $unwind: "$order_details",
  },
  {
    $project: {
      _id: 0,
      orderID: 1,
      orderDate: 1,
      customerID: 1,
      order_details: 1,
    },
  },
  {
    $group: {
      _id: "$orderID",
      orderID: {
        $first: "$orderID",
      },
      orderDate: {
        $first: "$orderDate",
      },
      customerID: {
        $first: "$customerID",
      },
      orderValue: {
        $sum: {
          $multiply: [
            "$order_details.unitPrice",
            "$order_details.quantity",
            {
              $subtract: [
                1,
                "$order_details.discount",
              ],
            },
          ],
        },
      },
    },
  },
  {
    $lookup: {
      from: "customers",
      localField: "customerID",
      foreignField: "customerID",
      as: "customer",
    },
  },
  {
    $unwind: "$customer",
  },
  {
    $lookup: {
      from: "order_details",
      pipeline: [
        {
          $group: {
            _id: "$orderID",
            orderValue: {
              $sum: {
                $multiply: [
                  "$unitPrice",
                  "$quantity",
                  {
                    $subtract: [1, "$discount"],
                  },
                ],
              },
            },
          },
        },
        {
          $group: {
            _id: null,
            averageValue: {
              $avg: "$orderValue",
            },
          },
        },
      ],
      as: "average_summary",
    },
  },
  {
    $unwind: {
      path: "$average_summary",
      preserveNullAndEmptyArrays: true,
    },
  },
  {
    $match: {
      $expr: {
        $lt: [
          "$average_summary.averageValue",
          "$orderValue",
        ],
      },
    },
  },
  {
    $project: {
      _id: 0,
      orderID: 1,
      orderDate: 1,
      customerName: "$customer.companyName",
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/514c3de0-99cb-45c3-92d2-3b56c7d79390)



#### 14. Produce a customer report (must also include those we have not yet done business with) showing CustomerID, Customer name, and total sales made to that customer
##### SQL Query
```
SELECT c.customerID, c.companyName, COALESCE(SUM(od.unitPrice * od.quantity * (1 - od.discount)), 0) AS TotalSales
FROM customers c
LEFT JOIN orders o ON c.customerID = o.customerID
LEFT JOIN order_details od ON o.orderID = od.orderID
GROUP BY c.CustomerID, c.companyName;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/b6842fc8-2cb5-4590-91ba-b2108bf9b357)

##### MongoDB Query
```
db.customers.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "customerID",
      foreignField: "customerID",
      as: "orders",
    },
  },
  {
    $unwind: {
      path: "$orders",
      preserveNullAndEmptyArrays: true,
    },
  },
  {
    $lookup: {
      from: "order_details",
      localField: "orders.orderID",
      foreignField: "orderID",
      as: "order_details",
    },
  },
  {
    $unwind: {
      path: "$order_details",
      preserveNullAndEmptyArrays: true,
    },
  },
  {
    $group: {
      _id: {
        customerID: "$customerID",
        companyName: "$companyName",
      },
      TotalSales: {
        $sum: {
          $multiply: [
            "$order_details.unitPrice",
            "$order_details.quantity",
            {
              $subtract: [
                1,
                "$order_details.discount",
              ],
            },
          ],
        },
      },
    },
  },
  {
    $project: {
      _id: 0,
      customerID: "$_id.customerID",
      companyName: "$_id.companyName",
      TotalSales: 1,
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/c3f1c5e1-aaa1-4efd-9091-ee1c249a8320)



#### 15. List all products that need to be re-ordered. Do not include discontinued products in this report
##### SQL Query
```
SELECT *
FROM products
WHERE unitsInStock + unitsOnOrder <= reorderLevel
  AND discontinued = 0;
```
##### SQL Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/09bbc4ae-5b07-46a4-b0ee-0d7000670c3e)

##### MongoDB Query
```
db.products.aggregate([
  {
    $match: {
      $expr: {
        $and: [
          {
            $eq: ["$discontinued", 0],
          },
          {
            $lte: [
              {
                $add: [
                  "$unitsInStock",
                  "$unitsOnOrder",
                ],
              },
              "$reorderLevel",
            ],
          },
        ],
      },
    },
  },
])
```
##### MnogoDB Output
![image](https://github.com/dhruv-mehta99/vebholics-assignment/assets/64527640/6f25044c-86e3-4b18-bcc4-0a79d7cc6abc)



