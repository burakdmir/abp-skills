# ABP Exception Handling Skill

## Trigger
User asks about exception handling, error handling, RemoteServiceErrorResponse, error codes, business exceptions, HTTP status mapping, self-logging exceptions, or API error responses in ABP Framework.

---

## Core Concepts

ABP provides a built-in exception handling infrastructure that:
- **Automatically handles all exceptions** and sends standardized error messages for API/AJAX requests
- **Hides internal infrastructure errors** and returns safe error messages
- **Maps standard exceptions to HTTP status codes**
- **Supports localized exception messages**
- **Provides configurable custom exception mapping**

---

## Error Message Format

All API errors return a `RemoteServiceErrorResponse` JSON:

### Basic Error
```json
{
  "error": {
    "message": "This topic is locked and can not add a new message"
  }
}
```

### Error with Code
```json
{
  "error": {
    "code": "App:010046",
    "message": "An error occurred while processing your request."
  }
}
```

Implement `IHasErrorCode` to include error codes:
```csharp
public class MyBusinessException : Exception, IHasErrorCode
{
    public string Code => "App:010046";
}
```

### Error with Details
```json
{
  "error": {
    "code": "App:010046",
    "message": "Something went wrong.",
    "details": "Stack trace or additional context..."
  }
}
```

Implement `IHasErrorDetails` to include details:
```csharp
public class MyException : Exception, IHasErrorDetails
{
    public string Details => "Additional diagnostic information...";
}
```

### Validation Errors
```json
{
  "error": {
    "code": "App:010046",
    "message": "Your request is not valid, please correct and try again!",
    "validationErrors": [
      {
        "message": "Username should be minimum length of 3.",
        "members": ["userName"]
      },
      {
        "message": "Password is required",
        "members": ["password"]
      }
    ]
  }
}
```

`AbpValidationException` implements `IHasValidationErrors` and is auto-thrown on invalid input.

---

## Business Exceptions

### Creating Custom Business Exceptions

```csharp
public class BookNotFoundException : Exception
{
    public Guid BookId { get; }

    public BookNotFoundException(Guid bookId)
        : base($"Book not found with id: {bookId}")
    {
        BookId = bookId;
    }
}
```

### With Error Code

```csharp
public class InsufficientStockException : Exception, IHasErrorCode
{
    public string Code => "BookStore:010001";

    public InsufficientStockException(string message) : base(message) { }
}
```

### With Localized Message

```csharp
public class UserFriendlyException : Exception
{
    public UserFriendlyException(
        string message,
        string details = null,
        string code = null
    ) : base(message) { }
}
```

Use localization resource:
```csharp
throw new UserFriendlyException(
    L["ErrorMessage:InsufficientStock"]
);
```

---

## HTTP Status Code Mapping

ABP automatically maps exceptions to HTTP status codes:

| Exception Type | HTTP Status |
|---|---|
| `AbpValidationException` | 400 Bad Request |
| `AbpAuthorizationException` | 403 Forbidden |
| `EntityNotFoundException` | 404 Not Found |
| `UserFriendlyException` | 400 Bad Request |
| `EntityAlreadyExistsException` | 409 Conflict |
| `NonImplementException` | 501 Not Implemented |
| Generic `Exception` | 500 Internal Server Error |

### Custom Exception Mapping

```csharp
Configure<AbpExceptionHandlingOptions>(options =>
{
    options.MapToStatusCode<MyCustomException>(StatusCodes.Status409Conflict);
});
```

---

## Logging

### Automatic Logging

Caught exceptions are automatically logged by ABP.

### Log Level Control

Implement `IHasLogLevel` to control the log level:

```csharp
public class MyExpectedException : Exception, IHasLogLevel
{
    public LogLevel LogLevel => LogLevel.Warning;

    public MyExpectedException(string message) : base(message) { }
}
```

Default log levels:
- `AbpValidationException` → `Warning`
- `AbpAuthorizationException` → `Warning`
- `UserFriendlyException` → `Warning`
- Generic `Exception` → `Error`

### Self-Logging Exceptions

Exceptions can write additional logs by implementing `IExceptionWithSelfLogging`:

```csharp
public class MyException : Exception, IExceptionWithSelfLogging
{
    public void Log(ILogger logger)
    {
        logger.LogWarning("Additional context: {ContextData}", SomeData);
    }
}
```

Use `ILogger.LogException` extension method for manual logging:
```csharp
logger.LogException(myException);
```

---

## Exception Handling Options

```csharp
Configure<AbpExceptionHandlingOptions>(options =>
{
    // Send exception details to client (development only!)
    options.SendExceptionsDetailsToClients = true;

    // Include stack trace in response (development only!)
    options.IncludeStackTraceInExceptionDetails = true;

    // Custom exception mappers
    options.MapToStatusCode<MyException>(StatusCodes.Status409Conflict);
});
```

> **Warning:** Never enable `SendExceptionsDetailsToClients` or `IncludeStackTraceInExceptionDetails` in production — they expose internal implementation details.

---

## Best Practices

1. **Use `UserFriendlyException`** for errors that should be shown directly to users
2. **Implement `IHasErrorCode`** for machine-readable error codes (client-side handling)
3. **Implement `IHasLogLevel`** to control noise in logs (expected errors = Warning)
4. **Implement `IExceptionWithSelfLogging`** for exceptions that need additional context logged
5. **Map custom exceptions to HTTP status codes** using `AbpExceptionHandlingOptions`
6. **Localize error messages** for multi-language support
7. **Never expose stack traces** in production
8. **Use specific exception types** for different error scenarios (not generic `Exception`)
9. **Let ABP handle exceptions** — don't catch and re-throw unless adding context
10. **Use `EntityNotFoundException`** for missing entities (auto-mapped to 404)

---

## Common Patterns

### Application Service Error Handling

```csharp
public class BookAppService : ApplicationService
{
    public async Task<BookDto> GetAsync(Guid id)
    {
        var book = await _bookRepository.FindAsync(id);
        if (book == null)
        {
            throw new EntityNotFoundException(typeof(Book), id);
        }
        return ObjectMapper.Map<Book, BookDto>(book);
    }
}
```

### Business Rule Violation

```csharp
public class InsufficientStockException : Exception, IHasErrorCode
{
    public string Code => "BookStore:010001";
    public int RequestedQuantity { get; }
    public int AvailableQuantity { get; }

    public InsufficientStockException(int requested, int available)
        : base($"Requested {requested} but only {available} available")
    {
        RequestedQuantity = requested;
        AvailableQuantity = available;
    }
}
```

### Client-Side Error Handling

```typescript
try {
    await bookService.create(input);
} catch (error) {
    if (error.response?.data?.error?.code === 'BookStore:010001') {
        // Handle insufficient stock
        showStockWarning(error.response.data.error.validationErrors);
    }
}
```

---

## Exception Flow

```
Request arrives
    ↓
ABP handles request
    ↓
Exception thrown
    ↓
ABP Exception Handler intercepts
    ↓
Determines HTTP status code
    ↓
Logs exception (respects IHasLogLevel)
    ↓
Calls IExceptionWithSelfLogging.Log() if implemented
    ↓
Builds RemoteServiceErrorResponse
    ↓
Returns HTTP response with error JSON
```

---

## Related

- [Validation](./validation.md) — AbpValidationException and validation errors
- [Audit Logging](./audit-logging.md) — Exceptions are logged in audit trail
- [Logging](./logging.md) — ASP.NET Core logging infrastructure
