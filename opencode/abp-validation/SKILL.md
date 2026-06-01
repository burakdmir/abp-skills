---
name: abp-validation
description: "ABP Framework v10.4 validation: DTO validation, Data Annotations, FluentValidation, IValidatableObject, AbpValidationException. Use when you need input validation or DTO validation in ABP."
---

# ABP Validation Skill

## Trigger
Validation, DTO validation, FluentValidation, IValidatableObject, validation errors, AbpValidationException, input validation.

---

## Quick Reference

### Data Annotations
```csharp
public class CreateBookDto
{
    [Required] [StringLength(100)] public string Name { get; set; }
    [Range(0, 999.99)] public decimal Price { get; set; }
}
```
Auto-validated on app service/controller call.

### IValidatableObject
```csharp
public class CreateBookDto : IValidatableObject
{
    public IEnumerable<ValidationResult> Validate(ValidationContext ctx)
    {
        var results = new List<ValidationResult>();
        if (Price <= 0) results.Add(new ValidationResult("Price > 0", new[] { nameof(Price) }));
        return results;
    }
}
```
Resolve services: `ctx.GetRequiredService<IMyService>()`

### FluentValidation
```bash
abp add-package Volo.Abp.FluentValidation
```
```csharp
public class CreateBookDtoValidator : AbstractValidator<CreateBookDto>
{
    public CreateBookDtoValidator()
    {
        RuleFor(x => x.Name).Length(3, 10);
        RuleFor(x => x.Price).ExclusiveBetween(0.0f, 999.0f);
    }
}
```
Auto-discovered, associated with DTO type.

### IValidationEnabled
```csharp
public class MyService : ITransientDependency, IValidationEnabled
{
    public virtual async Task DoItAsync(MyInput input) { }
}
```
Method must be `virtual` or use interface injection.

### Validation Error Response
```json
{
  "error": {
    "code": "App:010046",
    "message": "Your request is not valid, please correct and try again!",
    "validationErrors": [
      { "message": "Username min length 3.", "members": ["userName"] }
    ]
  }
}
```

### Best Practices
- Data Annotations → simple rules
- FluentValidation → complex/cross-property rules
- IValidatableObject → DTO-tight coupling only
- Domain validation → domain services, not DTOs
- Methods must be `virtual` for interception
- Localize validation messages

## Related

[DDD](../abp-ddd/SKILL.md) · [Exception Handling](../abp-exception-handling/SKILL.md) · [Localization](../abp-localization/SKILL.md) · Docs: https://abp.io/docs/latest/framework/fundamentals/validation
