# Coding Interview Practice

**C# | SQL Server | Angular / TypeScript**

এই ফোল্ডারটি hands-on practice-এর জন্য। প্রতিটি problem নিজে solve করে test case চালাবে।

## 1. C# Coding Problems

### Problem 1: String Reverse
একটি string reverse করো।

```csharp
string Reverse(string input)
{
    char[] chars = input.ToCharArray();
    Array.Reverse(chars);
    return new string(chars);
}
```

### Problem 2: Palindrome Check

```csharp
bool IsPalindrome(string input)
{
    string value = input.ToLowerInvariant();
    return value.SequenceEqual(value.Reverse());
}
```

### Problem 3: Second Largest Distinct Number

```csharp
int? SecondLargest(IEnumerable<int> numbers)
{
    return numbers.Distinct()
                  .OrderByDescending(x => x)
                  .Skip(1)
                  .Cast<int?>()
                  .FirstOrDefault();
}
```

### Problem 4: Character Frequency

```csharp
var frequency = "developer"
    .GroupBy(c => c)
    .ToDictionary(g => g.Key, g => g.Count());
```

### Problem 5: Remove Duplicates

```csharp
int[] RemoveDuplicates(int[] values) =>
    values.Distinct().ToArray();
```

### আরও C# Practice
1. Array থেকে duplicate বের করা।
2. Missing number খুঁজে বের করা।
3. Prime number check।
4. Factorial iterative ও recursive।
5. Fibonacci series।
6. Anagram check।
7. Word frequency count।
8. Two Sum।
9. Merge two sorted arrays।
10. Linked list reverse।
11. Balanced parentheses।
12. Binary search।
13. Sorting algorithm implement।
14. LINQ দিয়ে group ও aggregate।
15. Async method এবং cancellation implement।

## 2. SQL Server Interview Queries

নিচের উদাহরণে table ধরা হয়েছে: `Employees(Id, Name, Salary, DepartmentId, CreatedDate)`।

### Problem 1: দ্বিতীয় সর্বোচ্চ Salary

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM dbo.Employees
WHERE Salary < (SELECT MAX(Salary) FROM dbo.Employees);
```

### Problem 2: Duplicate Name

```sql
SELECT Name, COUNT(*) AS Total
FROM dbo.Employees
GROUP BY Name
HAVING COUNT(*) > 1;
```

### Problem 3: Department-wise Employee Count

```sql
SELECT DepartmentId, COUNT(*) AS TotalEmployees
FROM dbo.Employees
GROUP BY DepartmentId;
```

### Problem 4: Top 3 Salary per Department

```sql
WITH RankedEmployees AS
(
    SELECT *,
           DENSE_RANK() OVER
           (PARTITION BY DepartmentId ORDER BY Salary DESC) AS SalaryRank
    FROM dbo.Employees
)
SELECT *
FROM RankedEmployees
WHERE SalaryRank <= 3;
```

### Problem 5: Department ছাড়া Employee

```sql
SELECT e.*
FROM dbo.Employees e
LEFT JOIN dbo.Departments d ON d.Id = e.DepartmentId
WHERE d.Id IS NULL;
```

### আরও SQL Practice
1. Nth highest salary।
2. Duplicate row identify ও safe delete।
3. গত ৩০ দিনে যোগ দেওয়া employee।
4. Customer যাদের কোনো order নেই।
5. প্রতিটি department-এর highest salary।
6. প্রতি মাসে মোট sales।
7. Running total।
8. Consecutive date records।
9. OFFSET/FETCH pagination।
10. `ROW_NUMBER`, `RANK`, `DENSE_RANK` ব্যবহার।
11. CTE দিয়ে hierarchy query।
12. Index দিয়ে slow query optimize।

## 3. Angular / TypeScript Practice

1. Employee CRUD screen তৈরি।
2. Reactive Form ও custom validator।
3. Search box-এ `debounceTime` ও `switchMap`।
4. HTTP interceptor দিয়ে global error handling।
5. Route guard দিয়ে protected page।
6. Lazy-loaded feature route।
7. Signal দিয়ে counter ও derived state।
8. Server-side pagination, sorting ও filtering।
9. Parent-child component communication।
10. Loading, empty ও error state তৈরি।
11. Dynamic FormArray।
12. API response strongly typed model-এ map করা।

### Search Example

```typescript
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.employeeService.search(term))
);
```

## 4. Practice করার নিয়ম

প্রতিটি problem-এর জন্য:
1. Problem statement বুঝবে।
2. Input/output ও edge case লিখবে।
3. প্রথমে নিজে solution করবে।
4. Empty input, duplicate, null এবং large input test করবে।
5. Time ও space complexity ব্যাখ্যা করবে।
6. Alternative solution-এর trade-off বুঝবে।

## 5. Complexity Quick Reference

| Complexity | নাম | উদাহরণ |
|---|---|---|
| O(1) | Constant | Array index access |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Single loop |
| O(n log n) | Linearithmic | Efficient comparison sort |
| O(n²) | Quadratic | Nested loop over same input |

**লক্ষ্য:** শুধু code compile করানো নয়—correctness, edge case, readability এবং complexity ব্যাখ্যা করতে শেখা।
