# ASP.NET Core Configuration Guide

## Table of Contents
- [Overview](#overview)
- [Configuration Files](#configuration-files)
- [Environment-Based Configuration](#environment-based-configuration)
- [Accessing Configuration Values](#accessing-configuration-values)
- [Implementation Example](#implementation-example)

## Overview

ASP.NET Core uses a flexible configuration system based on key-value pairs. These configurations can be loaded from multiple sources and can be environment-specific.

## Configuration Files

| File Name | Purpose | Environment |
|-----------|---------|-------------|
| appsettings.json | Base configuration file with default settings | All |
| appsettings.Development.json | Development-specific settings | Development |
| appsettings.Staging.json | Staging-specific settings | Staging |
| appsettings.Production.json | Production-specific settings | Production |

### Common Configuration Settings
- Connection Strings
- Logging Levels
- Application-specific settings
- Third-party service configurations

```json
{
    "ConnectionStrings": {
        "DefaultConnection": "Server=...;Database=...;"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Information"
        }
    },
    "MyKey": "MyValue"
}
```

## Environment-Based Configuration

```mermaid
graph TD
    A[Application Start] --> B{Check Environment}
    B -->|ASPNETCORE_ENVIRONMENT| C[Load Base Settings]
    C --> D[Load Environment Settings]
    D --> E{Environment Type}
    E -->|Development| F[appsettings.Development.json]
    E -->|Staging| G[appsettings.Staging.json]
    E -->|Production| H[appsettings.json]
```

### Environment Detection
- Located in `Properties/launchSettings.json`
- Controlled by `ASPNETCORE_ENVIRONMENT` variable
- Common values:
  - Development
  - Staging
  - Production

## Accessing Configuration Values

### Using IConfiguration Interface

```csharp
// Basic injection
private readonly IConfiguration _configuration;

public MyClass(IConfiguration configuration)
{
    _configuration = configuration;
}
```

### Reading Configuration Values

| Access Method | Example | Use Case |
|--------------|---------|----------|
| Direct Access | `_configuration["MyKey"]` | Simple key-value |
| Nested Access | `_configuration["ConnectionStrings:DefaultConnection"]` | Nested objects |
| Helper Method | `_configuration.GetConnectionString("DefaultConnection")` | Connection strings |

### Configuration Access Examples

```csharp
// Simple value
var simpleValue = _configuration["MyKey"];

// Nested value
var logLevel = _configuration["Logging:LogLevel:Default"];

// Connection string
var connString = _configuration.GetConnectionString("DefaultConnection");
```

## Implementation Example

```csharp
[Route("[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    private readonly IConfiguration _configuration;

    public AuthController(IAuthService authService, IConfiguration configuration)
    {
        _authService = authService;
        _configuration = configuration;
    }

    [HttpGet("Test")]
    public IActionResult Test()
    {
        var config = new
        {
            MyKey = _configuration["MyKey"]
        };
        return Ok(config);
    }

    [HttpPost("")]
    public async Task<IActionResult> LoginAsync(
        LoginRequest request, 
        CancellationToken cancellationToken)
    {
        var authResult = await _authService.GetTokenAsync(
            request.Email,
            request.Password,
            cancellationToken);

        return authResult is null 
            ? BadRequest("Invalid Email or Password") 
            : Ok(authResult);
    }
}
```

## Best Practices

1. **Environment-Specific Settings**
   - Keep sensitive data in environment-specific files
   - Use different connection strings per environment
   - Override settings as needed for each environment

2. **Configuration Organization**
   - Group related settings together
   - Use meaningful section names
   - Keep consistent naming conventions

3. **Security Considerations**
   - Never commit sensitive data to source control
   - Use user secrets in development
   - Use secure configuration providers in production

---

This guide provides a comprehensive overview of ASP.NET Core configuration system. For more detailed information, refer to the official Microsoft documentation.
