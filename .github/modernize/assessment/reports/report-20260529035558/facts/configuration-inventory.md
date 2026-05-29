# Configuration & Externalized Settings Inventory

This repository uses a compact configuration model centered on web.config and project build metadata. Configuration is primarily local file based with no external configuration server integration detected.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| web.config | .NET XML runtime configuration | /web.config | Contains connection strings, appSettings, auth settings, machineKey |
| ZavaAccountManager.csproj | MSBuild project config | /ZavaAccountManager.csproj | Defines target framework, references, build output settings |
| packages.config | NuGet package list | /packages.config | Declares Newtonsoft.Json package |
| Dockerfile | Container runtime configuration | /Dockerfile | Defines mono runtime image and startup command |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | Build configuration selection | Local or development build output | Microsoft.CSharp.targets with framework references |
| Release | Build configuration selection | Release package output | Microsoft.CSharp.targets with framework references |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default | ASP.NET runtime load | web.config | targetFramework 4.8, forms auth, appSettings values |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| connectionStrings ZavaBankDb | Server sqlserver,1433;Database ZavaBankDB;User Id=sa;****** | Default | web.config |
| appSettings AuthGatewayLoginUrl | http://localhost/auth/Login.aspx | Default | web.config |
| appSettings AuthGatewayLogoutUrl | http://localhost/auth/Logout.aspx | Default | web.config |
| appSettings LedgerBaseUrl | http://zava-ledger:8080 | Default | web.config |
| system.web compilation debug | true | Default | web.config |
| system.web compilation targetFramework | 4.8 | Default | web.config |
| forms timeout | 30 | Default | web.config |
| forms cookie name | .ZAVAAUTH | Default | web.config |
| forms slidingExpiration | true | Default | web.config |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ZavaAccountManager (IIS/ASP.NET) | None explicitly configured in repository | Not specified | Not specified |
| ZavaAccountManager (Docker mono xsp4) | xsp4 --port 8080 --address 0.0.0.0 --nonstop | Not specified | 1 container process in Dockerfile |

## Startup Dependency Chain

1. ZavaAccountManager starts and loads web.config settings.
2. User requests trigger database access, requiring SQL Server availability at configured host and port.
3. Balance and transaction features require ledger service availability at configured base URL.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| ZavaBankDb connection string password | Database credential | web.config value [MASKED] |
| machineKey validationKey | Cryptographic key | web.config value [MASKED] |
| machineKey decryptionKey | Cryptographic key | web.config value [MASKED] |

### Secrets Provisioning Workflow

Secrets are statically provided in repository tracked configuration. At runtime the web application reads values directly from web.config and uses them for database and cryptographic operations. No managed identity, external secret store, or deployment time secret injection workflow is configured in this codebase.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N A | N A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework target | v4.8 | ZavaAccountManager.csproj |
| ASP.NET Web Forms | .NET Framework 4.8 stack | ZavaAccountManager.csproj and web.config |
| Newtonsoft.Json | 13.0.3 | packages.config |
| Mono container image | 6.12 | Dockerfile |
| MSBuild ToolsVersion | 4.0 | ZavaAccountManager.csproj |
