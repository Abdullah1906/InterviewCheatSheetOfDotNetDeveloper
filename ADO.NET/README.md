# ADO.NET --- Professional Interview & Practical Handbook

> **Stack:** C# · .NET 8 · SQL Server · Microsoft.Data.SqlClient ·
> Stored Procedures\
> **Level:** Beginner → Intermediate → Advanced\
> **Purpose:** Interview preparation, practical implementation, and
> production-oriented database access.

## Table of Contents

1.  Fundamentals and architecture
2.  Connection management and pooling
3.  Commands and execution methods
4.  DataReader, DataTable, DataSet and DataAdapter
5.  Parameters and SQL Injection
6.  CRUD with stored procedures
7.  Transactions and ACID
8.  Async, cancellation and resource management
9.  Advanced ADO.NET
10. Exceptions and reliability
11. Performance optimization
12. Security
13. ASP.NET Core and Clean Architecture
14. ADO.NET vs Dapper vs EF Core
15. Real-world scenarios
16. Interview quick revision
17. Practical coding exercises
18. Common mistakes and best practices
19. Final checklist

------------------------------------------------------------------------

# 1. Fundamentals and Architecture

## Q1. What is ADO.NET?

**Short answer:** ADO.NET হলো .NET application থেকে relational database
access করার data-access technology।

**বিস্তারিত:** এটি connection, command, parameter, reader এবং
transaction-এর মতো API দেয়। SQL Server-এর জন্য `Microsoft.Data.SqlClient`
ব্যবহার করা যায়। Application SQL statement বা stored procedure পাঠায়;
database result ফেরত দেয়; ADO.NET সেই result পড়তে ও map করতে সাহায্য করে।

**কেন দরকার?** Database access-এর ওপর সরাসরি control, parameter binding,
transaction এবং efficient data retrieval-এর জন্য।

## Q2. What is a .NET Data Provider?

Data Provider হলো database-specific API-এর collection, যা .NET
application-কে নির্দিষ্ট database-এর সঙ্গে যোগাযোগ করতে দেয়।

-   SQL Server: `Microsoft.Data.SqlClient`
-   SQLite: `Microsoft.Data.Sqlite`
-   PostgreSQL: Npgsql

## Q3. What is Microsoft.Data.SqlClient?

এটি SQL Server ও Azure SQL Database-এর জন্য Microsoft-এর .NET data
provider। এর মাধ্যমে `SqlConnection`, `SqlCommand`, `SqlDataReader`,
`SqlTransaction` ইত্যাদি ব্যবহার করা হয়।

``` csharp
using Microsoft.Data.SqlClient;

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
```

## Q4. What is SqlConnection?

`SqlConnection` SQL Server database-এর সঙ্গে logical connection পরিচালনা
করে। এটি connection string ব্যবহার করে server, database এবং
authentication information পায়।

## Q5. What is SqlCommand?

`SqlCommand` SQL statement, stored procedure বা supported command
execute করে। এটি একটি `SqlConnection`-এর সঙ্গে কাজ করে।

## Q6. What is SqlDataReader?

এটি database result set sequentially পড়ার জন্য forward-only, read-only
API। সাধারণত বড় result set পড়ার ক্ষেত্রে memory-efficient, কারণ সব row
একসঙ্গে memory-তে load করে না।

## Q7. What is SqlDataAdapter?

`SqlDataAdapter` database এবং disconnected `DataSet`/`DataTable`-এর মধ্যে
bridge হিসেবে কাজ করে। `Fill()` দিয়ে data আনা এবং configured commands
দিয়ে update করা যায়।

## Q8. What is DataTable?

`DataTable` হলো memory-তে থাকা tabular data structure। এতে rows,
columns, constraints এবং data type থাকে।

## Q9. What is DataSet?

`DataSet` হলো memory-resident collection of DataTables, relations এবং
constraints। এটি একাধিক related table disconnected অবস্থায় ধরে রাখতে
পারে।

## Q10. Connected vs Disconnected architecture?

  -----------------------------------------------------------------------
  Connected                           Disconnected
  ----------------------------------- -----------------------------------
  Data পড়ার সময় connection সাধারণত    Data memory-তে load করার পর
  open থাকে                           connection বন্ধ করা যায়

  উদাহরণ: `SqlDataReader`             উদাহরণ: `DataTable`, `DataSet`

  Streaming-এর জন্য উপযোগী             Offline/in-memory manipulation-এর
                                      জন্য উপযোগী

  সাধারণত কম memory লাগে              বড় dataset হলে বেশি memory লাগে
  -----------------------------------------------------------------------

## Q11. Is ADO.NET an ORM?

না। ADO.NET নিজে ORM নয়; এটি low-level data-access API। ORM যেমন EF Core
object-relational mapping করে। Dapper হলো ADO.NET-এর ওপর তৈরি
lightweight object mapper।

## Q12. Is ADO.NET obsolete?

না। ADO.NET এখনও .NET-এর গুরুত্বপূর্ণ data-access foundation। Dapper-ও
ADO.NET-এর ওপর নির্ভর করে। প্রয়োজন অনুযায়ী ADO.NET, Dapper বা EF Core বেছে
নিতে হয়।

## Q13. Explain the ADO.NET execution flow.

``` text
ASP.NET Core Controller
        |
        v
Application Service
        |
        v
Repository
        |
        v
SqlConnection -> SqlCommand -> SQL Server
                         |
                         v
                 DataReader / Scalar
                         |
                         v
                    C# Model
```

------------------------------------------------------------------------

# 2. Connection Management and Pooling

## Q14. How do you create and open a SQL connection?

``` csharp
using Microsoft.Data.SqlClient;

var connectionString =
    "Server=.;Database=CompanyDb;Integrated Security=True;TrustServerCertificate=True;";

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
```

`OpenAsync()` asynchronousভাবে connection open করার চেষ্টা করে।
Production-এ connection string configuration থেকে নিতে হবে; secret
hardcode করা যাবে না।

## Q15. What is Connection Pooling?

Connection Pooling হলো একই configuration-এর database connection পুনরায়
ব্যবহার করার ব্যবস্থা। Connection dispose/close করলে physical connection
সাধারণত pool-এ ফেরত যায়। ফলে পরবর্তী request-এ নতুন physical connection
তৈরির overhead কমতে পারে।

**মনে রাখবে:** `Close()` বা `Dispose()` করা মানেই প্রতিবার physical
network connection destroy করা নয়।

## Q16. How does Connection Pooling work?

1.  Application `Open()` বা `OpenAsync()` call করে।
2.  Provider matching pool-এ available connection খোঁজে।
3.  Available connection থাকলে reuse হয়।
4.  না থাকলে pool limit অনুমতি দিলে নতুন physical connection তৈরি হয়।
5.  Connection close/dispose হলে pool-এ ফেরত যায়।

Pool matching connection string ও অন্যান্য connection configuration-এর ওপর
নির্ভর করে।

## Q17. What is Max Pool Size?

একটি connection pool-এ সর্বোচ্চ কত connection রাখা/ব্যবহার করা যাবে তা
নির্ধারণ করে। Limit-এ পৌঁছে গেলে connection available না হওয়া পর্যন্ত নতুন
open request wait করতে পারে এবং timeout হতে পারে।

## Q18. What is Min Pool Size?

Pool-এ ন্যূনতম কত connection ধরে রাখার configuration। সব application-এ এটি
বাড়ানো দরকার হয় না; workload ও resource usage বুঝে নির্ধারণ করতে হয়।

## Q19. What causes connection pool exhaustion?

-   Connection dispose/close না করা।
-   দীর্ঘ সময় connection open রাখা।
-   Slow query বা blocking-এর কারণে connection আটকে থাকা।
-   অতিরিক্ত concurrent request।
-   অপ্রয়োজনীয়ভাবে অনেক আলাদা connection string তৈরি করা।

**সমাধান:** `using`/`await using`, short-lived connection scope, query
optimization, concurrency control এবং pool metrics/log পরীক্ষা।

## Q20. Should a SqlConnection be static or shared?

সাধারণত না। `SqlConnection` object request বা operation scope-এ তৈরি করে
কাজ শেষে dispose করো। Pooling physical connection reuse করে। Shared
mutable connection concurrency ও state-management সমস্যা তৈরি করতে পারে।

## Q21. Difference between Close() and Dispose()?

দুটিই connection resource release করে এবং pooled connection ফেরত দেওয়ার
workflow-এ অংশ নেয়। `Dispose()` object-এর lifetime শেষ করে; তাই `using`
pattern পরিষ্কার। Disposed object reuse করা উচিত নয়।

## Q22. Connection Timeout vs CommandTimeout?

  -----------------------------------------------------------------------
  Connection Timeout                  CommandTimeout
  ----------------------------------- -----------------------------------
  Database connection establish করার  Command execute করার সময়সীমা
  সময়সীমা                             

  Connection string-এ configure হয়    `SqlCommand.CommandTimeout`

  Connection opening-এ প্রযোজ্য         Query/SP execution-এ প্রযোজ্য
  -----------------------------------------------------------------------

`CommandTimeout` default সাধারণত 30 seconds। `0` দিলে timeout
নেই---সতর্কতা ছাড়া ব্যবহার করা উচিত নয়।

## Q23. What happens when the connection pool is exhausted?

Pool limit-এ পৌঁছে গেলে নতুন open request available connection-এর জন্য
অপেক্ষা করে। নির্দিষ্ট সময়ের মধ্যে connection না পেলে pool acquisition
timeout হয়। আগে leak, long-running query, blocking ও concurrency
investigate করো; শুধু pool size বাড়িও না।

------------------------------------------------------------------------

# 3. Commands and Execution Methods

## Q24. What is CommandType?

`CommandType` command কীভাবে interpret হবে তা নির্ধারণ করে।

-   `Text`: SQL text execute হবে।
-   `StoredProcedure`: Stored procedure execute হবে।
-   `TableDirect`: Provider support করলে table access; SQL Server
    application-এ সাধারণত ব্যবহার করা হয় না।

## Q25. ExecuteReader vs ExecuteScalar vs ExecuteNonQuery?

  -----------------------------------------------------------------------
  Method                  ব্যবহার                  Return
  ----------------------- ----------------------- -----------------------
  `ExecuteReader()`       এক বা একাধিক result set `SqlDataReader`
                          পড়া                     

  `ExecuteScalar()`       একটি value, যেমন COUNT  প্রথম result set-এর প্রথম
                          বা generated ID         row-এর প্রথম column

  `ExecuteNonQuery()`     INSERT/UPDATE/DELETE বা Affected rows বা
                          row-returning নয় এমন    provider-specific
                          command                 result
  -----------------------------------------------------------------------

``` csharp
await using var reader = await command.ExecuteReaderAsync();
object? value = await command.ExecuteScalarAsync();
int affectedRows = await command.ExecuteNonQueryAsync();
```

## Q26. When should you use ExecuteScalar?

যখন শুধু একটি value দরকার, যেমন total employee count, aggregate result বা
`SCOPE_IDENTITY()`-এর generated ID।

``` csharp
await using var command = new SqlCommand(
    "SELECT COUNT(*) FROM dbo.Employees", connection);

int count = Convert.ToInt32(await command.ExecuteScalarAsync());
```

## Q27. Does ExecuteNonQuery always return affected rows?

না। `SET NOCOUNT ON`, stored procedure, trigger, multiple statements বা
provider behavior-এর কারণে result সবসময় সরলভাবে interpret করা যায় না।
Business success যাচাইয়ের জন্য explicit output/status parameter ব্যবহার
করা ভালো।

## Q28. What is CommandBehavior?

`ExecuteReader()`-এর behavior control করার enum। উদাহরণ:
`CloseConnection`, `SingleRow`, `SingleResult`, `SequentialAccess`।

## Q29. What is CommandBehavior.CloseConnection?

Reader close করলে associated connection-ও close হবে। তবে সাধারণ
application code-এ connection ও reader একই scope-এ dispose করা বেশি
পরিষ্কার।

## Q30. Can one SqlCommand execute multiple SQL statements?

হ্যাঁ, `CommandType.Text`-এ একাধিক statement পাঠানো সম্ভব। তবে
parameterization, transaction, result-set handling এবং least privilege
নিশ্চিত করতে হবে।

------------------------------------------------------------------------

# 4. DataReader, DataTable, DataSet and DataAdapter

## Q31. Why is SqlDataReader considered fast?

এটি forward-only, read-only stream হিসেবে row পড়ে। সাধারণত পুরো result
set memory-তে materialize করে না। তাই sequential processing-এ memory
overhead কম হতে পারে।

## Q32. How do you read data using SqlDataReader?

``` csharp
await using var command = new SqlCommand(
    "SELECT Id, Name FROM dbo.Employees", connection);

await using var reader = await command.ExecuteReaderAsync();

int idOrdinal = reader.GetOrdinal("Id");
int nameOrdinal = reader.GetOrdinal("Name");

while (await reader.ReadAsync())
{
    int id = reader.GetInt32(idOrdinal);
    string name = reader.GetString(nameOrdinal);
    Console.WriteLine($"{id}: {name}");
}
```

`GetOrdinal()` column name থেকে index বের করে। Loop-এর বাইরে ordinal
cache করলে repeated lookup কমে।

## Q33. What is DBNull.Value?

`DBNull.Value` database-এর SQL NULL represent করে। C# `null`
reference/nullable value অনুপস্থিত বোঝায়। Parameter-এ SQL NULL পাঠাতে
সাধারণত `DBNull.Value` ব্যবহার করা হয়।

## Q34. How do you handle nullable columns?

``` csharp
int emailOrdinal = reader.GetOrdinal("Email");

string? email = reader.IsDBNull(emailOrdinal)
    ? null
    : reader.GetString(emailOrdinal);
```

`IsDBNull()` দিয়ে SQL NULL check করতে হয়; সরাসরি `GetString()` করলে
NULL-এর ক্ষেত্রে exception হতে পারে।

## Q35. What is NextResult()?

একই command থেকে পাওয়া পরবর্তী result set-এ যায়। Async code-এ
`NextResultAsync()` ব্যবহার করা যায়।

## Q36. What is SequentialAccess?

`CommandBehavior.SequentialAccess` বড় binary/text field sequentially
পড়তে সাহায্য করে, যাতে পুরো বড় field memory-তে একবারে load না করতে হয়।
Column access-এর order সংক্রান্ত restriction মানতে হয়।

## Q37. DataReader vs DataTable?

  -----------------------------------------------------------------------
  DataReader                          DataTable
  ----------------------------------- -----------------------------------
  Forward-only, read-only             In-memory editable table

  Streaming access                    Disconnected access

  সাধারণত কম memory                   Dataset বড় হলে বেশি memory

  Reader চলাকালে connection open থাকে Fill শেষ হলে connection বন্ধ করা যায়

  Sequential processing               Sorting/filtering/editing in memory
  -----------------------------------------------------------------------

## Q38. DataTable vs DataSet?

`DataTable` একটি table ধারণ করে। `DataSet` একাধিক `DataTable`, relation
ও constraint ধারণ করতে পারে।

## Q39. How does SqlDataAdapter.Fill() work?

`Fill()` command execute করে result set memory-র `DataTable` বা
`DataSet`-এ load করে। Provider প্রয়োজন অনুযায়ী connection open/close করতে
পারে; আগে থেকে open থাকলে সাধারণত সেটি open-ই থাকে।

``` csharp
var table = new DataTable();

using var adapter = new SqlDataAdapter(
    "SELECT Id, Name FROM dbo.Employees", connection);

adapter.Fill(table);
```

`SqlDataAdapter.Fill()` synchronous API। Async streaming দরকার হলে
`SqlDataReader`-এর async methods উপযোগী।

## Q40. Does DataAdapter automatically know how to update every DataTable?

না। Simple query-তে `SqlCommandBuilder` কিছু update command generate করতে
পারে, কিন্তু production-এ explicit Insert/Update/Delete command বা
repository operation সাধারণত বেশি predictable।

------------------------------------------------------------------------

# 5. Parameters and SQL Injection

## Q41. Why are SQL parameters important?

-   User input-কে SQL code হিসেবে interpret হওয়া থেকে রক্ষা করে।
-   Data type ও value আলাদা রাখে।
-   Query plan reuse-এ সহায়তা করতে পারে।
-   String concatenation-এর ভুল কমায়।

## Q42. What is SQL Injection?

যখন attacker user input-এর মাধ্যমে SQL statement-এর intended structure
বদলে দিতে পারে, তখন SQL Injection হয়।

**Unsafe:**

``` csharp
string sql = "SELECT * FROM dbo.Users WHERE Email = '" + email + "'";
```

**Safe:**

``` csharp
await using var command = new SqlCommand(
    "SELECT Id, Email FROM dbo.Users WHERE Email = @Email", connection);

command.Parameters.Add("@Email", SqlDbType.NVarChar, 256).Value = email;
```

## Q43. Input, Output, InputOutput and ReturnValue parameters?

-   `Input`: Application value পাঠায়।
-   `Output`: Stored procedure value ফেরত দেয়।
-   `InputOutput`: Application value পাঠায় এবং procedure সেটি পরিবর্তন
    করে ফেরত দিতে পারে।
-   `ReturnValue`: Stored procedure-এর integer return status গ্রহণ করে।

## Q44. How do you pass NULL to SQL Server?

``` csharp
command.Parameters.Add("@Phone", SqlDbType.NVarChar, 30).Value =
    phone is null ? DBNull.Value : phone;
```

## Q45. Why specify SqlDbType and size explicitly?

এতে parameter-এর database type ও size স্পষ্ট থাকে, implicit conversion ও
unexpected type inference কমে এবং query behavior predictable হয়।

``` csharp
command.Parameters.Add("@Name", SqlDbType.NVarChar, 100).Value = name;

var salary = command.Parameters.Add("@Salary", SqlDbType.Decimal);
salary.Precision = 18;
salary.Scale = 2;
salary.Value = 45000m;
```

## Q46. What is parameter sniffing?

SQL Server stored procedure বা parameterized query compile করার সময়
parameter value দেখে execution plan তৈরি করতে পারে। সেই plan অন্য
parameter value-তে optimal নাও হতে পারে। এটি SQL Server query
optimization বিষয়; ADO.NET parameterization-এর bug নয়।

## Q47. Do stored procedures automatically prevent SQL Injection?

না। Stored procedure-এ parameterized SQL নিরাপদ হতে পারে, কিন্তু untrusted
input concatenate করে dynamic SQL বানালে injection সম্ভব। `sp_executesql`
এবং properly bound parameters ব্যবহার করা উচিত।

## Q48. Can parameters replace table or column names?

সাধারণ SQL parameter দিয়ে table/column identifier bind করা যায় না।
Dynamic identifier দরকার হলে strict allow-list mapping ব্যবহার করো; user
input সরাসরি SQL string-এ বসাবে না।

------------------------------------------------------------------------

# 6. CRUD with Stored Procedures

নিচের উদাহরণে `Employees` table ব্যবহার করা হয়েছে।

## 6.1 SQL table

``` sql
CREATE TABLE dbo.Employees
(
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Email NVARCHAR(256) NOT NULL UNIQUE,
    Salary DECIMAL(18,2) NOT NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);
```

## 6.2 Insert stored procedure

``` sql
CREATE OR ALTER PROCEDURE dbo.Employee_Insert
    @Name NVARCHAR(100),
    @Email NVARCHAR(256),
    @Salary DECIMAL(18,2),
    @NewId INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO dbo.Employees (Name, Email, Salary)
    VALUES (@Name, @Email, @Salary);

    SET @NewId = CONVERT(INT, SCOPE_IDENTITY());
END;
```

## Q49. How do you execute a stored procedure?

``` csharp
await using var command =
    new SqlCommand("dbo.Employee_Insert", connection);

command.CommandType = CommandType.StoredProcedure;

command.Parameters.Add("@Name", SqlDbType.NVarChar, 100).Value = "Rahim";
command.Parameters.Add("@Email", SqlDbType.NVarChar, 256).Value = "rahim@example.com";

var salary = command.Parameters.Add("@Salary", SqlDbType.Decimal);
salary.Precision = 18;
salary.Scale = 2;
salary.Value = 45000m;

var newId = command.Parameters.Add("@NewId", SqlDbType.Int);
newId.Direction = ParameterDirection.Output;

await command.ExecuteNonQueryAsync();

int employeeId = (int)newId.Value;
```

**ব্যাখ্যা:** `CommandType.StoredProcedure` বলে command text একটি
procedure name। Parameter-এর type ও size procedure signature-এর সঙ্গে
match করে। `ExecuteNonQueryAsync()` শেষ হওয়ার পর output parameter পড়া
হয়।

## 6.3 GetById stored procedure

``` sql
CREATE OR ALTER PROCEDURE dbo.Employee_GetById
    @Id INT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT Id, Name, Email, Salary, CreatedAt
    FROM dbo.Employees
    WHERE Id = @Id;
END;
```

## Q50. How do you map a DataReader to a C# model?

``` csharp
public sealed class Employee
{
    public int Id { get; init; }
    public string Name { get; init; } = "";
    public string Email { get; init; } = "";
    public decimal Salary { get; init; }
}
```

``` csharp
public async Task<Employee?> GetByIdAsync(
    int id, CancellationToken cancellationToken)
{
    await using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync(cancellationToken);

    await using var command =
        new SqlCommand("dbo.Employee_GetById", connection);

    command.CommandType = CommandType.StoredProcedure;
    command.Parameters.Add("@Id", SqlDbType.Int).Value = id;

    await using var reader =
        await command.ExecuteReaderAsync(cancellationToken);

    if (!await reader.ReadAsync(cancellationToken))
        return null;

    return new Employee
    {
        Id = reader.GetInt32(reader.GetOrdinal("Id")),
        Name = reader.GetString(reader.GetOrdinal("Name")),
        Email = reader.GetString(reader.GetOrdinal("Email")),
        Salary = reader.GetDecimal(reader.GetOrdinal("Salary"))
    };
}
```

## Q51. How do you implement Update and Delete?

Stored procedure-এ typed parameters নিয়ে `UPDATE` বা `DELETE` চালাও।
`ExecuteNonQueryAsync()`-এর affected row count দেখে record পাওয়া গেছে কি
না বোঝা যেতে পারে, তবে `SET NOCOUNT ON` থাকলে explicit output/status
pattern ব্যবহার করা ভালো।

``` sql
UPDATE dbo.Employees
SET Name = @Name,
    Email = @Email,
    Salary = @Salary
WHERE Id = @Id;

DELETE FROM dbo.Employees
WHERE Id = @Id;
```

## Q52. Stored procedure vs parameterized SQL text?

দুটিই parameterized হলে নিরাপদভাবে execute করা যায়। Stored procedure
database-এ named object হিসেবে থাকে, permission ও deployment manage করা
যায়। Parameterized SQL text application code-এ query রাখা সহজ করে।
সিদ্ধান্ত team convention, deployment, maintainability এবং database
governance-এর ওপর নির্ভর করে।

------------------------------------------------------------------------

# 7. Transactions and ACID

## Q53. What is a database transaction?

Transaction হলো এক বা একাধিক database operation-এর logical unit।
Transaction commit হলে changes স্থায়ী হয়; rollback হলে transaction-এর
changes বাতিল হয়।

## Q54. What is ACID?

  Property      Meaning
  ------------- -----------------------------------------------------
  Atomicity     সব operation সফল, অথবা কোনোটি নয়
  Consistency   Database constraints/business rules বজায় থাকে
  Isolation     Concurrent transaction-এর interaction নিয়ন্ত্রিত থাকে
  Durability    Commit হওয়া data failure-এর পরও টিকে থাকে

## Q55. How do you implement a transaction in ADO.NET?

``` csharp
await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

await using var transaction =
    (SqlTransaction)await connection.BeginTransactionAsync();

try
{
    await using var debit = new SqlCommand(
        "UPDATE dbo.Accounts SET Balance = Balance - @Amount " +
        "WHERE Id = @FromId AND Balance >= @Amount;",
        connection, transaction);

    debit.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 500m;
    debit.Parameters.Add("@FromId", SqlDbType.Int).Value = 1;

    int debited = await debit.ExecuteNonQueryAsync();

    if (debited != 1)
        throw new InvalidOperationException(
            "Insufficient balance or source account not found.");

    await using var credit = new SqlCommand(
        "UPDATE dbo.Accounts SET Balance = Balance + @Amount WHERE Id = @ToId;",
        connection, transaction);

    credit.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 500m;
    credit.Parameters.Add("@ToId", SqlDbType.Int).Value = 2;

    int credited = await credit.ExecuteNonQueryAsync();

    if (credited != 1)
        throw new InvalidOperationException("Destination account not found.");

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

**ব্যাখ্যা:**

-   একই connection-এ transaction শুরু হয়েছে।
-   Debit ও credit command-এ একই transaction দেওয়া হয়েছে।
-   Debit সফল না হলে credit হবে না।
-   Credit ব্যর্থ হলে debit-ও rollback হবে।
-   সব সফল হলে commit হবে।

> Production banking system-এ concurrency, locking strategy, isolation,
> idempotency, audit trail এবং deadlock handling আলাদাভাবে design করতে
> হয়।

## Q56. What is Commit?

Transaction-এর changes সফলভাবে স্থায়ী করে।

## Q57. What is Rollback?

Commit-এর আগে transaction-এর changes বাতিল করে।

## Q58. What is Isolation Level?

একাধিক concurrent transaction একে অপরের uncommitted বা changing data
কতটা দেখতে পারবে তা নিয়ন্ত্রণ করে।

## Q59. Explain common SQL Server isolation levels.

  -----------------------------------------------------------------------
  Isolation                           ধারণা
  ----------------------------------- -----------------------------------
  Read Uncommitted                    Uncommitted data পড়তে পারে; dirty
                                      read সম্ভব

  Read Committed                      Uncommitted data পড়া আটকায়; SQL
                                      Server-এর সাধারণ default

  Repeatable Read                     একই row পুনরায় পড়লে committed value
                                      পরিবর্তন ঠেকায়

  Serializable                        Range-level protection দিয়ে phantom
                                      insert ঠেকায়; concurrency কমতে পারে

  Snapshot                            Versioned row পড়ে; database option
                                      প্রয়োজন
  -----------------------------------------------------------------------

`Read Committed Snapshot` database-level row-versioning configuration;
এটি `Snapshot` isolation level-এর সঙ্গে এক জিনিস নয়।

## Q60. What is Dirty Read?

এক transaction অন্য transaction-এর commit না হওয়া data পড়ে। পরে writer
rollback করলে reader এমন data পড়েছিল যা কখনো committed হয়নি।

## Q61. What is Non-repeatable Read?

এক transaction একই row দুইবার পড়ে; মাঝখানে অন্য transaction update ও
commit করায় দুইবার আলাদা value পায়।

## Q62. What is Phantom Read?

এক transaction একই condition দিয়ে query দুইবার চালায়; মাঝখানে অন্য
transaction matching row insert/delete করায় result set বদলে যায়।

## Q63. What is a Deadlock?

দুই বা তার বেশি transaction পরস্পরের lock release-এর জন্য অপেক্ষা করে cycle
তৈরি করলে deadlock হয়। SQL Server একটি transaction victim হিসেবে
rollback করে।

**Prevention:** একই order-এ resource access, ছোট transaction, উপযুক্ত
index, user interaction চলাকালে transaction open না রাখা এবং deadlock
graph বিশ্লেষণ।

## Q64. What is TransactionScope?

`TransactionScope` ambient transaction API। Scope-এর মধ্যে enlist করা
resource একই transaction context-এ অংশ নিতে পারে। একাধিক resource enlist
হলে distributed transaction প্রয়োজন হতে পারে; platform ও infrastructure
support যাচাই করতে হয়। একটি SQL connection-এর স্পষ্ট local transaction-এর
জন্য `SqlTransaction` অনেক সময় সহজ।

## Q65. Why must every command use the same transaction?

একই transaction-এর অংশ করতে হলে command-কে একই connection এবং
transaction object দিতে হবে। অন্য connection-এ command চালালে সেটি
স্বয়ংক্রিয়ভাবে local transaction-এর অংশ হয় না।

------------------------------------------------------------------------

# 8. Async, Cancellation and Resource Management

## Q66. Why use asynchronous ADO.NET methods?

Database I/O চলাকালে thread-কে অপ্রয়োজনীয়ভাবে block না রেখে ASP.NET
Core-এর scalability উন্নত করতে সাহায্য করে। Async query নিজে থেকে SQL
query-কে দ্রুত করে না।

## Q67. ExecuteReader vs ExecuteReaderAsync?

প্রথমটি synchronous; call return না হওয়া পর্যন্ত calling thread block হতে
পারে। দ্বিতীয়টি asynchronous I/O API, যা `await` করা যায়।

## Q68. What is CancellationToken?

Caller operation cancel করার signal দিতে পারে। ASP.NET Core-এ request-এর
`HttpContext.RequestAborted` token repository পর্যন্ত পাঠানো যায়।

``` csharp
public async Task<List<Employee>> GetAllAsync(
    CancellationToken cancellationToken)
{
    var employees = new List<Employee>();

    await using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync(cancellationToken);

    await using var command = new SqlCommand(
        "dbo.Employee_GetAll", connection);

    command.CommandType = CommandType.StoredProcedure;

    await using var reader =
        await command.ExecuteReaderAsync(cancellationToken);

    while (await reader.ReadAsync(cancellationToken))
    {
        employees.Add(new Employee
        {
            Id = reader.GetInt32(reader.GetOrdinal("Id")),
            Name = reader.GetString(reader.GetOrdinal("Name")),
            Email = reader.GetString(reader.GetOrdinal("Email")),
            Salary = reader.GetDecimal(reader.GetOrdinal("Salary"))
        });
    }

    return employees;
}
```

## Q69. What is IDisposable?

`IDisposable` deterministic cleanup pattern। `SqlConnection`,
`SqlCommand`, `SqlDataReader`-এর মতো object-এর resources release করতে
`using` ব্যবহার করা হয়।

## Q70. What is IAsyncDisposable?

Asynchronous cleanup support করে। `await using` দিয়ে async disposal করা
যায়।

## Q71. Why use await using?

যে object `IAsyncDisposable` implement করে, তার asynchronous disposal
নিশ্চিত করতে। `SqlConnection`, `SqlCommand`, `SqlDataReader`-এর async
disposal pattern ব্যবহার করা যায়।

## Q72. Should you use Task.Run around database calls?

সাধারণত প্রয়োজন নেই। ADO.NET-এর native async methods ব্যবহার করো।
`Task.Run()` দিয়ে synchronous database I/O thread-pool thread-এ সরানো
scalability বাড়ায় না।

------------------------------------------------------------------------

# 9. Advanced ADO.NET

## Q73. What is SqlBulkCopy?

`SqlBulkCopy` SQL Server table-এ বড় সংখ্যক row দ্রুত insert করার API।
Import, migration এবং ETL workload-এ কাজে লাগে।

``` csharp
await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

using var bulkCopy = new SqlBulkCopy(connection)
{
    DestinationTableName = "dbo.ImportEmployees",
    BatchSize = 5000,
    BulkCopyTimeout = 120
};

bulkCopy.ColumnMappings.Add("Name", "Name");
bulkCopy.ColumnMappings.Add("Email", "Email");

await bulkCopy.WriteToServerAsync(dataTable);
```

Target table, mapping, validation, duplicate policy এবং transaction
strategy আগে design করতে হবে।

## Q74. What is Table-Valued Parameter (TVP)?

TVP দিয়ে একাধিক row এক structured parameter হিসেবে stored procedure-এ
পাঠানো যায়।

**SQL type:**

``` sql
CREATE TYPE dbo.EmployeeInputType AS TABLE
(
    Name NVARCHAR(100),
    Email NVARCHAR(256)
);
```

**Stored procedure:**

``` sql
CREATE OR ALTER PROCEDURE dbo.Employee_BulkInsert
    @Employees dbo.EmployeeInputType READONLY
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO dbo.Employees (Name, Email, Salary)
    SELECT Name, Email, 0
    FROM @Employees;
END;
```

**C# parameter:**

``` csharp
var parameter = command.Parameters.AddWithValue("@Employees", dataTable);
parameter.SqlDbType = SqlDbType.Structured;
parameter.TypeName = "dbo.EmployeeInputType";
```

TVP SQL Server-এ `READONLY`; procedure-এর ভিতরে TVP data সরাসরি update
করা যায় না।

## Q75. What is Multiple Active Result Sets (MARS)?

MARS একই SQL Server connection-এ একাধিক active batch/result set support
করে। Connection string-এ `MultipleActiveResultSets=True` দিয়ে enable করা
যায়। এটি সব concurrency problem সমাধান করে না; concurrent independent
work-এর জন্য আলাদা connection অনেক সময় পরিষ্কার design।

## Q76. What is SqlBulkCopyOptions?

Bulk copy operation-এর behavior control করে। যেমন `KeepIdentity`,
`CheckConstraints`, `TableLock`, `FireTriggers`। Option নির্বাচন data
integrity ও performance requirement অনুযায়ী করতে হবে।

## Q77. What is GetSchema()?

Provider/database schema metadata যেমন tables, columns বা supported
schema collections সম্পর্কে তথ্য পেতে ব্যবহৃত হয়।

## Q78. What is Prepare()?

Command prepare করার অনুরোধ করে, যাতে provider/server execution প্রস্তুত
করতে পারে। সব workload-এ performance improvement নিশ্চিত নয়; benchmark
ছাড়া ব্যবহার করা উচিত নয়।

## Q79. What is a transaction savepoint?

SQL transaction-এর মধ্যে savepoint তৈরি করে, যাতে প্রয়োজনে transaction-এর
নির্দিষ্ট অংশ পর্যন্ত rollback করা যায়। Savepoint পুরো transaction commit করে
না।

------------------------------------------------------------------------

# 10. Exceptions and Reliability

## Q80. What is SqlException?

SQL Server provider-এর database-related exception। এতে error number,
state, class, server এবং একাধিক `SqlError` থাকতে পারে।

``` csharp
try
{
    await command.ExecuteNonQueryAsync();
}
catch (SqlException ex)
{
    foreach (SqlError error in ex.Errors)
    {
        logger.LogError(
            "SQL error {Number}: {Message}",
            error.Number,
            error.Message);
    }

    throw;
}
```

## Q81. SqlException vs Exception?

`Exception` হলো .NET-এর general base exception। `SqlException` হলো SQL
Server provider-specific exception, যাতে SQL Server error details থাকে।

## Q82. How do you handle duplicate key errors?

SQL Server-এ unique constraint violation সাধারণত error number 2627 বা
2601 হতে পারে। Exception number দেখে known conflict-কে application-level
conflict response-এ map করা যায়। সব exception-কে duplicate ধরে নেওয়া
যাবে না।

## Q83. Should you catch every exception in Repository?

শুধু log করার জন্য catch করে একই exception rethrow করা সবসময় দরকার নেই।
Handle করার নির্দিষ্ট কারণ থাকলে catch করো; otherwise exception উপরের
layer-এ propagate হতে দাও।

## Q84. How should retry be implemented?

Transient failure-এর ক্ষেত্রে bounded retry, exponential backoff এবং
jitter ব্যবহার করা যায়। কিন্তু non-idempotent operation blind retry করলে
duplicate payment বা insert হতে পারে। SQL error classification ও
operation idempotency জরুরি।

## Q85. What is a transient database error?

সাময়িক connectivity issue, failover বা throttling-এর মতো সমস্যা, যা কিছু
সময় পর retry করলে ঠিক হতে পারে। সব `SqlException` transient নয়।

## Q86. What should be logged for database troubleshooting?

-   Operation name ও correlation/trace ID
-   Duration ও timeout
-   Sanitized SQL operation identifier
-   SQL error number/state
-   Retry count
-   Relevant business key, যদি sensitive না হয়

Password, access token, full connection string বা unnecessary personal
data log করবে না।

------------------------------------------------------------------------

# 11. Performance Optimization

## Q87. How do you improve ADO.NET performance?

1.  প্রয়োজনীয় column select করা।
2.  Query plan ও index পরীক্ষা করা।
3.  Connection pooling ব্যবহার করতে দেওয়া।
4.  Connection scope ছোট রাখা।
5.  Streaming দরকার হলে DataReader ব্যবহার।
6.  বড় import-এ BulkCopy/TVP বিবেচনা।
7.  Async I/O ব্যবহার।
8.  N+1 query pattern এড়ানো।
9.  Network round trip কমানো।
10. Realistic workload দিয়ে benchmark করা।

## Q88. Is DataReader always faster than DataTable?

সব ক্ষেত্রে নয়। Sequential, read-only large result set-এর জন্য DataReader
সাধারণত কম memory ব্যবহার করে। In-memory sorting, editing বা repeated
access দরকার হলে DataTable বেশি সুবিধাজনক হতে পারে।

## Q89. What is Batch Processing?

একাধিক operation একবারে বা কম round trip-এ execute করার পদ্ধতি। BulkCopy,
TVP এবং set-based SQL batch processing-এর উদাহরণ।

## Q90. Why avoid SELECT \*?

অপ্রয়োজনীয় column transfer হয়, schema change-এ application impact বাড়ে
এবং index/covering query design কঠিন হতে পারে। প্রয়োজনীয় column explicit
select করো।

## Q91. What is N+1 query problem?

একটি main query-এর পর প্রতিটি row-এর জন্য আলাদা query চালানো হলে N+1 round
trip তৈরি হয়। JOIN, batch query বা TVP দিয়ে সমাধান করা যায়।

## Q92. Does async make SQL query faster?

না। Async মূলত calling thread block না রেখে I/O wait manage করে। Slow SQL
query optimize করতে index, query plan, statistics, blocking এবং data
volume পরীক্ষা করতে হবে।

------------------------------------------------------------------------

# 12. Security

## Q93. How do you prevent SQL Injection?

-   Parameterized query/stored procedure parameters।
-   Dynamic identifier-এর জন্য allow-list।
-   Least-privilege database account।
-   Input validation।
-   Stored procedure-এর ভিতরেও dynamic SQL parameterize করা।
-   Error response-এ internal SQL detail প্রকাশ না করা।

## Q94. Where should connection strings be stored?

Development-এ User Secrets, production-এ environment-based configuration
বা managed secret store ব্যবহার করো। Source control-এ password commit
করবে না।

## Q95. What is Least Privilege?

Application database identity-কে শুধু প্রয়োজনীয় database/schema/object
permission দেওয়া। Application login-কে অকারণে `db_owner` বা `sysadmin`
দেওয়া যাবে না।

## Q96. Is input validation a replacement for parameterized queries?

না। Validation business rule ও accepted input যাচাই করে।
Parameterization SQL code এবং data আলাদা রাখে। দুটোই প্রয়োজন।

## Q97. Should database exception details be returned to API clients?

না। Client-কে safe, meaningful error response দাও; internal exception
details server-side log-এ রাখো। এতে schema, query বা infrastructure
information leak কমে।

------------------------------------------------------------------------

# 13. ADO.NET in ASP.NET Core Clean Architecture

## Q98. Where should ADO.NET code live in Clean Architecture?

সাধারণত Infrastructure layer-এ। Application layer repository interface
বা abstraction define করতে পারে; Infrastructure layer
`Microsoft.Data.SqlClient` দিয়ে implementation দেয়।

``` text
MyApp
├── MyApp.Domain
├── MyApp.Application
│   └── Interfaces
│       └── IEmployeeRepository.cs
├── MyApp.Infrastructure
│   └── Persistence
│       └── EmployeeRepository.cs
└── MyApp.Api
    └── Controllers
```

## Q99. Should Controller directly create SqlConnection?

সাধারণত না। Controller HTTP concern handle করবে। Database access
repository/infrastructure service-এ রাখলে testing, maintenance এবং
separation of concerns উন্নত হয়।

## Q100. What is Repository Pattern?

Repository data-access logic-কে abstraction-এর আড়ালে রাখে। Application
layer storage implementation-এর details না জেনেও data operation করতে
পারে।

## Q101. What is Unit of Work?

একটি business operation-এর একাধিক repository/database change এক
transaction-এ coordinate করার pattern। ADO.NET-এ shared connection ও
`SqlTransaction`-এর মাধ্যমে implement করা যায়।

## Q102. Should SqlConnection be registered as Singleton?

না। `SqlConnection` instance singleton হিসেবে share করা উচিত নয়।
Connection factory singleton হতে পারে, যদি সেটি শুধু immutable
configuration ধরে এবং প্রতিটি operation-এ নতুন connection তৈরি করে।

## Q103. What is a Connection Factory?

একটি abstraction/class যা configured connection string ব্যবহার করে নতুন
`SqlConnection` তৈরি করে। এটি connection creation centralize করে;
connection lifetime repository/operation manage করে।

``` csharp
public interface ISqlConnectionFactory
{
    SqlConnection CreateConnection();
}

public sealed class SqlConnectionFactory : ISqlConnectionFactory
{
    private readonly string _connectionString;

    public SqlConnectionFactory(IConfiguration configuration)
    {
        _connectionString =
            configuration.GetConnectionString("Default")
            ?? throw new InvalidOperationException(
                "Default connection string is missing.");
    }

    public SqlConnection CreateConnection()
        => new SqlConnection(_connectionString);
}
```

DI registration:

``` csharp
builder.Services.AddSingleton<ISqlConnectionFactory, SqlConnectionFactory>();
```

Factory singleton হতে পারে কারণ এটি শুধু connection string ধরে; প্রতিটি
operation-এ নতুন connection তৈরি করে।

## Q104. How do you pass CancellationToken from API to repository?

Controller action-এ `CancellationToken cancellationToken` parameter নাও
এবং সেটি Application service ও repository method-এ pass করো। ASP.NET
Core request cancellation token bind করে দিতে পারে।

## Q105. Should business rules be placed inside stored procedures or C#?

দুটিরই উপযুক্ত ব্যবহার আছে। Referential integrity, unique constraints এবং
database-centric atomic operation database-এ থাকা উচিত। Application
workflow ও domain rule সাধারণত application/domain layer-এ রাখা ভালো। একই
rule দুই জায়গায় duplicate করলে inconsistency হতে পারে।

------------------------------------------------------------------------

# 14. ADO.NET vs Dapper vs EF Core

## Q106. ADO.NET vs Dapper?

  -----------------------------------------------------------------------
  ADO.NET                             Dapper
  ----------------------------------- -----------------------------------
  Low-level provider API              ADO.NET-এর ওপর micro-ORM

  Manual mapping                      Automatic object mapping

  বেশি boilerplate                    কম boilerplate

  SQL-এর ওপর পূর্ণ control              SQL-এর ওপর উচ্চ control

  Explicit resource management        Underlying connection lifecycle
                                      developer manage করে
  -----------------------------------------------------------------------

## Q107. ADO.NET vs Entity Framework Core?

  -----------------------------------------------------------------------
  ADO.NET                             EF Core
  ----------------------------------- -----------------------------------
  Direct SQL/provider API             Full ORM

  Manual mapping                      Entity mapping

  SQL control বেশি                    LINQ ও change tracking সুবিধা

  Boilerplate বেশি                    CRUD code কম

  Lightweight low-level access        Complex object graph ও domain
                                      model-এ সুবিধাজনক
  -----------------------------------------------------------------------

## Q108. When should you choose ADO.NET?

Stored procedure-heavy system, fine-grained SQL control, manual mapping
গ্রহণযোগ্য, performance-sensitive path বা existing ADO.NET codebase-এ।

## Q109. When should you choose Dapper?

SQL control রেখে mapping boilerplate কমাতে চাইলে Dapper উপযোগী।

## Q110. When should you choose EF Core?

LINQ, change tracking, migrations, relationship mapping এবং
domain-oriented CRUD workflow দরকার হলে EF Core উপযোগী।

------------------------------------------------------------------------

# 15. Real-world Scenario-based Questions

## Scenario 1: Bank transfer

**Question:** Account A থেকে Account B-তে টাকা পাঠাতে হলে কীভাবে data
consistency নিশ্চিত করবে?

**Answer:**

-   একই SQL connection-এ transaction শুরু।
-   Source account-এর balance পর্যাপ্ত কি না যাচাই।
-   Debit ও credit update একই transaction-এ।
-   দুই operation-এর affected row/status যাচাই।
-   সব সফল হলে commit; যেকোনো failure-এ rollback।
-   Concurrent transfer-এর জন্য suitable locking/isolation strategy।
-   API retry-এর জন্য idempotency key।
-   Audit record একই transaction-এ রাখা, যদি একই database boundary হয়।

## Scenario 2: Duplicate email registration

দুইটি request একই email দিয়ে একই সময়ে register করলে কী করবে?

-   Email normalize/validate।
-   Database-এ unique constraint রাখবে।
-   Application pre-check করা যায়, কিন্তু সেটি concurrency-safe guarantee
    নয়।
-   Unique violation 2601/2627 handle করে conflict response দেবে।

## Scenario 3: Large Excel/CSV import

100,000 row insert করতে হলে কী করবে?

-   Input validate ও normalize।
-   `DataTable` বা `IDataReader` source তৈরি।
-   `SqlBulkCopy` দিয়ে batch insert।
-   Staging table ব্যবহার করে validation/deduplication।
-   প্রয়োজন হলে transaction ও error-row report।
-   Memory usage ও batch size benchmark।

## Scenario 4: API request cancelled

User request cancel করলে running query কীভাবে stop করার চেষ্টা করবে?

`CancellationToken` Controller থেকে repository পর্যন্ত পাঠাবে এবং
`OpenAsync`, `ExecuteReaderAsync`, `ReadAsync`-এ ব্যবহার করবে।
Cancellation server-side operation বন্ধ করার চেষ্টা করে; database/server
state সবসময় তাৎক্ষণিকভাবে থামবে এমন guarantee নয়।

## Scenario 5: Connection pool exhausted

Production-এ "Timeout expired prior to obtaining a connection from the
pool" এলে কী পরীক্ষা করবে?

1.  সব connection/reader/command dispose হচ্ছে কি না।
2.  Long-running query বা blocking আছে কি না।
3.  Connection open রেখে external API call বা user interaction হচ্ছে কি
    না।
4.  Traffic ও concurrency spike।
5.  আলাদা connection string-এর কারণে pool fragmentation।
6.  Pool size বাড়ানোর আগে root cause।

## Scenario 6: Stored procedure slow

ADO.NET command timeout হচ্ছে। কী করবে?

-   SQL Server execution plan দেখবে।
-   Missing/inefficient index, scan, parameter sniffing পরীক্ষা।
-   Blocking/deadlock wait পরীক্ষা।
-   Actual parameter দিয়ে query reproduce।
-   Result set ও transferred column কমানো।
-   Timeout বাড়ানো কেবল justified হলে।

## Scenario 7: Multiple repositories in one business operation

Order create এবং stock decrease একসঙ্গে সফল করতে হবে। কীভাবে করবে?

একই connection ও `SqlTransaction` উভয় repository operation-এ coordinate
করতে হবে। Transaction ownership একটি application service/unit-of-work
boundary-তে স্পষ্ট রাখতে হবে। দুইটি independent connection হলে local
transaction একসঙ্গে তাদের cover করবে না।

------------------------------------------------------------------------

# 16. Interview Quick Revision --- 60 Questions

## Beginner

1.  What is ADO.NET?
2.  What is a Data Provider?
3.  What is Microsoft.Data.SqlClient?
4.  What is SqlConnection?
5.  What is SqlCommand?
6.  What is SqlDataReader?
7.  What is DataTable?
8.  What is DataSet?
9.  What is SqlDataAdapter?
10. What is Connected architecture?
11. What is Disconnected architecture?
12. What is a connection string?
13. What is CommandType?
14. What is a stored procedure?
15. What is a parameterized query?
16. What is ExecuteReader()?
17. What is ExecuteScalar()?
18. What is ExecuteNonQuery()?
19. What is DBNull.Value?
20. What is IDisposable?

## Intermediate

21. Explain Connection Pooling.
22. How does connection pool exhaustion happen?
23. Connection Timeout vs CommandTimeout?
24. DataReader vs DataAdapter?
25. DataTable vs DataSet?
26. What is NextResult()?
27. What is GetOrdinal()?
28. How do you handle SQL NULL?
29. Explain Input and Output parameters.
30. How do you pass NULL to a stored procedure?
31. How do you prevent SQL Injection?
32. What is SqlTransaction?
33. Explain ACID properties.
34. What is an isolation level?
35. What is a deadlock?
36. What is CancellationToken?
37. Why use async database methods?
38. What is SqlException?
39. What is SqlBulkCopy?
40. What is TVP?

## Advanced

41. Explain parameter sniffing.
42. Explain MARS and its limitations.
43. What is SequentialAccess?
44. What is TransactionScope?
45. Local vs distributed transaction?
46. How do you implement retry safely?
47. How do you prevent duplicate concurrent registration?
48. How do you coordinate multiple repositories in one transaction?
49. How do you design ADO.NET in Clean Architecture?
50. How do you troubleshoot connection pool exhaustion?
51. How do you optimize a slow stored procedure?
52. How do you process millions of rows efficiently?
53. How do you handle optimistic concurrency?
54. How do you implement idempotent payment processing?
55. How do you secure database credentials?
56. How do you test repository implementations?
57. How do you avoid N+1 queries?
58. How do you choose between ADO.NET, Dapper and EF Core?
59. What are the limitations of ExecuteNonQuery() row counts?
60. How do you handle multiple result sets?

------------------------------------------------------------------------

# 17. Practical Coding Exercises

## Exercise 1 --- Employee CRUD

Implement:

-   `CreateAsync(Employee employee, CancellationToken ct)`
-   `GetByIdAsync(int id, CancellationToken ct)`
-   `GetAllAsync(CancellationToken ct)`
-   `UpdateAsync(Employee employee, CancellationToken ct)`
-   `DeleteAsync(int id, CancellationToken ct)`

**Requirements:** Stored procedures, typed parameters, async methods,
reader mapping, cancellation token, safe exception handling.

## Exercise 2 --- Stored procedure Output Parameter

একটি procedure তৈরি করো যা employee insert করে generated identity output
parameter দিয়ে ফেরত দেয়।

## Exercise 3 --- Transaction

Order creation implement করো:

1.  Insert order header.
2.  Insert order items.
3.  Decrease stock.
4.  সব সফল হলে commit।

## Exercise 4 --- TVP

একটি structured parameter-এ employee list পাঠিয়ে stored procedure দিয়ে
insert করো।

## Exercise 5 --- BulkCopy

কমপক্ষে 50,000 row staging table-এ import করো। Duration measure করে
row-by-row insert-এর সঙ্গে compare করো।

## Exercise 6 --- NULL Handling

Nullable database column নিরাপদে C# nullable property-তে map করো।

## Exercise 7 --- Multiple Result Sets

একটি procedure থেকে employee ও department আলাদা result set হিসেবে ফেরত
দাও। `NextResultAsync()` দিয়ে পড়ো।

## Exercise 8 --- Repository Pattern

Application layer-এ `IEmployeeRepository` এবং Infrastructure layer-এ
ADO.NET implementation তৈরি করো।

## Exercise 9 --- Cancellation

ASP.NET Core endpoint থেকে repository query-তে request cancellation pass
করো।

## Exercise 10 --- Duplicate-safe Registration

Database unique constraint ব্যবহার করে duplicate-key SQL error-কে
conflict response-এ map করো।

------------------------------------------------------------------------

# 18. Common Mistakes and Best Practices

  -------------------------------------------------------------------------
  Mistake                 কেন সমস্যা               Better approach
  ----------------------- ----------------------- -------------------------
  SQL string              SQL Injection           Typed parameters
  concatenation                                   

  Connection dispose না   Pool exhaustion         `await using`
  করা                                             

  Shared static           Concurrency/state issue Per-operation connection
  connection                                      

  `SELECT *`              Unnecessary transfer    Explicit columns

  সব exception swallow    Failure hidden          Log/propagate/translate
  করা                                             appropriately

  Business operation      Inconsistent data       Correct transaction
  transaction ছাড়া চালানো                         boundary

  Transaction দীর্ঘ সময়    Lock contention         Short transaction
  open রাখা                                       

  সব timeout বাড়িয়ে দেওয়া Root cause ঢাকা পড়ে     Query/blocking diagnose

  Blind retry             Duplicate side effects  Idempotency +
                                                  transient-only retry

  Stored procedure        Injection risk          Parameterized dynamic SQL
  dynamic SQL concatenate                         

  `Task.Run()` around     Thread-pool waste       Native async API
  sync SQL                                        

  সব data DataTable-এ     High memory use         Reader streaming when
  load                                            suitable
  -------------------------------------------------------------------------

## Production best-practices checklist

-   [ ] `Microsoft.Data.SqlClient` ব্যবহার।
-   [ ] SQL parameter-এর type ও size explicit করা।
-   [ ] Connection/command/reader dispose করা।
-   [ ] Request cancellation propagate করা।
-   [ ] Connection string secret store-এ রাখা।
-   [ ] Least-privilege DB identity ব্যবহার।
-   [ ] Transaction boundary business operation অনুযায়ী রাখা।
-   [ ] Transaction ছোট রাখা।
-   [ ] Query plan, index ও blocking পরীক্ষা।
-   [ ] Large import-এ TVP/SqlBulkCopy বিবেচনা।
-   [ ] SQL exception details client-কে প্রকাশ না করা।
-   [ ] Retry policy idempotency-aware করা।
-   [ ] Integration test SQL Server-compatible environment-এ চালানো।
-   [ ] Logging-এ secret ও sensitive data বাদ দেওয়া।

------------------------------------------------------------------------

# 19. Final Interview Checklist

-   [ ] ADO.NET architecture নিজের ভাষায় ব্যাখ্যা করতে পারি।
-   [ ] Connection, Command, Reader-এর lifecycle বুঝি।
-   [ ] Connection Pooling ব্যাখ্যা করতে পারি।
-   [ ] তিনটি Execute method-এর পার্থক্য জানি।
-   [ ] Stored procedure parameter লিখতে পারি।
-   [ ] SQL NULL handle করতে পারি।
-   [ ] SQL Injection-এর vulnerable ও safe code চিনি।
-   [ ] DataReader দিয়ে model mapping করতে পারি।
-   [ ] CRUD repository implement করতে পারি।
-   [ ] Transaction দিয়ে debit/credit বা order/stock operation করতে
    পারি।
-   [ ] ACID ও isolation level ব্যাখ্যা করতে পারি।
-   [ ] Deadlock ও pool exhaustion troubleshoot করতে পারি।
-   [ ] Async ও CancellationToken ব্যবহার করতে পারি।
-   [ ] TVP ও SqlBulkCopy-এর use case বুঝি।
-   [ ] ASP.NET Core Clean Architecture-এ ADO.NET integrate করতে পারি।
-   [ ] ADO.NET, Dapper ও EF Core-এর trade-off ব্যাখ্যা করতে পারি।

------------------------------------------------------------------------

## Important Technical Notes

1.  `ExecuteNonQuery()`-এর return value সব stored procedure-এ business
    success-এর নির্ভরযোগ্য indicator নয়। Explicit output/status design
    করো।
2.  `SqlDataAdapter.Fill()` synchronous API; async web request path-এ
    reader-based async access বিবেচনা করো।
3.  `SqlConnection` object share করো না; pooling-এর কারণে per-operation
    connection তৈরি/dispose করা স্বাভাবিক।
4.  Stored procedure থাকলেই SQL Injection অসম্ভব হয় না; dynamic SQL
    parameterize করতে হবে।
5.  Async I/O thread scalability উন্নত করে; query execution plan optimize
    করার বিকল্প নয়।
6.  `TransactionScope` distributed transaction trigger করতে পারে;
    deployment environment support যাচাই করো।
7.  Isolation level ও locking behavior SQL Server configuration এবং
    workload অনুযায়ী পরীক্ষা করো।

