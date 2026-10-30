# C# Interview Questions & Answers

**Basic → Intermediate → Advanced**

## 1. C# ও .NET Fundamentals

### Q1. C# কী?
C# হলো Microsoft-এর তৈরি একটি আধুনিক, strongly typed, object-oriented programming language। .NET platform ব্যবহার করে C# দিয়ে Web API, MVC, desktop, cloud এবং enterprise application তৈরি করা যায়।

### Q2. C# ও .NET-এর পার্থক্য কী?
- **C#** হলো programming language।
- **.NET** হলো application build ও run করার platform; এতে runtime, libraries এবং SDK থাকে।

### Q3. CLR কী?
CLR (Common Language Runtime) হলো .NET-এর execution environment। এটি JIT compilation, garbage collection, exception handling, type safety এবং thread management-এর মতো কাজ করে।

### Q4. Value Type ও Reference Type-এর পার্থক্য কী?
- Value type assignment-এ সাধারণত value copy হয়। উদাহরণ: `int`, `bool`, `struct`, `enum`।
- Reference type assignment-এ object-এর reference copy হয়। উদাহরণ: `class`, `array`, `string`, `delegate`।
- Value type সবসময় stack-এ থাকে—এটি সঠিক নয়; storage কোথায় হবে তা context-এর ওপর নির্ভর করে।

```csharp
int a = 10;
int b = a;
b = 20;
Console.WriteLine(a); // 10
```

### Q5. `var`, `dynamic`, `object`-এর পার্থক্য কী?
- `var`: compile-time type inference; type একবার নির্ধারিত হলে বদলায় না।
- `dynamic`: member access-এর validation runtime-এ হয়।
- `object`: সব .NET type-এর base type; নির্দিষ্ট member ব্যবহার করতে cast বা pattern matching লাগে।

### Q6. `const` ও `readonly`-এর পার্থক্য কী?
- `const` compile-time constant; declaration-এর সময় value দিতে হয়।
- `readonly` field declaration বা constructor-এ assign করা যায়।
- `const` implicitly static; `readonly` instance বা static—দুই ধরনের হতে পারে।

### Q7. Boxing ও Unboxing কী?
Value type-কে `object` বা interface reference-এ রূপান্তর করলে boxing হয়; object থেকে value type-এ ফিরলে unboxing হয়।

```csharp
int number = 42;
object boxed = number;       // Boxing
int result = (int)boxed;     // Unboxing
```

### Q8. `string` immutable কেন?
একটি string তৈরি হওয়ার পর তার content পরিবর্তন করা যায় না। পরিবর্তনের মতো operation নতুন string তৈরি করে। অনেকবার string জোড়া লাগাতে `StringBuilder` কার্যকর।

```csharp
var builder = new System.Text.StringBuilder();
builder.Append("Hello");
builder.Append(" .NET");
Console.WriteLine(builder);
```

## 2. OOP

### Q9. OOP-এর চারটি মূলনীতি কী?
1. **Encapsulation:** data ও behavior একত্রে রাখা এবং controlled access দেওয়া।
2. **Inheritance:** existing class-এর behavior extend করা।
3. **Polymorphism:** একই contract-এর ভিন্ন implementation।
4. **Abstraction:** implementation detail আড়াল করে প্রয়োজনীয় contract প্রকাশ করা।

### Q10. Interface ও Abstract Class-এর পার্থক্য কী?
- Abstract class shared state, constructor এবং implemented method রাখতে পারে।
- Interface contract নির্ধারণ করে; আধুনিক C#-এ default implementation ও static members-ও থাকতে পারে।
- একটি class একটি base class inherit করতে পারে, কিন্তু একাধিক interface implement করতে পারে।
- কোনটি ব্যবহার হবে তা design requirement-এর ওপর নির্ভর করে।

### Q11. Overloading ও Overriding কী?
- **Overloading:** একই method name, কিন্তু parameter list ভিন্ন।
- **Overriding:** derived class-এ `virtual`/`abstract` member-এর নতুন implementation।

### Q12. `virtual`, `override`, `new` কী?
- `virtual`: derived class-কে override করার সুযোগ দেয়।
- `override`: inherited virtual/abstract member-এর implementation বদলায়।
- `new`: base member-কে hide করে; polymorphic override নয়।

### Q13. Composition বনাম Inheritance?
Inheritance হলো “is-a”; composition হলো “has-a” সম্পর্ক। যেমন `Dog is an Animal`, `Car has an Engine`। Flexible design-এর ক্ষেত্রে composition অনেক সময় ভালো পছন্দ।

## 3. Collections, Generics ও LINQ

### Q14. Array, List, Dictionary-এর পার্থক্য কী?
- `Array`: fixed length।
- `List<T>`: dynamic-size collection।
- `Dictionary<TKey,TValue>`: key দিয়ে value lookup; key unique হতে হয়।

### Q15. Generics কেন ব্যবহার করা হয়?
Generics reusable ও type-safe code তৈরি করে এবং অনেক ক্ষেত্রে boxing/casting কমায়।

```csharp
public class Repository<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);
    public IReadOnlyList<T> GetAll() => _items;
}
```

### Q16. LINQ কী?
LINQ (Language Integrated Query) collection, database provider এবং অন্যান্য data source query করার syntax ও operator দেয়।

```csharp
var activeNames = employees
    .Where(e => e.IsActive)
    .OrderBy(e => e.Name)
    .Select(e => e.Name)
    .ToList();
```

### Q17. `IEnumerable<T>` ও `IQueryable<T>`-এর পার্থক্য কী?
- `IEnumerable<T>` সাধারণত in-memory iteration-এর জন্য।
- `IQueryable<T>` expression tree তৈরি করে; provider সেটিকে SQL-এর মতো query-তে translate করতে পারে।
- `IQueryable`-এ সব .NET method SQL-এ translate হবে—এমন নিশ্চয়তা নেই।

### Q18. `FirstOrDefault()` ও `SingleOrDefault()`?
- `FirstOrDefault()` প্রথম match দেয়; না পেলে default।
- `SingleOrDefault()` সর্বোচ্চ একটি match প্রত্যাশা করে; একাধিক match হলে exception দেয়।

## 4. Async, Memory ও Exception

### Q19. `async` ও `await` কী?
Asynchronous operation সহজভাবে লেখার syntax। I/O-bound কাজের অপেক্ষায় thread অকারণে block না করাই এর বড় সুবিধা।

```csharp
public async Task<string> LoadAsync(HttpClient client)
{
    return await client.GetStringAsync("https://example.com");
}
```

### Q20. `Task` ও `Thread`-এর পার্থক্য কী?
`Thread` হলো execution thread; `Task` হলো asynchronous operation-এর representation। সাধারণ application code-এ `Task`/`Task<T>` ব্যবহার করা হয়; প্রতিটি Task-এর জন্য আলাদা Thread তৈরি হয় না।

### Q21. `Task.WhenAll()` কী?
একাধিক task একসঙ্গে শুরু করে সবগুলো শেষ হওয়া পর্যন্ত asynchronously অপেক্ষা করে।

```csharp
var usersTask = GetUsersAsync();
var ordersTask = GetOrdersAsync();

await Task.WhenAll(usersTask, ordersTask);
```

### Q22. `CancellationToken` কেন লাগে?
Caller যেন চলমান asynchronous কাজ cancel করার request দিতে পারে। Cancellation cooperative—operation-কে token পরীক্ষা বা pass করতে হয়।

### Q23. Garbage Collection কী?
.NET-এর managed memory থেকে unreachable object-এর memory reclaim করে। GC-তে Generation 0, 1, 2 এবং Large Object Heap রয়েছে।

### Q24. `IDisposable` ও `using` কী?
`IDisposable` resource cleanup-এর contract। `using` scope শেষ হলে `Dispose()` নিশ্চিত করে—যেমন database connection, stream।

```csharp
using var connection = new Microsoft.Data.SqlClient.SqlConnection(connectionString);
```

### Q25. `throw;` ও `throw ex;`-এর পার্থক্য?
`throw;` original stack trace বজায় রাখে। `throw ex;` stack trace reset করতে পারে। Catch block থেকে একই exception পুনরায় ছুড়তে `throw;` ব্যবহার করো।

## 5. Advanced C#

### Q26. Delegate কী?
Delegate হলো type-safe method reference। `Func`, `Action`, `Predicate` হলো built-in delegate types।

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(2, 3));
```

### Q27. Event কী?
Event delegate-ভিত্তিক notification mechanism। Publisher কোনো ঘটনা ঘটলে subscriber-দের notify করে।

### Q28. Record কী?
Record data-centric type; value-based equality এবং concise syntax দেয়।

```csharp
public record EmployeeDto(int Id, string Name);
```

### Q29. `ref`, `out`, `in` কী?
- `ref`: caller-এর initialized variable reference হিসেবে pass।
- `out`: method-এর মধ্যে value assign করতে হয়।
- `in`: readonly reference হিসেবে pass।

### Q30. `lock` কেন ব্যবহার হয়?
একই process-এর একাধিক thread যেন একই critical section একই সময়ে execute না করে, তা নিশ্চিত করে। Async code-এ `lock`-এর ভেতরে `await` করা যায় না; প্রয়োজনে `SemaphoreSlim` বিবেচনা করো।

## দ্রুত Revision Checklist

- [ ] Value type বনাম reference type
- [ ] OOP চারটি principle
- [ ] Interface বনাম abstract class
- [ ] `IEnumerable` বনাম `IQueryable`
- [ ] LINQ deferred execution
- [ ] async/await ও cancellation
- [ ] Task বনাম Thread
- [ ] Exception handling ও stack trace
- [ ] Delegate, Event, Generics
- [ ] IDisposable ও Garbage Collection
