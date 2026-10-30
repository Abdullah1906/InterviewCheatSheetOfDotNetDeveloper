# ASP.NET Core Interview Questions & Answers

**.NET 8+ | MVC | Web API | Middleware | DI | Security**

## 1. .NET ও ASP.NET Core

### Q1. .NET কী?
.NET একটি cross-platform, open-source development platform। এতে runtime, SDK, base libraries এবং application framework রয়েছে।

### Q2. ASP.NET Core কী?
ASP.NET Core হলো .NET-এর web framework। এটি REST API, MVC, Razor Pages, SignalR এবং web application তৈরিতে ব্যবহৃত হয়।

### Q3. .NET Framework ও আধুনিক .NET-এর পার্থক্য?
.NET Framework মূলত Windows-কেন্দ্রিক legacy platform। আধুনিক .NET cross-platform এবং নতুন application development-এর জন্য ব্যবহৃত হয়।

### Q4. CLR কী?
Common Language Runtime managed .NET code execute করে এবং memory management, JIT, exception handling ও type safety পরিচালনা করে।

### Q5. Kestrel কী?
Kestrel হলো ASP.NET Core-এর cross-platform web server। Production-এ এটি সরাসরি বা reverse proxy (যেমন IIS/Nginx)-এর পেছনে চলতে পারে।

### Q6. `Program.cs`-এর কাজ কী?
Application-এর entry point। এখানে service register, configuration এবং middleware pipeline configure করা হয়।

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.MapControllers();

app.Run();
```

## 2. Web API ও HTTP

### Q7. REST API কী?
REST হলো resource-oriented architectural style। HTTP method ও resource URL ব্যবহার করে client-server communication করা হয়।

### Q8. HTTP method-গুলোর ব্যবহার কী?
| Method | সাধারণ ব্যবহার |
|---|---|
| GET | Resource পড়া |
| POST | Resource তৈরি বা operation শুরু |
| PUT | Resource সম্পূর্ণ replace |
| PATCH | Resource আংশিক update |
| DELETE | Resource delete |

### Q9. গুরুত্বপূর্ণ HTTP status code কী?
- `200 OK`: সফল request
- `201 Created`: resource তৈরি
- `204 No Content`: সফল, response body নেই
- `400 Bad Request`: invalid request
- `401 Unauthorized`: authentication অনুপস্থিত/অকার্যকর
- `403 Forbidden`: পরিচয় জানা, কিন্তু permission নেই
- `404 Not Found`: resource নেই
- `409 Conflict`: state conflict
- `429 Too Many Requests`: rate limit অতিক্রম
- `500 Internal Server Error`: server-side unexpected error

### Q10. 401 ও 403-এর পার্থক্য কী?
401 authentication সমস্যা; 403 authenticated user-এর permission নেই।

### Q11. DTO কেন ব্যবহার করা হয়?
API contract ও database entity আলাদা রাখতে, sensitive field expose না করতে এবং request/response shape নিয়ন্ত্রণ করতে DTO ব্যবহার করা হয়।

```csharp
public sealed class CreateEmployeeRequest
{
    public required string Name { get; init; }
    public int DepartmentId { get; init; }
}
```

### Q12. `IActionResult` ও `ActionResult<T>`?
`IActionResult` বিভিন্ন ধরনের HTTP result দিতে পারে। `ActionResult<T>` typed response-এর পাশাপাশি `NotFound()`, `BadRequest()` ইত্যাদি result দিতে পারে।

```csharp
[HttpGet("{id:int}")]
public ActionResult<EmployeeDto> Get(int id)
{
    var employee = FindEmployee(id);
    if (employee is null) return NotFound();
    return Ok(employee);
}
```

### Q13. Model Binding কী?
Route, query string, header, form বা body থেকে request data action parameter/model-এ map করাকে Model Binding বলে।

### Q14. Model Validation কী?
Data Annotation বা custom validator দিয়ে request data-এর নিয়ম যাচাই করা। `[ApiController]` ব্যবহার করলে invalid model-এর জন্য automatic 400 response পাওয়া যায়।

## 3. Dependency Injection

### Q15. Dependency Injection কী?
কোনো class নিজের dependency তৈরি না করে বাইরে থেকে গ্রহণ করে। এতে loose coupling, testability ও maintainability বাড়ে।

### Q16. DI lifetime কী কী?
| Lifetime | আচরণ |
|---|---|
| Transient | প্রতিবার resolve-এ নতুন instance |
| Scoped | প্রতি request scope-এ এক instance |
| Singleton | application container lifetime-এ এক instance |

### Q17. Singleton থেকে Scoped service inject করা ঝুঁকিপূর্ণ কেন?
Singleton দীর্ঘ সময় বেঁচে থাকে; Scoped dependency-কে capture করলে request-specific state ভুলভাবে reuse হতে পারে। এটি captive dependency সমস্যা।

### Q18. Interface-based DI-এর উদাহরণ?

```csharp
builder.Services.AddScoped<IEmployeeService, EmployeeService>();
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();
```

## 4. Middleware ও Request Pipeline

### Q19. Middleware কী?
HTTP request ও response pipeline-এর component। এটি request process করতে, পরের middleware-এ পাঠাতে বা pipeline short-circuit করতে পারে।

### Q20. Middleware order গুরুত্বপূর্ণ কেন?
Middleware-এর order execution ও security-তে প্রভাব ফেলে। উদাহরণ:

```csharp
app.UseExceptionHandler();
app.UseHttpsRedirection();
app.UseRouting();
app.UseCors("Frontend");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

Authentication অবশ্যই Authorization-এর আগে চলবে।

### Q21. Custom Middleware কীভাবে তৈরি করবে?

```csharp
public sealed class RequestLogMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLogMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context, ILogger<RequestLogMiddleware> logger)
    {
        logger.LogInformation("Request: {Path}", context.Request.Path);
        await _next(context);
    }
}
```

Registration: `app.UseMiddleware<RequestLogMiddleware>();`

## 5. Configuration, CORS ও API Documentation

### Q22. `appsettings.json` কী?
Application configuration রাখে। Password, signing key বা production secret source control-এ রাখা উচিত নয়; Secret Manager বা secure environment configuration ব্যবহার করো।

### Q23. `IConfiguration` ও Options pattern কী?
`IConfiguration` configuration value পড়তে দেয়। Options pattern strongly typed configuration class-এ section bind করে।

### Q24. CORS কী?
Browser-এর cross-origin request কোন origin থেকে অনুমোদিত হবে তা নিয়ন্ত্রণ করে। CORS authentication বা API security-এর বিকল্প নয়।

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("Frontend", policy =>
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod());
});
```

### Q25. Swagger/OpenAPI কী?
OpenAPI API contract বর্ণনার specification। Swagger UI-সহ tools API documentation দেখা ও test করতে সাহায্য করে।

## 6. Authentication ও Authorization

### Q26. Authentication বনাম Authorization?
Authentication user কে তা যাচাই করে। Authorization user কী করতে পারবে তা নির্ধারণ করে।

### Q27. JWT কী?
JSON Web Token হলো claims বহনকারী token format। Signed JWT-এর structure: Header, Payload, Signature। Payload সাধারণত encoded—encrypted নয়; তাই secret data রাখা যাবে না।

### Q28. Access Token ও Refresh Token?
Access token API access-এর জন্য স্বল্প মেয়াদি credential। Refresh token নতুন access token পাওয়ার জন্য ব্যবহৃত হয়। Refresh token secure storage, rotation, expiry ও revocation দরকার।

### Q29. Role ও Policy-based authorization?

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("{id:int}")]
public IActionResult Delete(int id) => NoContent();
```

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanManageEmployees",
        policy => policy.RequireClaim("permission", "employee.manage"));
});
```

### Q30. CORS কি JWT-কে নিরাপদ করে?
না। CORS browser policy; token validation, authorization, HTTPS, XSS/CSRF protection আলাদা security control।

## 7. Error Handling, Logging ও Performance

### Q31. Global Exception Handling কেন?
এক জায়গায় unexpected exception handle করে consistent error response দেওয়া এবং internal details client-কে না দেখানোর জন্য।

### Q32. `ILogger<T>` কী?
ASP.NET Core logging abstraction। Structured logging ব্যবহার করো এবং password/token-এর মতো sensitive data log করো না।

### Q33. API performance কীভাবে উন্নত করবে?
- Async I/O
- Database query ও index optimize
- Pagination
- Caching
- Response payload ছোট রাখা
- N+1 query এড়ানো
- Profiling ও monitoring

### Q34. Rate Limiting কী?
নির্দিষ্ট সময়ের মধ্যে request সংখ্যা সীমাবদ্ধ করে abuse ও overload কমানো।

### Q35. Health Check কী?
Application ও dependency—যেমন SQL Server/Redis—healthy কিনা পর্যবেক্ষণের endpoint/ব্যবস্থা।

## Scenario Questions

### Q36. API 500 দিচ্ছে—কীভাবে debug করবে?
1. Server log ও exception details দেখবে।
2. Request payload ও route যাচাই করবে।
3. DI registration এবং configuration পরীক্ষা করবে।
4. Database connection ও stored procedure পরীক্ষা করবে।
5. Debugger দিয়ে failure point বের করবে।
6. Production response-এ stack trace প্রকাশ করবে না।

### Q37. API 401 দিচ্ছে—কী পরীক্ষা করবে?
Token পাঠানো হয়েছে কি না, expiry, signature key, issuer, audience, signing algorithm, authentication middleware এবং Authorization header পরীক্ষা করবে।

### Q38. Angular থেকে API-তে CORS error হলে?
Exact frontend origin, allowed methods/headers, preflight OPTIONS response, CORS middleware order এবং credentials configuration যাচাই করবে।

## Revision Checklist

- [ ] REST & HTTP status codes
- [ ] DTO, Model Binding, Validation
- [ ] DI lifetime
- [ ] Middleware pipeline/order
- [ ] CORS
- [ ] JWT, Claims, Roles, Policies
- [ ] Exception handling & logging
- [ ] Rate limiting & caching
- [ ] API versioning & pagination
- [ ] Health checks & deployment
