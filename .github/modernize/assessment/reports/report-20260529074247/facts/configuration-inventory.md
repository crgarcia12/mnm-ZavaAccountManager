# Configuration & Externalized Settings Inventory

ZavaAccountManager uses a single `web.config` XML file as its only configuration source; there are no environment-specific overrides, external config servers, or secrets management integrations — all settings including credentials are hardcoded in that file.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|---|---|---|---|
| web.config | XML app config | `/web.config` | Single config file covering connection strings, app settings, authentication, and ASP.NET HTTP runtime settings |
| packages.config | NuGet package manifest | `/packages.config` | Empty — no third-party NuGet packages declared |
| ZavaAccountManager.csproj | MSBuild project file | `/ZavaAccountManager.csproj` | Declares framework references and compile targets |
| Dockerfile | Container definition | `/Dockerfile` | Defines build and runtime image using Mono 6.12 / xsp4 |

No external config server, Kubernetes ConfigMaps/Secrets, Azure App Configuration, HashiCorp Vault, or `.env` files are present.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies / Plugins |
|---|---|---|---|
| Debug | Default MSBuild configuration | Development build with debug symbols; outputs to `bin\` | None beyond default MSBuild targets |
| Release | Manual (`-c Release`) | Production build; outputs to `bin\` | None beyond default MSBuild targets |
| Docker (Mono) | `docker build` | Compiles source with `mcs` (Mono C# compiler) on Debian Buster, serves with `xsp4` | `mono:6.12` base image, `mono-xsp4` apt package |

No conditional compilation symbols or environment-specific build-time switches are declared.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Single (no profiles) | N/A — no profile mechanism | `web.config` only | No runtime profile system; all environments use the same config |

ASP.NET Web Forms on .NET Framework 4.8 does not have a built-in profile/environment system equivalent to `ASPNETCORE_ENVIRONMENT`. There are no `web.Development.config` or `web.Production.config` transform files.

## Properties Inventory

### ZavaAccountManager — `web.config`

| Property Key | Default / Value | Profile | Source |
|---|---|---|---|
| `connectionStrings > ZavaBankDb` | `Server=sqlserver,1433;Database=ZavaBankDB;User Id=sa;******;TrustServerCertificate=true` | All | `web.config` |
| `appSettings > AuthGatewayLoginUrl` | `http://localhost/auth/Login.aspx` | All | `web.config` |
| `appSettings > AuthGatewayLogoutUrl` | `http://localhost/auth/Logout.aspx` | All | `web.config` |
| `appSettings > LedgerBaseUrl` | `http://zava-ledger:8080` | All | `web.config` |
| `system.web > compilation debug` | `true` | All | `web.config` |
| `system.web > httpRuntime targetFramework` | `4.8` | All | `web.config` |
| `system.web > customErrors mode` | `Off` | All | `web.config` |
| `system.web > authentication mode` | `Forms` | All | `web.config` |
| `system.web > forms loginUrl` | `~/Login.aspx` | All | `web.config` |
| `system.web > forms timeout` | `30` (minutes) | All | `web.config` |
| `system.web > forms name` | `.ZAVAAUTH` | All | `web.config` |
| `system.web > forms protection` | `All` | All | `web.config` |
| `system.web > forms slidingExpiration` | `true` | All | `web.config` |
| `system.web > machineKey validationKey` | [MASKED — static value] | All | `web.config` |
| `system.web > machineKey decryptionKey` | [MASKED — static value] | All | `web.config` |
| `system.web > machineKey validation` | `SHA1` | All | `web.config` |
| `system.web > machineKey decryption` | `AES` | All | `web.config` |
| `system.web > authorization` | `deny users="?"` (default); `allow users="*"` for Login.aspx and Logout.aspx | All | `web.config` |

## Startup Parameters & Resource Requirements

| Service | Runtime Options | Port | Memory / CPU | Notes |
|---|---|---|---|---|
| ZavaAccountManager (Docker) | `xsp4 --port 8080 --address 0.0.0.0 --nonstop` | 8080 (container) | Not specified | No JVM; Mono CLR; no heap or resource limits declared in Dockerfile |
| ZavaAccountManager (IIS) | Default IIS/ASP.NET worker process settings | 80 (HTTP) | Not specified | Standard .NET Framework app pool; no custom machine.config or applicationHost.config overrides committed |

No `-Xms`/`-Xmx` equivalents, no `ASPNETCORE_ENVIRONMENT`, and no resource limits are configured.

## Startup Dependency Chain

The application has no explicit startup dependency declarations. At runtime it requires:

1. **SQL Server** (`sqlserver:1433`) — must be reachable before any page that queries the database is served.
2. **ZavaAuthGateway** (`AuthGatewayLoginUrl` / `AuthGatewayLogoutUrl`) — must be reachable for users to authenticate.
3. **Zava Ledger Service** (`zava-ledger:8080`) — required for balance/transaction pages; the application silently falls back to SQL on failure.

There are no `depends_on`, health-check wait mechanisms, readiness probes, or dockerize patterns defined. Startup failures of dependencies will result in runtime errors or partial degradation rather than a controlled startup failure.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Stored In | Notes |
|---|---|---|---|
| `ZavaBankDb` connection string password | Database credential | `web.config` (plaintext) | Password `[MASKED]` hardcoded; committed to source control |
| `machineKey validationKey` | Cookie signing key | `web.config` (plaintext) | Static key hardcoded; must be the same across all nodes but should not be in source control |
| `machineKey decryptionKey` | Cookie encryption key | `web.config` (plaintext) | Static AES key hardcoded; same risk as validationKey |
| `AuthGatewayLoginUrl` | External service URL | `web.config` | Points to `http://localhost` — plaintext HTTP, no TLS |
| `AuthGatewayLogoutUrl` | External service URL | `web.config` | Points to `http://localhost` — plaintext HTTP, no TLS |
| `LedgerBaseUrl` | External service URL | `web.config` | Points to `http://zava-ledger:8080` — plaintext HTTP, no TLS |

### Secrets Provisioning Workflow

No secrets provisioning workflow exists. All sensitive values (database password, machine keys, service URLs) are stored in plaintext in `web.config` and committed directly to the source repository. There is no integration with Azure Key Vault, HashiCorp Vault, AWS Secrets Manager, environment-variable injection, or any other secrets management system. Rotation of any credential requires editing and redeploying `web.config` manually.

## Feature Flags

No feature flag framework is used. There are no `@ConditionalOnProperty` equivalents, no `IFeatureManager` (.NET FeatureManagement), no LaunchDarkly or Unleash integrations, and no custom toggle patterns in the codebase.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework | 4.8 | `ZavaAccountManager.csproj` (`TargetFrameworkVersion`) |
| ASP.NET Web Forms | 4.8 (bundled with .NET Framework 4.8) | `web.config` (`targetFramework="4.8"`) |
| MSBuild Tools version | 4.0 | `ZavaAccountManager.csproj` (`ToolsVersion="4.0"`) |
| Mono (container runtime) | 6.12 | `Dockerfile` (`FROM mono:6.12`) |
| XSP4 (Mono web server) | Bundled with mono-xsp4 apt package | `Dockerfile` (`apt-get install mono-xsp4`) |
| Debian base image | Buster (archived) | `Dockerfile` (patched to use `archive.debian.org`) |
| System.Data.SqlClient | 4.8 (BCL) | `ZavaAccountManager.csproj` |
| System.Web / System.Web.Extensions | 4.8 (BCL) | `ZavaAccountManager.csproj` |
