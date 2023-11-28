# C# Azure App Config - Feature Flag

Azure App Configuration service provides a centralized configuration store for your application settings. Feature flags, also known as feature toggles or feature switches, are a common use case for controlling the behavior of your application dynamically. In Azure App Configuration, you can implement feature flags using key-value pairs, where the key represents the feature flag and the value represents the state of the feature.

## C# Integration

Here's a step-by-step guide on how to integrate Azure App Configuration feature flags into a C# application:

### Step 1: Set Up Azure App Configuration

1. **`Create an Azure App Configuration service`**:

* Go to the Azure portal.
* Create a new Azure App Configuration service.
  
2. **`Add Feature Flags`**:

* In your App Configuration service, add key-value pairs where the keys represent your feature flags, and the values represent their states (e.g., "True" for enabled and "False" for disabled).

### Step 2: Install Required Packages

* Install the Microsoft.Azure.AppConfiguration.AspNetCore NuGet package to your C# project. You can do this using the following command in the Package Manager Console:

```bash
Install-Package Microsoft.Azure.AppConfiguration.AspNetCore
```

### Step 3: Update Startup.cs

* In your Startup.cs file, add the following code to configure Azure App Configuration:

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        // Other service configurations

        // Add Azure App Configuration
        services.AddAzureAppConfiguration(Configuration["ConnectionStrings:AppConfig"]);
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // Other app configurations

        // Enable feature flags
        app.UseAzureAppConfiguration();
    }
}
```

## Feature Flag as Configuration

1. **`Use Feature Flags in Your Code`**

In your C# code, you can now use the **`IConfiguration`** interface to access feature flags:

```csharp
using Microsoft.Extensions.Configuration;

public class MyClass
{
    private readonly IConfiguration _configuration;

    public MyClass(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void MyMethod()
    {
        // Check if a feature flag is enabled
        bool isFeatureEnabled = _configuration.GetValue<bool>("FeatureFlags:MyFeatureFlag");

        if (isFeatureEnabled)
        {
            // Feature-specific logic
        }
        else
        {
            // Default logic when the feature is disabled
        }
    }
}
```

Replace **`"FeatureFlags:MyFeatureFlag"`** with the key of your specific feature flag.

2. **`Update Feature Flags in Azure App Configuration`**

To dynamically control the state of your feature flags, you can update the values in the Azure App Configuration service without redeploying your application.

## Feature Flag as Controller Attribute

To implement feature flags using Azure App Configuration in a C# ASP.NET Core application with a controller attribute, you can follow these steps:

1. **`Install Required Packages`**:

* Make sure you have the necessary NuGet packages installed. You'll need the Microsoft.Extensions.Configuration.AzureAppConfiguration package to integrate Azure App Configuration.

```bash
dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
```

2. **`Configure Azure App Configuration`**:

* Set up Azure App Configuration in your Azure portal and retrieve the connection string.

3. **`Configure App Settings`**:

* Add the Azure App Configuration connection string to your appsettings.json or other configuration sources:

```json
{
  "ConnectionStrings": {
    "AppConfig": "Endpoint=<your-endpoint>;Id=<your-id>;Secret=<your-secret>"
  }
}
```

3. **`Configure Startup.cs`**:

In the Startup.cs file, configure the application to use Azure App Configuration:

```csharp
using Microsoft.Extensions.Configuration.AzureAppConfiguration;

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        // Other service configurations

        services.AddAzureAppConfiguration();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // Other configurations

        app.UseAzureAppConfiguration();
    }
}
```

4. **`Create a Feature Flag Attribute`**:

* Create a custom attribute that checks the feature flag value:

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = false, Inherited = false)]
public class FeatureFlagAttribute : ActionFilterAttribute
{
    private readonly string _featureFlagKey;

    public FeatureFlagAttribute(string featureFlagKey)
    {
        _featureFlagKey = featureFlagKey;
    }

    public override void OnActionExecuting(ActionExecutingContext context)
    {
        var config = context.HttpContext.RequestServices.GetRequiredService<IConfiguration>();
        var featureFlagValue = config[_featureFlagKey];

        if (string.IsNullOrEmpty(featureFlagValue) || !bool.Parse(featureFlagValue))
        {
            context.Result = new ContentResult
            {
                Content = "Feature not enabled",
                StatusCode = StatusCodes.Status403Forbidden,
            };
        }

        base.OnActionExecuting(context);
    }
}
```

5. **`Use the Feature Flag Attribute`**:

* Apply the FeatureFlag attribute to your controller methods:

```csharp
[ApiController]
[Route("api/[controller]")]
public class MyController : ControllerBase
{
    [HttpGet]
    [FeatureFlag("MyFeatureFlag")]
    public IActionResult Get()
    {
        return Ok("Feature is enabled!");
    }
}
```

In this example, the Get method is protected by the FeatureFlag attribute, which checks the value of the MyFeatureFlag feature flag in Azure App Configuration.

Ensure that your Azure App Configuration settings are correctly configured, and the feature flag key matches the key in your Azure App Configuration store. This way, you can control access to the controller action based on the feature flag's value.
