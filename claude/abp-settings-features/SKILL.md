# ABP Settings & Features Skill

## Trigger
User asks about settings, ISettingProvider, ISettingManager, ISettingDefinitionProvider, setting values, features, IFeatureChecker, IFeatureDefinitionProvider, feature toggles, or feature management in ABP Framework.

---

## Part 1: Settings System

### Core Concepts

ABP's setting system provides a hierarchical, extensible way to manage configuration values with fallback from user → tenant → global → configuration → default.

### Defining Settings

Create a class inheriting `SettingDefinitionProvider`:

```csharp
using Volo.Abp.Settings;

namespace Acme.BookStore.Settings
{
    public class BookStoreSettingDefinitionProvider : SettingDefinitionProvider
    {
        public override void Define(ISettingDefinitionContext context)
        {
            context.Add(
                new SettingDefinition(
                    "App.UI.LayoutType",
                    defaultValue: "LeftMenu",
                    displayName: L["LayoutType"],
                    isVisibleToClients: true
                ),
                new SettingDefinition(
                    "Smtp.EnableSsl",
                    defaultValue: "false",
                    displayName: L["EnableSsl"],
                    isVisibleToClients: false
                )
            );
        }

        private static LocalizableString L(string name)
        {
            return LocalizableString.Create<BookStoreResource>(name);
        }
    }
}
```

- ABP auto-discovers this class
- `defaultValue` is a string — all setting values stored as strings
- `isVisibleToClients: true` exposes the value to the browser for conditional UI

### Changing Setting Definitions of a Dependent Module

```csharp
public class MyModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        PreConfigure<SettingDefinitionContext>(options =>
        {
            // Modify identity module settings before they're finalized
        });
    }
}
```

### Reading Setting Values

```csharp
public class MyService : ITransientDependency
{
    private readonly ISettingProvider _settingProvider;

    public MyService(ISettingProvider settingProvider)
    {
        _settingProvider = settingProvider;
    }

    public async Task FooAsync()
    {
        // Get as string (null if not set)
        string userName = await _settingProvider.GetOrNullAsync("Smtp.UserName");

        // Get as bool, fallback to default
        bool enableSsl = await _settingProvider.GetAsync<bool>("Smtp.EnableSsl");

        // Get as bool, fallback to provided default
        bool enableSsl = await _settingProvider.GetAsync<bool>(
            "Smtp.EnableSsl", defaultValue: true);

        // Shortcut: IsTrueAsync
        bool enableSsl = await _settingProvider.IsTrueAsync("Smtp.EnableSsl");

        // Get as int
        int port = await _settingProvider.GetAsync<int>("Smtp.Port");

        // Get as nullable int
        int? port = (await _settingProvider.GetOrNullAsync("Smtp.Port"))?.To<int>();
    }
}
```

> `ApplicationService`, `DomainService`, and other base classes already property-inject `ISettingProvider`. Use `SettingProvider` property directly.

### Reading on Client Side

Settings with `isVisibleToClients: true` are available via JavaScript:

```javascript
const layoutType = abp.setting.values['App.UI.LayoutType'];
```

### Setting Value Providers (Fallback Chain)

5 pre-built providers, evaluated bottom → top:

| Provider | Name | Source |
|---|---|---|
| DefaultValueSettingValueProvider | "D" | Default value in setting definition |
| ConfigurationSettingValueProvider | "C" | IConfiguration (appsettings.json) |
| GlobalSettingValueProvider | "G" | System-wide (database) |
| TenantSettingValueProvider | "T" | Current tenant (database) |
| UserSettingValueProvider | "U" | Current user (database) |

### Setting Values in Application Configuration

In `appsettings.json`:
```json
{
  "Settings": {
    "Smtp.EnableSsl": "true",
    "Smtp.Port": "587"
  }
}
```

### Encrypting Setting Values

```csharp
public class MySettingDefinitionProvider : SettingDefinitionProvider
{
    public override void Define(ISettingDefinitionContext context)
    {
        context.Add(
            new SettingDefinition(
                "Smtp.Password",
                defaultValue: "",
                isVisibleToClients: false,
                isEncrypted: true  // Encrypted in storage
            )
        );
    }
}
```

### Custom Setting Value Providers

```csharp
public class CustomSettingValueProvider : SettingValueProvider
{
    public override string Name => "Custom";

    public CustomSettingValueProvider(ISettingStore settingStore)
        : base(settingStore) { }

    public override Task<string> GetOrNullAsync(SettingDefinition setting)
    {
        // Return setting value or null
        // Use SettingStore or another data source
    }
}

// Register
Configure<AbpSettingOptions>(options =>
{
    options.ValueProviders.Add<CustomSettingValueProvider>();
});
```

### ISettingEncryptionService

Custom encryption implementation:

```csharp
public class MyEncryptionService : ISettingEncryptionService, ITransientDependency
{
    public string Decrypt(string encryptedValue) { /* ... */ }
    public string Encrypt(string plainValue) { /* ... */ }
}
```

---

## Part 2: Setting Management Module

### ISettingManager

Used to **get and set** setting values (for building setting management UIs):

```csharp
public class MyService : ITransientDependency
{
    private readonly ISettingManager _settingManager;

    public MyService(ISettingManager settingManager)
    {
        _settingManager = settingManager;
    }

    public async Task FooAsync()
    {
        Guid user1Id = ...;
        Guid tenant1Id = ...;

        // Current user
        string layout = await _settingManager.GetOrNullForCurrentUserAsync("App.UI.LayoutType");
        await _settingManager.SetForCurrentUserAsync("App.UI.LayoutType", "LeftMenu");

        // Specific user
        await _settingManager.SetForUserAsync(user1Id, "App.UI.LayoutType", "LeftMenu");

        // Current tenant
        await _settingManager.SetForCurrentTenantAsync("App.UI.LayoutType", "LeftMenu");

        // Specific tenant
        await _settingManager.SetForTenantAsync(tenant1Id, "App.UI.LayoutType", "LeftMenu");

        // Global
        await _settingManager.SetGlobalAsync("App.UI.LayoutType", "TopMenu");
        string global = await _settingManager.GetOrNullGlobalAsync("App.UI.LayoutType");
    }
}
```

> Use `ISettingProvider` for **reading only** (implements caching). Use `ISettingManager` for **setting management UIs**.

### Setting Cache

Setting values are cached via distributed cache. Always use `ISettingManager` to change values — it manages the cache.

### Setting Management Providers

5 pre-built providers (reverse order execution):

| Provider | Can Get | Can Set |
|---|---|---|
| DefaultValueSettingManagementProvider | Yes | No |
| ConfigurationSettingManagementProvider | Yes | No |
| GlobalSettingManagementProvider | Yes | Yes |
| TenantSettingManagementProvider | Yes | Yes |
| UserSettingManagementProvider | Yes | Yes |

### Custom Setting Management Provider

```csharp
public class CustomSettingProvider : SettingManagementProvider, ITransientDependency
{
    public override string Name => "Custom";

    public CustomSettingProvider(ISettingManagementStore store)
        : base(store) { }
}

Configure<SettingManagementOptions>(options =>
{
    options.Providers.Add<CustomSettingProvider>();
});
```

### Setting Management UI

The module provides default UI for:
- Email settings (with "Send test email" button)
- Feature management
- Timezone settings

Extensible — add custom tabs:

**MVC:**
```csharp
public class MySettingGroupViewComponent : AbpSettingManagementViewComponent
{
    public IViewComponentResult Invoke()
    {
        return View("~/Views/Shared/Components/MySettingGroup/Default.cshtml");
    }
}
```

---

## Part 3: Features System

### Core Concepts

Features are tenant-scoped toggles that enable/disable functionality per tenant. Different from settings — features control **what a tenant can use**.

### Defining Features

```csharp
using Volo.Abp.Features;
using Volo.Abp.Validation.StringValues;

namespace Acme.BookStore.Features
{
    public class BookStoreFeatureDefinitionProvider : FeatureDefinitionProvider
    {
        public override void Define(IFeatureDefinitionContext context)
        {
            var myGroup = context.AddGroup("BookStore");

            // Boolean toggle feature
            myGroup.AddFeature(
                "BookStore.PdfReporting",
                defaultValue: "false",
                displayName: LocalizableString.Create<BookStoreResource>("PdfReporting"),
                valueType: new ToggleStringValueType()
            );

            // Numeric feature with validator
            myGroup.AddFeature(
                "BookStore.MaxProductCount",
                defaultValue: "10",
                displayName: LocalizableString.Create<BookStoreResource>("MaxProductCount"),
                valueType: new FreeTextStringValueType(
                    new NumericValueValidator(0, 1000000)
                )
            );
        }
    }
}
```

### Feature Value Types

| Type | UI | Use Case |
|---|---|---|
| `ToggleStringValueType` | Checkbox | on/off, enabled/disabled |
| `FreeTextStringValueType` | Textbox | Free text, numbers |
| `SelectionStringValueType` | Dropdown | Select from predefined list |

### Other Feature Properties

```csharp
myGroup.AddFeature(
    "BookStore.Advanced",
    defaultValue: "false",
    displayName: L["AdvancedFeature"],
    description: L["AdvancedFeatureDescription"],
    valueType: new ToggleStringValueType(),
    isVisibleToClients: true,  // Expose to browser (default: true)
    properties: new Dictionary<string, object>
    {
        { "CustomProperty", "value" }
    }
);
```

### Child Features

```csharp
var reporting = myGroup.AddFeature(
    "BookStore.Reporting",
    defaultValue: "false",
    displayName: L["Reporting"],
    valueType: new ToggleStringValueType()
);

reporting.AddChild(
    "BookStore.PdfExport",
    defaultValue: "false",
    displayName: L["PdfExport"],
    valueType: new ToggleStringValueType()
);
```

Child features only available when parent is enabled.

### Checking Features

#### RequiresFeature Attribute

```csharp
[RequiresFeature("BookStore.PdfReporting")]
public class PdfReportAppService : ApplicationService, IPdfReportAppService
{
    public async Task<PdfReportResultDto> GetPdfReportAsync()
    {
        // Only accessible if feature is enabled
    }
}
```

#### IFeatureChecker Service

```csharp
public class ReportingAppService : ApplicationService
{
    public async Task DoWorkAsync()
    {
        // Check if enabled
        if (await FeatureChecker.IsEnabledAsync("BookStore.PdfReporting"))
        {
            // Feature is enabled
        }

        // Get value
        string maxCount = await FeatureChecker.GetOrNullAsync("BookStore.MaxProductCount");

        // Extension methods
        bool isEnabled = await FeatureChecker.IsTrueAsync("BookStore.PdfReporting");
        int max = (await FeatureChecker.GetAsync<int>("BookStore.MaxProductCount"));
    }
}
```

> `ApplicationService` base class already has `FeatureChecker` property injected.

### Feature Management Modal

Features are managed via the Feature Management UI (available with Identity module):
- Shown per-tenant
- Toggle features on/off
- Set feature values
- Child features shown nested under parent

---

## Best Practices

### Settings
1. Use `ISettingProvider` for reading (caching), `ISettingManager` for writing
2. Set `isVisibleToClients: true` only for settings needed in browser
3. Use `isEncrypted: true` for sensitive values (passwords, API keys)
4. Group settings with prefixes: `App.`, `Smtp.`, `Emailing.`
5. Always provide sensible `defaultValue`
6. Localize `displayName` for UI display

### Features
1. Use features for **tenant-scoped** functionality toggles
2. Use `ToggleStringValueType` for simple on/off features
3. Use `FreeTextStringValueType` with validators for numeric limits
4. Use child features for dependent functionality
5. Use `[RequiresFeature]` attribute for automatic feature checking
6. Set `isVisibleToClients: false` for internal-only features
7. Localize feature `displayName` and `description`

---

## Settings vs Features

| Aspect | Settings | Features |
|---|---|---|
| Purpose | Configuration values | Functionality toggles |
| Scope | User, Tenant, Global | Tenant only |
| Value types | Any string | Toggle, FreeText, Selection |
| UI | Setting Management page | Feature Management modal |
| Fallback | 5-level chain | Default value only |
| Encryption | Supported | Not typically needed |
| Client access | `isVisibleToClients` | `isVisibleToClients` |

---

## Related

- [Setting Management Module](../../modules/setting-management.md) — Full module docs
- [Multi-Tenancy](../architecture/multi-tenancy/index.md) — Tenant-scoped settings
- [Configuration](../fundamentals/configuration.md) — appsettings.json integration
- [Options Pattern](../fundamentals/options.md) — Configure options
