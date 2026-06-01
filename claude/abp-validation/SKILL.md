---
name: abp-validation
description: "ABP Framework v10.4 validation: DTO validation, Data Annotations, FluentValidation, IValidatableObject, AbpValidationException. Use when you need input validation or DTO validation in ABP."
---

# ABP Validation Skill

## Trigger
User asks about validation, DTO validation, FluentValidation, IValidatableObject, validation errors, AbpValidationException, or input validation in ABP Framework.

---

## Core Concepts

ABP provides automatic validation for application service inputs using:
1. **Data Annotation Attributes** — Declarative validation on DTOs
2. **IValidatableObject** — Custom validation logic in DTOs
3. **FluentValidation** — External validator classes
4. **IValidationEnabled** — Enable validation on any DI-registered service

Validation errors are automatically caught and returned as standardized error responses.

---

## Data Annotation Validation

### Basic Usage

```csharp
public class CreateBookDto
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Required]
    [StringLength(1000)]
    public string Description { get; set; }

    [Range(0, 999.99)]
    public decimal Price { get; set; }
}
```

- Automatically validated when used as application service/controller parameter
- Localized validation exception thrown and handled by ABP
- Common attributes: `Required`, `StringLength`, `Range`, `EmailAddress`, `RegularExpression`, `MinLength`, `MaxLength`

---

## IValidatableObject

### Custom Validation in DTOs

```csharp
public class CreateBookDto : IValidatableObject
{
    public string Name { get; set; }
    public decimal Price { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        var results = new List<ValidationResult>();

        if (Price <= 0)
        {
            results.Add(new ValidationResult(
                "Price must be greater than zero.",
                new[] { nameof(Price) }
            ));
        }

        if (string.IsNullOrWhiteSpace(Name))
        {
            results.Add(new ValidationResult(
                "Name cannot be empty.",
                new[] { nameof(Name) }
            ));
        }

        return results;
    }
}
```

### Resolving Services in Validate

```csharp
public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
{
    var myService = validationContext.GetRequiredService<IMyService>();
    // Use service for validation logic
    // ...
}
```

> **Warning:** Resolving services in `Validate` is possible but not recommended. Keep DTOs simple — they should transfer data, not contain domain validation logic.

---

## FluentValidation Integration

### Installation

```bash
abp add-package Volo.Abp.FluentValidation
```

Or manually:
```bash
dotnet add package Volo.Abp.FluentValidation
```

Add module dependency:
```csharp
[DependsOn(typeof(AbpFluentValidationModule))]
public class MyModule : AbpModule { }
```

### Usage

```csharp
public class CreateUpdateBookDtoValidator : AbstractValidator<CreateUpdateBookDto>
{
    public CreateUpdateBookDtoValidator()
    {
        RuleFor(x => x.Name).Length(3, 10);
        RuleFor(x => x.Price).ExclusiveBetween(0.0f, 999.0f);
        RuleFor(x => x.Description).NotEmpty().MaximumLength(1000);
    }
}
```

- ABP auto-discovers validator classes
- Automatically associated with the DTO type
- Can be placed in the same project as the DTO

---

## Validation Infrastructure

### IValidationEnabled Interface

Enable validation on any DI-registered service:

```csharp
public class MyService : ITransientDependency, IValidationEnabled
{
    public virtual async Task DoItAsync(MyInput input)
    {
        // input is automatically validated
    }
}
```

**Requirements:**
- Method must be `virtual` OR service used via interface
- Class must be registered in DI (implements `ITransientDependency`, `ISingletonDependency`, or `IScopedDependency`)

### Enabling/Disabling Validation

```csharp
public class MyService : ITransientDependency, IValidationEnabled
{
    public bool IsValidationEnabled { get; set; } = true; // Default

    public virtual async Task DoItAsync(MyInput input) { }
}
```

### AbpValidationException

Thrown automatically when validation fails:

```csharp
public class AbpValidationException : AbpException, IHasValidationErrors
{
    public IList<ValidationResult> ValidationErrors { get; }
}
```

- Implements `IHasValidationErrors` — errors serialized in API response
- Automatically handled by ABP's exception handler
- Returns HTTP 400 with validation error details

---

## Validation Error Response Format

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

---

## Best Practices

1. **Use Data Annotations** for simple, declarative validation (required, length, range)
2. **Use FluentValidation** for complex validation rules, cross-property validation
3. **Use IValidatableObject** sparingly — only when validation is tightly coupled to the DTO
4. **Keep domain validation in domain services**, not in DTOs
5. **Enable validation on application services** — they are validated by default
6. **Use IValidationEnabled** on custom services that need input validation
7. **Methods must be virtual** for interception-based validation to work
8. **Localize validation messages** using resource files for multi-language support

---

## Validation + Exception Handling Flow

```
Client Request
    ↓
DTO Deserialization
    ↓
ABP Validation Interceptor
    ↓ (if invalid)
AbpValidationException thrown
    ↓
ABP Exception Handler
    ↓
HTTP 400 + validationErrors JSON
    ↓
Client receives structured error
```

---

## Common Validation Attributes Reference

| Attribute | Purpose | Example |
|---|---|---|
| `[Required]` | Non-null, non-empty | `[Required] public string Name { get; set; }` |
| `[StringLength(max)]` | Max length | `[StringLength(100)]` |
| `[StringLength(min, max)]` | Min/max length | `[StringLength(3, 100)]` |
| `[Range(min, max)]` | Numeric range | `[Range(0, 999.99)]` |
| `[EmailAddress]` | Email format | `[EmailAddress]` |
| `[RegularExpression(pattern)]` | Regex match | `[RegularExpression(@"^[a-zA-Z]+$")]` |
| `[MinLength(n)]` | Minimum collection/string length | `[MinLength(3)]` |
| `[MaxLength(n)]` | Maximum collection/string length | `[MaxLength(100)]` |
| `[Url]` | URL format | `[Url]` |
| `[Phone]` | Phone format | `[Phone]` |

---

## Related

- [Exception Handling](../fundamentals/exception-handling.md) — How validation errors are handled
- [FluentValidation](https://fluentvalidation.net/) — External validation library docs
- [ASP.NET Core Validation](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation) — Microsoft's validation documentation
