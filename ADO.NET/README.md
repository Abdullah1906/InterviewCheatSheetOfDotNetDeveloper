# ADO.NET Interview Questions & Answers

**SQL Server | Microsoft.Data.SqlClient | Stored Procedure | Transaction**

## 1. Fundamentals

### Q1. ADO.NET কী?
ADO.NET হলো .NET-এর data access technology। SQL Server-এর জন্য `Microsoft.Data.SqlClient` provider-এর `SqlConnection`, `SqlCommand`, `SqlDataReader`, `SqlTransaction` ইত্যাদি ব্যবহার করা হয়।

### Q2. `SqlConnection` কী?
SQL Server database connection তৈরি ও পরিচালনা করে।

```csharp
using Microsoft.Data.SqlClient;

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
```

### Q3. `SqlCommand` কী?
SQL statement বা stored procedure execute করে।

### Q4. `SqlDataReader` কী?
Forward-only, read-only data stream। বড় result set sequentially পড়ার জন্য উপযোগী।

```csharp
await using var command = new SqlCommand(
    "SELECT Id, Name FROM dbo.Employees", connection);

await using var reader = await command.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    int id = reader.GetInt32(0);
    string name = reader.GetString(1);
}
```

### Q5. `ExecuteReader`, `ExecuteScalar`, `ExecuteNonQuery`?
| Method | কাজ |
|---|---|
| ExecuteReader | Row/result set পড়ে |
| ExecuteScalar | প্রথম row-এর প্রথম column ফেরত দেয় |
| ExecuteNonQuery | সাধারণত INSERT/UPDATE/DELETE-এর affected row count দেয় |

### Q6. `SqlDataAdapter` ও `DataTable` কী?
DataAdapter database থেকে disconnected `DataSet`/`DataTable`-এ data fill করতে পারে। `DataTable` হলো memory-তে tabular data structure।

### Q7. Connected ও Disconnected model?
`SqlDataReader` connection খোলা রেখে stream পড়ে। `DataTable`/`DataSet`-এ data load করলে পরে connection ছাড়াই data নিয়ে কাজ করা যায়।

## 2. Stored Procedure ও Parameters

### Q8. Stored Procedure execute করবে কীভাবে?

```csharp
await using var command = new SqlCommand("dbo.sp_GetEmployeeById", connection);
command.CommandType = CommandType.StoredProcedure;
command.Parameters.Add("@Id", SqlDbType.Int).Value = 10;

await using var reader = await command.ExecuteReaderAsync();
```

### Q9. Parameter কেন ব্যবহার করতে হবে?
SQL Injection প্রতিরোধ, type-safe value passing এবং query plan reuse-এ সহায়তার জন্য। User input string concatenate করে SQL বানানো যাবে না।

### Q10. Output Parameter কী?

```csharp
var output = command.Parameters.Add("@Total", SqlDbType.Int);
output.Direction = ParameterDirection.Output;

await command.ExecuteNonQueryAsync();
int total = (int)output.Value;
```

Stored procedure-এ parameter-টি `OUTPUT` হিসেবে declare করতে হবে।

### Q11. SQL Injection কীভাবে প্রতিরোধ করবে?
Parameterized query, stored procedure parameters, least-privilege database account এবং input validation ব্যবহার করবে। Stored procedure-এ dynamic SQL থাকলেও সেটি parameterized হতে হবে।

## 3. Transaction ও Resource Management

### Q12. ADO.NET transaction কীভাবে করবে?

```csharp
await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

await using var transaction = await connection.BeginTransactionAsync();

try
{
    await using var command = new SqlCommand(
        "UPDATE dbo.Accounts SET Balance = Balance - @Amount WHERE Id = @Id",
        connection,
        (SqlTransaction)transaction);

    command.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 100m;
    command.Parameters.Add("@Id", SqlDbType.Int).Value = 1;

    await command.ExecuteNonQueryAsync();
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

বাস্তবে debit/credit-এর মতো business operation-এ উভয় update একই transaction-এ করতে হবে এবং affected row/available balance যাচাই করতে হবে।

### Q13. Connection Pooling কী?
ADO.NET physical connection reuse করে connection creation overhead কমায়। Connection দ্রুত open/close করা স্বাভাবিক pattern; `using` দিয়ে dispose করা উচিত।

### Q14. `IDisposable` কেন?
Connection, command, reader-এর মতো resource deterministic cleanup করার জন্য। `using`/`await using` ব্যবহার করো।

### Q15. `CommandTimeout` কী?
Command কতক্ষণ execute করার সুযোগ পাবে তা নির্ধারণ করে। Timeout বাড়ানো সবসময় সমাধান নয়—query performance ও blocking-ও পরীক্ষা করতে হবে।

## 4. Advanced ADO.NET

### Q16. `SqlBulkCopy` কী?
বড় পরিমাণ row SQL Server table-এ দ্রুত insert করার API।

```csharp
using var bulkCopy = new SqlBulkCopy(connection);
bulkCopy.DestinationTableName = "dbo.ImportEmployees";
await bulkCopy.WriteToServerAsync(dataTable);
```

### Q17. TVP কী?
Table-Valued Parameter দিয়ে একাধিক row structured parameter হিসেবে stored procedure-এ পাঠানো যায়। SQL Server-এ user-defined table type তৈরি করতে হয় এবং parameter `READONLY` হয়।

### Q18. Multiple Result Set পড়বে কীভাবে?
`SqlDataReader.NextResultAsync()` দিয়ে পরবর্তী result set-এ যাওয়া যায়।

### Q19. ADO.NET বনাম Dapper?
ADO.NET low-level database API। Dapper ADO.NET-এর ওপর lightweight mapping layer; SQL control বেশি থাকে, mapping boilerplate কমে।

### Q20. Repository Pattern-এ ADO.NET কীভাবে ব্যবহার করবে?
Repository interface Application layer-এ থাকবে; Infrastructure implementation-এ connection factory, parameterized command, mapping এবং transaction handling থাকবে। Controller থেকে সরাসরি SQL চালানো এড়িয়ে চলবে।

## Best Practices

- [ ] `Microsoft.Data.SqlClient` ব্যবহার
- [ ] Parameterized query
- [ ] Connection/Command/Reader dispose
- [ ] Async API ব্যবহার
- [ ] Transaction boundary স্পষ্ট রাখা
- [ ] `SELECT *` এড়িয়ে প্রয়োজনীয় column নেওয়া
- [ ] Timeout ও cancellation token বিবেচনা
- [ ] Exception log করে rethrow/handle করা
- [ ] Connection string secret হিসেবে রাখা
- [ ] Large import-এ `SqlBulkCopy`/TVP বিবেচনা
