# SQL Server & SSMS Interview Questions

**SQL Query | Stored Procedure | Index | Transaction | Optimization**

## 1. Database Fundamentals

### Q1. SQL Server কী?
Microsoft-এর Relational Database Management System (RDBMS)। Structured data সংরক্ষণ, query, transaction ও security পরিচালনা করে।

### Q2. SSMS কী?
SQL Server Management Studio হলো SQL Server manage, query execute, database object তৈরি ও performance troubleshoot করার graphical tool।

### Q3. Primary Key কী?
প্রতিটি row-কে uniquely identify করে। Primary key column NULL হতে পারে না; একটি table-এ একটি primary key constraint থাকে, যা একাধিক column-এর composite হতে পারে।

### Q4. Foreign Key কী?
এক table-এর column অন্য table-এর candidate/primary key-এর সঙ্গে relationship তৈরি করে এবং referential integrity বজায় রাখে।

### Q5. UNIQUE constraint কী?
একটি column বা column combination-এর value unique রাখে। SQL Server-এ single-column UNIQUE constraint সাধারণত একটি NULL value অনুমোদন করে।

## 2. JOIN ও Query

### Q6. JOIN কত প্রকার?
- INNER JOIN: দুই পাশে match থাকা row।
- LEFT JOIN: বাম পাশের সব row, ডান পাশের match থাকলে data।
- RIGHT JOIN: ডান পাশের সব row।
- FULL OUTER JOIN: উভয় পাশের সব row।
- CROSS JOIN: Cartesian product।

```sql
SELECT e.Id, e.Name, d.DepartmentName
FROM dbo.Employees AS e
LEFT JOIN dbo.Departments AS d
    ON d.Id = e.DepartmentId;
```

### Q7. WHERE ও HAVING-এর পার্থক্য?
`WHERE` grouping-এর আগে row filter করে; `HAVING` GROUP BY-এর পর group filter করে।

```sql
SELECT DepartmentId, COUNT(*) AS Total
FROM dbo.Employees
WHERE IsActive = 1
GROUP BY DepartmentId
HAVING COUNT(*) >= 5;
```

### Q8. UNION ও UNION ALL?
`UNION` duplicate row বাদ দেয়; `UNION ALL` duplicate-সহ সব row ফেরত দেয়। Duplicate বাদ দেওয়ার প্রয়োজন না থাকলে UNION ALL সাধারণত কম কাজ করে।

### Q9. NULL কীভাবে check করবে?
`IS NULL` বা `IS NOT NULL` ব্যবহার করবে; `= NULL` নয়।

### Q10. CTE কী?
Common Table Expression হলো একটি statement-এর scope-এ নাম দেওয়া query result। Recursive hierarchy query-তেও ব্যবহার হয়।

```sql
WITH ActiveEmployees AS
(
    SELECT Id, Name
    FROM dbo.Employees
    WHERE IsActive = 1
)
SELECT * FROM ActiveEmployees;
```

## 3. Index ও Performance

### Q11. Index কী?
Index data খুঁজে পেতে সাহায্য করে। তবে অতিরিক্ত index storage নেয় এবং INSERT/UPDATE/DELETE-এর খরচ বাড়ায়।

### Q12. Clustered ও Nonclustered Index?
- Clustered index-এর leaf level table data page ধারণ করে; table-এ একটি clustered index হতে পারে।
- Nonclustered index আলাদা structure; একাধিক হতে পারে।
- Heap table-এ clustered index নাও থাকতে পারে।

### Q13. Composite Index কী?
একাধিক key column নিয়ে index তৈরি হয়। Column order গুরুত্বপূর্ণ।

```sql
CREATE INDEX IX_Employees_Department_Status
ON dbo.Employees(DepartmentId, Status);
```

### Q14. Index Seek ও Index Scan?
Seek নির্দিষ্ট key range-এ navigate করে; scan index/table-এর range বা সব অংশ পড়ে। Scan সবসময় খারাপ নয়—বেশি row দরকার হলে scan কার্যকর হতে পারে।

### Q15. SARGable query কী?
যে predicate index seek-এর উপযোগী হতে পারে তাকে SARGable বলে।

কম index-friendly:

```sql
WHERE YEAR(CreatedDate) = 2026
```

বেশি index-friendly:

```sql
WHERE CreatedDate >= '20260101'
  AND CreatedDate <  '20270101'
```

### Q16. Execution Plan কী?
SQL Server query চালাতে কোন operator, join ও access method ব্যবহার করছে তার পরিকল্পনা। Actual Execution Plan, logical reads ও duration বিশ্লেষণ করো।

## 4. Transactions ও Concurrency

### Q17. ACID কী?
- Atomicity: সব operation commit, নয়তো rollback।
- Consistency: constraint/business invariant বজায় থাকে।
- Isolation: concurrent transaction-এর interaction নিয়ন্ত্রিত।
- Durability: commit হওয়া data স্থায়ী।

### Q18. Isolation Level কী কী?
READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SNAPSHOT, SERIALIZABLE। Isolation বাড়ালে consistency বাড়তে পারে, কিন্তু blocking/concurrency trade-off থাকে।

### Q19. Deadlock কী?
দুই বা ততোধিক transaction একে অপরের lock করা resource-এর জন্য অপেক্ষা করলে deadlock হতে পারে। SQL Server victim transaction rollback করে।

প্রতিরোধ/কমাতে:
- একই order-এ resource access
- ছোট transaction
- উপযুক্ত index
- অপ্রয়োজনীয় lock এড়ানো
- deadlock graph বিশ্লেষণ

### Q20. NOLOCK কী?
`READ UNCOMMITTED`-এর সমতুল্য table hint। Dirty/inconsistent read হতে পারে; performance fix হিসেবে নির্বিচারে ব্যবহার করা উচিত নয়।

## 5. Stored Procedure ও Functions

### Q21. Stored Procedure কী?

```sql
CREATE OR ALTER PROCEDURE dbo.GetEmployeeById
    @Id INT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT Id, Name
    FROM dbo.Employees
    WHERE Id = @Id;
END;
```

### Q22. Procedure ও Function-এর পার্থক্য?
Procedure `EXEC` দিয়ে চালানো হয় এবং output parameter/result set দিতে পারে। Function query expression-এ ব্যবহার করা যায় এবং scalar বা table value ফেরত দেয়; function-এর data modification-এ বিধিনিষেধ আছে।

### Q23. DELETE, TRUNCATE, DROP?
- DELETE: row delete; WHERE দেওয়া যায়।
- TRUNCATE: table-এর সব row দ্রুত remove; WHERE নেই, identity reset হয়।
- DROP: object definition-সহ object remove করে।
- Transaction, FK, permission ও object type অনুযায়ী আচরণ বিবেচনা করতে হবে।

## 6. Window Functions ও Common Queries

### Q24. ROW_NUMBER, RANK, DENSE_RANK?
- ROW_NUMBER: প্রতিটি row-কে unique sequential number।
- RANK: tie হলে একই rank, পরের rank-এ gap।
- DENSE_RANK: tie হলে একই rank, gap নেই।

```sql
SELECT
    Name,
    Salary,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS SalaryRank
FROM dbo.Employees;
```

### Q25. দ্বিতীয় সর্বোচ্চ salary কীভাবে বের করবে?

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM dbo.Employees
WHERE Salary < (SELECT MAX(Salary) FROM dbo.Employees);
```

### Q26. Department অনুযায়ী top 3 salary?

```sql
WITH Ranked AS
(
    SELECT *,
           ROW_NUMBER() OVER
           (PARTITION BY DepartmentId ORDER BY Salary DESC, Id) AS rn
    FROM dbo.Employees
)
SELECT *
FROM Ranked
WHERE rn <= 3;
```

### Q27. Pagination কীভাবে করবে?

```sql
SELECT Id, Name
FROM dbo.Employees
ORDER BY Id
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

Stable pagination-এর জন্য deterministic ORDER BY দরকার। খুব বড় page offset-এর ক্ষেত্রে keyset pagination বিবেচনা করা যায়।

## Query Optimization Checklist

- [ ] Actual Execution Plan
- [ ] Logical reads / IO statistics
- [ ] Suitable index ও index selectivity
- [ ] SARGable predicates
- [ ] প্রয়োজনীয় column select
- [ ] Join cardinality ও statistics
- [ ] Parameter-sensitive plan
- [ ] Blocking / deadlock
- [ ] Pagination strategy
- [ ] Realistic data দিয়ে benchmark
