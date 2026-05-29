# Configuration & Externalized Settings Inventory

This application uses a compact configuration model centered on `web.config` for runtime behavior and a Dockerfile for container startup. Externalized values are primarily connection strings and service endpoint URLs.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| web.config | XML runtime config | `/tmp/workspace/crgarcia12/mnm-ZavaAccountManager/web.config` | Connection strings, app settings, auth, machine key |
| ZavaAccountManager.csproj | Build config | `/tmp/workspace/crgarcia12/mnm-ZavaAccountManager/ZavaAccountManager.csproj` | Target framework and assembly references |
| Dockerfile | Container runtime config | `/tmp/workspace/crgarcia12/mnm-ZavaAccountManager/Dockerfile` | Mono base image, compile/start commands, exposed port |
| packages.config | Package config | `/tmp/workspace/crgarcia12/mnm-ZavaAccountManager/packages.config` | Empty package manifest |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | `Configuration=Debug` | Development build output to `bin` | .NET Framework references in csproj |
| Release | `Configuration=Release` | Release build output to `bin` | .NET Framework references in csproj |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default ASP.NET runtime | IIS/ASP.NET runtime load | web.config | Forms auth enabled, custom errors off, app settings and connection string |
| Container runtime | Docker start command (`xsp4`) | Dockerfile | Listens on port 8080, uses Mono runtime |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| connectionStrings:ZavaBankDb | `Server=sqlserver,1433;Database=ZavaBankDB;...` | Default | web.config |
| appSettings:AuthGatewayLoginUrl | `http://localhost/auth/Login.aspx` | Default | web.config |
| appSettings:AuthGatewayLogoutUrl | `http://localhost/auth/Logout.aspx` | Default | web.config |
| appSettings:LedgerBaseUrl | `http://zava-ledger:8080` | Default | web.config |
| system.web/compilation@debug | `true` | Default | web.config |
| system.web/forms@timeout | `30` | Default | web.config |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| ZavaAccountManager (container path) | `xsp4 --port 8080 --address 0.0.0.0 --nonstop` | Not specified | Not specified |

## Startup Dependency Chain

1. ZavaAccountManager starts and loads `web.config` settings.
2. SQL Server (`sqlserver:1433`) must be reachable for core customer/account operations.
3. Zava Ledger service should be reachable for balance and transaction APIs (transactions have SQL fallback).
4. Auth gateway URLs must be reachable for full login/logout redirection experience.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `connectionStrings:ZavaBankDb` password segment | Database credential | web.config value present (masked in inventory) |
| `system.web/machineKey` validationKey | Cryptographic key | web.config value present (masked in inventory) |
| `system.web/machineKey` decryptionKey | Cryptographic key | web.config value present (masked in inventory) |

### Secrets Provisioning Workflow

Secrets are currently provisioned as static values in `web.config` rather than via an external secret manager. At runtime, the web application loads these values directly from configuration and uses them for SQL connectivity and authentication cookie cryptography.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework target | v4.8 | ZavaAccountManager.csproj |
| ASP.NET Web Forms runtime settings | 4.8 | web.config |
| Mono base image | 6.12 | Dockerfile |
| dotnet-appcat tool | 1.0.1127 | CLI execution output |
