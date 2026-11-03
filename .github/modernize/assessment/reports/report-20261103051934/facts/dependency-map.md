# Dependency Map

The .NET solution resolves 301 unique NuGet package/version pairs across its project dependency graphs, including 78 top-level package entries. Package versions are centrally managed in `Directory.Packages.props`.

## Dependencies

```mermaid
flowchart LR
    App["eShopOnWeb solution"]
    subgraph Web["Web Frameworks"]
        AspNet["ASP.NET Core 8.0.2"]
        Blazor["Blazor WebAssembly 8.0.2"]
        Endpoints["MinimalApi.Endpoint 1.3.0"]
    end
    subgraph Data["Database and ORM"]
        EF["EF Core SQL Server 8.0.2"]
        EFMem["EF Core InMemory 8.0.2"]
        Specs["Ardalis.Specification.EntityFrameworkCore 7.0.0"]
    end
    subgraph Identity["Security and Identity"]
        AspIdentity["ASP.NET Core Identity 8.0.2"]
        Jwt["Microsoft.AspNetCore.Authentication.JwtBearer 8.0.2"]
        Tokens["System.IdentityModel.Tokens.Jwt 7.3.1"]
        AzureIdentity["Azure.Identity 1.10.4"]
    end
    subgraph Utilities["Utilities and Mapping"]
        AutoMapper["AutoMapper DI 12.0.1"]
        MediatR["MediatR 12.0.1"]
        FluentValidation["FluentValidation 11.9.0"]
        Json["System.Text.Json 8.0.3"]
        Guard["Ardalis.GuardClauses 4.0.1"]
        ApiEndpoints["Ardalis.ApiEndpoints 4.1.0"]
    end
    subgraph Tests["Test Frameworks"]
        XUnit["xUnit 2.7.0"]
        NSub["NSubstitute 5.1.0"]
        MSTest["MSTest 3.2.2"]
        TestSdk["Microsoft.NET.Test.Sdk 17.9.0"]
    end

    App -->|"web and UI"| Web
    App -->|"persistence"| Data
    App -->|"authentication"| Identity
    App -->|"application utilities"| Utilities
    App -.->|"tests"| Tests
```

### Dependency Summary

| Category | Count shown | Key Libraries | Notes |
|---|---:|---|---|
| Web Frameworks | 3 | ASP.NET Core, Blazor WebAssembly, MinimalApi.Endpoint | Framework references are centrally versioned |
| Database / ORM | 3 | EF Core SQL Server, EF Core InMemory, Ardalis Specification | Supports SQL Server and in-memory provider |
| Security / Identity | 4 | ASP.NET Core Identity, JWT bearer, Azure.Identity | Authentication is split between cookies and JWT |
| Utilities | 6 | AutoMapper, MediatR, FluentValidation, System.Text.Json, Guard Clauses, API Endpoints | Shared application and endpoint utilities |

### Version & Compatibility Risks

Projects target `net8.0`, with framework and EF Core packages pinned to 8.0.2 and `System.Text.Json` to 8.0.3. A legacy `Microsoft.AspNetCore.Mvc` 2.2.0 package is also declared centrally and merits dependency-usage review. The dependency scan found advisory matches; see the security assessment for severity-filtered findings.

### Notable Observations

- Central package management in `Directory.Packages.props` keeps versions aligned across modules.
- The solution includes both SQL Server and in-memory EF Core providers.
- The resolved graph contains 301 unique package/version pairs; the package CLI also reported that `docker-compose.dcproj` uses legacy `packages.config`, so its packages are not represented in the JSON dependency graph.

## Test Dependencies

| Framework | Version | Notes |
|---|---:|---|
| xUnit | 2.7.0 | Unit and application tests |
| xUnit runner | 2.5.6 | Visual Studio test adapter |
| MSTest | 3.2.2 | Present in central package versions |
| NSubstitute | 5.1.0 | Test doubles |
| Microsoft.NET.Test.Sdk | 17.9.0 | Test execution infrastructure |
| coverlet.collector | 6.0.2 | Coverage collection |

Total test-scope dependencies: 6 listed central packages. The repository has unit, functional, integration, and public API integration test projects.
