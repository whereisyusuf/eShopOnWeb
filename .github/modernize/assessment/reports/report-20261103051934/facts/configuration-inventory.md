# Configuration & Externalized Settings Inventory

Configuration is primarily supplied by per-host `appsettings` files, launch profiles, environment variables, and Docker Compose. Development defaults include local database settings; deployment secrets are intended to come from deployment configuration, though the Compose sample contains a literal SQL Server administrator credential (masked below).

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| App settings | JSON | `src/Web/appsettings*.json`, `src/PublicApi/appsettings*.json` | Default, Development, and Docker variants |
| Admin app settings | JSON | `src/BlazorAdmin/wwwroot/appsettings*.json` | Client API and web base URLs; values are public client configuration |
| Launch profiles | JSON | `src/{Web,PublicApi,BlazorAdmin}/Properties/launchSettings.json` | Local ports and Development/Production environment selection |
| Container composition | YAML | `docker-compose.yml`, `docker-compose.override.yml` | Web, API, and SQL Server; Compose SQL administrator password is a literal and is masked here |
| Deployment parameters | Bicep parameters | `infra/main.parameters.json` | Environment substitutions and Key Vault secret-generation references |
| Central package versions | MSBuild XML | `Directory.Packages.props` | Central package management and target framework |
| Environment variables | Runtime | ASP.NET Core configuration provider | Added by Web and PublicApi hosts; overrides appsettings values |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | MSBuild configuration | Local/debug build | No conditional dependency set identified |
| Release | MSBuild configuration | Optimized/publish build | No conditional dependency set identified |
| Docker build | Dockerfiles and Compose | Build and run Web/API containers | Docker SDK targets and runtime configuration |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` / launch profile | `appsettings.Development.json` | Local API/web URLs and development logging |
| Docker | Container environment and Compose | `appsettings.Docker.json` | Container API/web URLs and SQL Server connection settings |
| Production | `ASPNETCORE_ENVIRONMENT=Production` | Base settings plus environment overrides | Production launch profile exists for Web; deployment overrides should supply endpoints and connection configuration |

## Properties Inventory

| Service | Property Key | Default / Source | Profiles | Source |
|---|---|---|---|---|
| Web / PublicApi | `ConnectionStrings:CatalogConnection` | LocalDB in base settings; Docker SQL Server in container settings | Default, Docker | `appsettings*.json` |
| Web / PublicApi | `ConnectionStrings:IdentityConnection` | LocalDB in base settings; Docker SQL Server in container settings | Default, Docker | `appsettings*.json` |
| Web / PublicApi / Admin | `baseUrls:apiBase` | Local PublicApi URL | All | `appsettings*.json` |
| Web / PublicApi / Admin | `baseUrls:webBase` | Local Web URL | All | `appsettings*.json` |
| Web / PublicApi | `CatalogBaseUrl` | Empty string in base settings | Base | `appsettings.json` |
| Infrastructure | `UseOnlyInMemoryDatabase` | False when absent | Environment/configuration override | `Dependencies.ConfigureServices` |
| Web / PublicApi | Logging levels | Host-specific defaults | Development, Docker, base | `appsettings*.json` |
| API | JWT signing key | Compile-time constant (not shown) | All | `AuthorizationConstants` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | ASP.NET Core; no fixed startup options found | Not specified | Not specified |
| PublicApi | ASP.NET Core; no fixed startup options found | Not specified | Not specified |
| SQL Server | Container environment configuration | Not specified | One Compose service |
| BlazorAdmin | WebAssembly client configuration | Browser-managed | Not applicable |

## Startup Dependency Chain

1. SQL Server starts before Web and PublicApi according to Docker Compose `depends_on`.
2. Web and PublicApi initialize their EF Core contexts and seed catalog and identity data during startup.
3. BlazorAdmin requires PublicApi availability for API-backed admin operations.
4. No readiness health-check dependency or explicit wait-for mechanism was found in the Compose configuration.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| SQL Server administrator password | Database credential | `docker-compose.yml` literal `[MASKED]` |
| Catalog and identity connection strings | Database configuration | Local/deployment connection references; no credential values reproduced |
| Seed account password | Application credential | Compile-time constant `[MASKED]` |
| JWT signing key | Cryptographic key | Compile-time constant `[MASKED]` |
| SQL/application deployment passwords | Deployment secrets | `infra/main.parameters.json` Key Vault secret-generation references |

### Secrets Provisioning Workflow

Infrastructure parameters reference Azure Key Vault for SQL and application user passwords during deployment. Runtime applications receive database connection settings through appsettings or environment overrides. The source also contains development/sample credential constants and a Compose SQL administrator password; actual values are intentionally omitted from this inventory.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | `false` | Configuration value; enables EF Core in-memory stores |
| Environment-dependent developer exception page | Disabled outside Development | ASP.NET Core environment |
| Catalog image upload | Disabled in current API behavior | Endpoint implementation uses a placeholder image |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Target framework | `net8.0` | `Directory.Packages.props` |
| ASP.NET Core packages | 8.0.2 | `Directory.Packages.props` |
| Entity Framework Core | 8.0.2 | `Directory.Packages.props` |
| System.Text.Json | 8.0.3 | `Directory.Packages.props` |
| SQL Server container | Azure SQL Edge | `docker-compose.yml` |
| .NET SDK selection | 8.0.x with latest feature roll-forward | `global.json` |
