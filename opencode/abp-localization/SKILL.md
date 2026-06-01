---
name: abp-localization
description: "ABP Framework v10.4 localization: localization resource, JSON files, culture fallback, L[] helper, IStringLocalizer. Use for multi-language, localization, or translation in ABP."
---

# ABP Localization Skill

## Trigger
Localization, i18n, localization resources, JSON files, culture, L[] helper, multi-language, text translation.

---

## Quick Reference

### Create Resource
```csharp
public class BookStoreResource { }
```

### Register Resource
```csharp
Configure<AbpVirtualFileSystemOptions>(options =>
{
    options.FileSets.AddEmbedded<MyModule>("YourRootNamespace");
});
Configure<AbpLocalizationOptions>(options =>
{
    options.Resources.Add<BookStoreResource>("en")
        .AddVirtualJson("/Localization/Resources/BookStore");
});
```

### JSON File
```json
{
  "culture": "en",
  "texts": {
    "HelloWorld": "Hello World!",
    "Menu:Home": "Home",
    "Hello": { "World": "Hello World!" }
  }
}
```
- `culture` field required (ABP ignores without it)
- Nested keys: `L["Hello__World"]` (double underscore)

### Usage
```csharp
// In ApplicationService
LocalizationResource = typeof(BookStoreResource);
var text = L["HelloWorld"];

// Via DI
private readonly IStringLocalizer<BookStoreResource> _localizer;
var text = _localizer["HelloWorld"];
```

### Multiple Files per Culture
```
Localization/MyResource/
├── en.json
├── en_Authors.json
├── en_Books.json
└── en_Users.json
```
Auto-merged. Split large modules by feature.

### Best Practices
- One resource per module
- Key prefixes: `Menu:`, `Permission:`, `Error:`
- Split large files by feature
- Always define `culture` in JSON
- Embed via Virtual File System
- Set `LocalizationResource` in base classes
- Localize all user-facing text

## Related

[Exception Handling](../abp-exception-handling/SKILL.md) · [UI](../abp-ui/SKILL.md) · [Settings & Features](../abp-settings-features/SKILL.md) · Docs: https://abp.io/docs/latest/framework/fundamentals/localization
