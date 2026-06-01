---
name: abp-settings-features
description: "ABP Framework v10.4 settings and features: ISettingProvider/ISettingManager, SettingDefinitionProvider, IFeatureChecker, feature toggle. Use for configuration management, settings, or feature flags in ABP."
---

# ABP Settings & Features Skill

## Trigger
Settings, ISettingProvider, ISettingManager, setting values, features, IFeatureChecker, feature toggles, feature management.

---

## Settings

### Define
```csharp
public class BookStoreSettingDefinitionProvider : SettingDefinitionProvider
{
    public override void Define(ISettingDefinitionContext context)
    {
        context.Add(
            new SettingDefinition("App.UI.LayoutType", defaultValue: "LeftMenu",
                displayName: L["LayoutType"], isVisibleToClients: true),
            new SettingDefinition("Smtp.Password", defaultValue: "", isEncrypted: true)
        );
    }
}
```

### Read (ISettingProvider — cached, read-only)
```csharp
string v = await _provider.GetOrNullAsync("Smtp.UserName");
bool ssl = await _provider.GetAsync<bool>("Smtp.EnableSsl");
bool ssl = await _provider.IsTrueAsync("Smtp.EnableSsl");
int port = await _provider.GetAsync<int>("Smtp.Port");
int? port = (await _provider.GetOrNullAsync("Smtp.Port"))?.To<int>();
```
`ApplicationService` has `SettingProvider` property pre-injected.

### Write (ISettingManager — for UIs)
```csharp
await _mgr.SetForCurrentTenantAsync("App.UI.LayoutType", "LeftMenu");
await _mgr.SetForUserAsync(userId, "App.UI.LayoutType", "LeftMenu");
await _mgr.SetGlobalAsync("App.UI.LayoutType", "TopMenu");
string v = await _mgr.GetOrNullGlobalAsync("App.UI.LayoutType");
```

### Fallback Chain (bottom → top)
D (Default) → C (Configuration) → G (Global) → T (Tenant) → U (User)

### appsettings.json
```json
{ "Settings": { "Smtp.EnableSsl": "true" } }
```

### Custom Value Provider
```csharp
public class CustomProvider : SettingValueProvider
{
    public override string Name => "Custom";
    public override Task<string> GetOrNullAsync(SettingDefinition s) { ... }
}
Configure<AbpSettingOptions>(o => o.ValueProviders.Add<CustomProvider>());
```

---

## Features

### Define
```csharp
public class FeatureDefProvider : FeatureDefinitionProvider
{
    public override void Define(IFeatureDefinitionContext ctx)
    {
        var g = ctx.AddGroup("BookStore");
        g.AddFeature("BookStore.PdfReporting", defaultValue: "false",
            displayName: L["PdfReporting"], valueType: new ToggleStringValueType());
        g.AddFeature("BookStore.MaxProductCount", defaultValue: "10",
            valueType: new FreeTextStringValueType(new NumericValueValidator(0, 1000000)));
    }
}
```

### Value Types
- `ToggleStringValueType` → checkbox (on/off)
- `FreeTextStringValueType` → textbox (with optional validator)
- `SelectionStringValueType` → dropdown

### Check
```csharp
[RequiresFeature("BookStore.PdfReporting")]  // Attribute
if (await FeatureChecker.IsEnabledAsync("BookStore.PdfReporting")) { }  // Service
string v = await FeatureChecker.GetOrNullAsync("BookStore.MaxProductCount");
```

### Child Features
```csharp
var parent = g.AddFeature("BookStore.Reporting", valueType: new ToggleStringValueType());
parent.AddChild("BookStore.PdfExport", valueType: new ToggleStringValueType());
```

---

## Settings vs Features
| | Settings | Features |
|---|---|---|
| Purpose | Config values | Functionality toggles |
| Scope | User/Tenant/Global | Tenant only |
| Fallback | 5-level chain | Default only |
| Encryption | Yes | No |

## Best Practices
- ISettingProvider for read, ISettingManager for write
- isEncrypted: true for passwords/API keys
- isVisibleToClients: true only for browser-needed settings
- Features = tenant-scoped toggles
- [RequiresFeature] for automatic checking
- Localize displayName/description

## Related

[Authorization](../abp-authorization/SKILL.md) · [Multi-Tenancy](../abp-multitenancy/SKILL.md) · [Infrastructure](../abp-infrastructure/SKILL.md) · Docs: https://abp.io/docs/latest/framework/infrastructure/settings
