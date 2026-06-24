### Data_Engineering - SQL

### Database
collection of data that is organised in a specific manner to facilitate easy access, retrieval and management.

### DBMS 
-is a software application that enables users to create, manage and access database.
-it provide an interface to the user to interact with database, allowing them to perform tasks such as addition, modification and deleting data

### Database Types
	1. Relational Database (RDBMS)
	2. Non-relational DB (NoSQL)


### 1. Relational Database (RDBMS)
   - Data stored in tables with defined schemas.
   - type of DBMS that organise data into one or more tables, each consisting of rows and columns.(Structured)
   - SQL (structured Query Language) is the primary language used to interact with the relational DB.
   - Eg: Oracle, MySQL, SQL server.
RDMS was developed by Edgar F. Codd in 1970 while working at IBM.

### Characteristics
Data must be stored in tabular form in DB files.
Each row called record/tuples. collection of such rows called cardinality
Each column of the table called attribute/field. Collection of such columns is called the arity of the table.
No two record of the db can be the same.
Tables related to each other with each other with foreign keys.
DB allows null values ,but primary key cannot have null value.


SQL - a query language used to manage and manipulate relational DB.
    - allows a user to create, modify and query db using commends such as select, insert, update and delete.

### 2. Non-relational DB (NoSQL-not only SQL)'
   - Also known as NoSQL
   - Flexible schema. Designed for scale and speed.
   - Designed to handle unstructured data that required high scalability and performance.
Types
Document (MongoDB)
Key-Value (Redis)
Column-family (Cassandra)
Graph (Neo4j)

   - No SQL db cannot handle SQL statement directly, some NoSQL db provide SQL like query languages that allow users to perform SQL-like queries.
Eg: Appache Cassandra provides SQL-like query language called CQL(Cassandra query language)

SQL
SQL - a query language used to manage and manipulate relational DB.
    - allows a user to create, modify and query db using commends such as select, insert, update and delete.

Installation - SQL Server Management Studio 22, Virtual Studio installer.

### Examples 
Create Database
Create database SQL_1;
use SQL_1;

### Create Table

CREATE TABLE EmployeeData (
EmpID INT PRIMARY KEY,
FirstName VARCHAR(50),
LastName VARCHAR(50),
Gender VARCHAR(10),
Age INT,
Department VARCHAR(50),
JobTitle VARCHAR(50),
Salary INT,
Experience INT,
City VARCHAR(50),
ManagerID INT,
JoinDate DATE
);

Insert Values
INSERT INTO EmployeeData VALUES
(1, 'Rahul', 'Sharma', 'Male', 28, 'IT', 'Software Engineer', 55000, 3, 'Mumbai', 101, '2021-05-10'),
(2, 'Priya', 'Mehta', 'Female', 32, 'HR', 'HR Manager', 65000, 7, 'Delhi', NULL, '2019-03-15'),
(3, 'Amit', 'Verma', 'Male', 45, 'Finance', 'Accountant', 48000, 15, 'Pune', 102, '2010-11-01'),
(4, 'Sneha', 'Patil', 'Female', 29, 'IT', 'Data Analyst', 52000, 4, 'Bangalore', 101, '2020-08-20'),
(5, 'Karan', 'Singh', 'Male', 35, 'Sales', 'Sales Executive', 40000, 8, 'Chennai', 103, '2016-06-18'),
(6, 'Divya', 'Iyer', 'Female', 26, 'Marketing', 'Marketing Executive', 38000, 2, 'Mumbai', 104,'2022-01-12'),
(7, 'Rohit', 'Nair', 'Male', 41, 'IT', 'Team Lead', 72000, 12, 'Delhi', 101, '2012-09-10'),
(8, 'Anjali', 'Gupta', 'Female', 30, 'Finance', 'Financial Analyst', 58000, 5, 'Pune', 102,'2018-04-22'),
(9, 'Vijay', 'Kumar', 'Male', 27, 'Sales', 'Sales Associate', 35000, 2, 'Chennai', 103, '2021-09-05'),
(10, 'Neha', 'Reddy', 'Female', 33, 'HR', 'HR Executive', 45000, 6, 'Hyderabad', 105, '2017-12-01'),
(11, 'Suresh', 'Menon', 'Male', 38, 'IT', 'System Admin', 50000, 10, 'Bangalore', 101, '2014-07-19'),
(12, 'Pooja', 'Joshi', 'Female', 24, 'Marketing', 'SEO Specialist', 36000, 1, 'Mumbai', 104,'2023-02-14'),
(13, 'Manoj', 'Desai', 'Male', 40, 'Finance', 'Senior Accountant', 62000, 14, 'Delhi', 102,'2011-10-30'),
(14, 'Kavya', 'Shah', 'Female', 28, 'IT', 'Backend Developer', 56000, 4, 'Pune', 101, '2019-06-15'),
(15, 'Harish', 'Rao', 'Male', 34, 'Sales', 'Sales Manager', 70000, 9, 'Hyderabad', 103, '2015-03-20'),
(16, 'Shruti', 'Kale', 'Female', 31, 'HR', 'Recruiter', 43000, 5, 'Delhi', 105, '2018-09-11'),
(17, 'Arjun', 'Mishra', 'Male', 29, 'Marketing', 'Content Writer', 37000, 3, 'Mumbai', 104,'2020-12-05'),
(18, 'Ritika', 'Soni', 'Female', 27, 'IT', 'Frontend Developer', 54000, 3, 'Bangalore', 101,'2021-03-09'),
(19, 'Vikas', 'Yadav', 'Male', 36, 'Finance', 'Auditor', 60000, 10, 'Pune', 102, '2013-05-01'),
(20, 'Meera', 'Pai', 'Female', 25, 'Sales', 'Sales Trainee', 30000, 1, 'Chennai', 103, '2022-06-22'),
(21, 'Lokesh', 'Bhat', 'Male', 39, 'IT', 'Cloud Engineer', 68000, 11, 'Hyderabad', 101, '2014-02-17'),
(22, 'Simran', 'Arora', 'Female', 30, 'HR', 'HR Coordinator', 42000, 4, 'Delhi', NULL, '2020-07-01'),
(23, 'Nitin', 'Sawant', 'Male', 33, 'Marketing', 'Brand Manager', 59000, 7, 'Mumbai', 104,'2016-11-23'),
(24, 'Ayesha', 'Khan', 'Female', 28, 'Finance', 'Tax Consultant', 53000, 4, 'Hyderabad', 102,'2019-04-16'),
(25, 'Sanjay', 'Puri', 'Male', 45, 'Sales', 'Regional Sales Head', 75000, 18, 'Delhi', 103,'2008-08-08'),
(26, 'Julie', 'Dcruz', 'Female', 26, 'Marketing', 'Digital Marketer', 39000, 2, 'Mumbai', 104,'2021-01-25'),
(27, 'Rohan', 'Kulkarni', 'Male', 31, 'IT', 'DevOps Engineer', 61000, 6, 'Bangalore', 101,'2017-05-29'),
(28, 'Tanya', 'Shetty', 'Female', 34, 'Finance', 'Risk Analyst', 64000, 9, 'Pune', 102, '2015-10-10'),
(29, 'Akash', 'Mali', 'Male', 23, 'HR', 'HR Intern', 22000, 0, 'Mumbai', 105, '2024-01-10'),
(30, 'Reema', 'Jain', 'Female', 29, 'IT', 'QA Engineer', 50000, 5, 'Chennai', 101, '2019-12-19');


### SQL Keys and Constraints

### Keys
Keys are attributes/columns used to identify records and maintain relationships
   between tables.
1 Primary Key (PK)
A column or set of columns that uniquely identifies each row in a table.
✔ Features:
• Must be unique
• Cannot be NULL
• Only one primary key per table (but can be composite)
✔ Example:
CREATE TABLE Employees (
EmpID INT PRIMARY KEY,
EmpName VARCHAR(50)
);

2.Foreign Key (FK)
A column that links one table to another.
It references the Primary Key of another table.
✔ Used for:
• Maintaining referential integrity
• Creating relationships (1–many, many–many)
✔ Example:
CREATE TABLE Orders (
OrderID INT PRIMARY KEY,
EmpID INT,

FOREIGN KEY (EmpID) REFERENCES Employees(EmpID)
);

3. Candidate Key
They are the candidate of being primary key.

4. Alternate Key
From the set of candidate keys, the key other than primary key is called alternate key.

### Constraints

Rules enforced on database columns to maintain data accuracy and integrity. They help prevent invalid data entries.

The following constraints are commonly used in SQL:

NOT NULL - Ensures that a column cannot have a NULL value
UNIQUE - Ensures that all values in a column are unique
PRIMARY KEY - Uniquely identifies each row in a table (a combination of a NOT NULL and UNIQUE)
FOREIGN KEY - Establishes a link between data in two tables, and prevents action that will destroy the link between them
CHECK - Ensures that the values in a column satisfies a specific condition
DEFAULT - Sets a default value for a column if no value is specified
CREATE INDEX - Creates indexes on columns to retrieve data from the database faster

### SQL Commands

1. DDL – Data Definition Language
Used to define or modify database structures (tables, schemas, indexes).

Command	Purpose	Example
CREATE	- Create database objects			CREATE TABLE Students (ID INT PRIMARY KEY, Name VARCHAR(50));
ALTER	- Modify existing objects			ALTER TABLE Students ADD Age INT;
DROP	- Delete objects				DROP TABLE Students;
TRUNCATE- Remove all data but keep structure		TRUNCATE TABLE Students;
RENAME	 -Rename objects				ALTER TABLE Students RENAME TO Learners;


2. DML – Data Manipulation Language
Used to insert, update, delete data in tables.

Command	Purpose	Example
INSERT	-Add new records					INSERT INTO Students (ID, Name, Age) VALUES (1, 'Amit', 20);
UPDATE	-Modify existing records				UPDATE Students SET Age = 21 WHERE ID = 1;
DELETE	-Remove records						DELETE FROM Students WHERE ID = 1;

3. DQL – Data Query Language
Used to retrieve data from the database.

Command	Purpose	Example
SELECT	Fetch data						SELECT Name, Age FROM Students WHERE Age > 18;

4. DCL – Data Control Language
Used to control access to data.

### Command	Purpose	Example
GRANT	Give permissions					GRANT SELECT ON Students TO user1;
REVOKE	Remove permissions					REVOKE SELECT ON Students FROM user1;

5. TCL – Transaction Control Language
Used to manage transactions in a database.

### Command	Purpose	Example
COMMIT	-Save changes	COMMIT;
ROLLBACK -Undo changes	ROLLBACK;
SAVEPOINT -Set a point to rollback to	SAVEPOINT sp1;
SET TRANSACTION	- Define transaction properties	SET TRANSACTION READ ONLY;

### Tip:
DDL commands auto-commit (changes are permanent immediately).
DML commands need COMMIT or ROLLBACK to finalize or undo changes.