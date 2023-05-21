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

