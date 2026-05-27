# ABP Exception Handling Skill

## Trigger
Exception handling, error handling, RemoteServiceErrorResponse, error codes, business exceptions, HTTP status mapping, API errors.

---

## Quick Reference

### Error Response Format
```json
{
  "error": {
    "code": "App:010046",
    "message": "Your request is not valid!",
    "validationErrors": [{ "message": "Min length 3", "members": ["userName"] }],
    "details": "..."
  }
}
```

### Custom Exception with Code
```csharp
public class InsufficientStockException : Exception, IHasErrorCode
{
    public string Code => "BookStore:010001";
}
```

### With Log Level
```csharp
public class ExpectedException : Exception, IHasLogLevel
{
    public LogLevel LogLevel => LogLevel.Warning;
}
```

### Self-Logging
```csharp
public class MyException : Exception, IExceptionWithSelfLogging
{
    public void Log(ILogger logger) => logger.LogWarning("Context: {Data}", Data);
}
```

### HTTP Status Mapping
```csharp
Configure<AbpExceptionHandlingOptions>(options =>
{
    options.MapToStatusCode<MyException>(StatusCodes.Status409Conflict);
    options.SendExceptionsDetailsToClients = false; // Production: always false
});
```

### Auto Mapped Exceptions
| Exception | HTTP Status |
|---|---|
| AbpValidationException | 400 |
| AbpAuthorizationException | 403 |
| EntityNotFoundException | 404 |
| UserFriendlyException | 400 |
| EntityAlreadyExistsException | 409 |
| Exception | 500 |

### Best Practices
- UserFriendlyException → user-facing errors
- IHasErrorCode → machine-readable codes
- IHasLogLevel → control log noise
- IExceptionWithSelfLogging → extra context
- Never expose stack traces in production
- Use specific exception types
- Localize error messages
