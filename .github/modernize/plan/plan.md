# Modernization Plan: Modernize Zava Account Manager for Azure

**Project**: ZavaAccountManager

---

## Technical Framework

- **Language**: C# (.NET Framework 4.8)
- **Framework**: ASP.NET Web Forms
- **Build Tool**: MSBuild (.csproj)
- **Database**: SQL Server (ZavaBankDB)
- **Key Dependencies**: System.Web, System.Data.SqlClient, Forms Authentication

---

## Overview

> This migration modernizes the Zava Account Manager application and deploys it
> to Azure. The application currently runs as a legacy ASP.NET Web Forms app on
> .NET Framework 4.8 with local configuration and service endpoints.
>
> The new architecture will:
>
> - Upgrade runtime and project structure to the latest supported .NET LTS stack
> - Adopt Azure cloud-native services for identity, configuration, and data
> - Deploy the modernized application to Azure Container Apps
>
> The migration follows a phased approach: baseline and upgrade first, then
> cloud-native service migration, security hardening, and Azure deployment.

---

## Migration Impact Summary

| Application | Original Service | New Azure Service | Authentication | Comments |
|-------------|------------------|-------------------|----------------|----------|
| ZavaAccountManager | .NET Framework 4.8 | .NET 10 (LTS) | N/A | Upgrade runtime and project format |
| ZavaAccountManager | SQL Server + local config | Azure SQL + Azure App Configuration + Key Vault | Managed Identity | Move to cloud-native services |
| ZavaAccountManager | Local/legacy hosting | Azure Container Apps | Managed Identity | Deploy modernized app to Azure |

---

## Security Compliance

**Description**: Scan all project dependencies for known CVEs and remediate any identified vulnerabilities to ensure the application is secure before deployment.

**Requirements**:
Upgrade vulnerable dependencies to the minimum patched version. If a CVE fix requires a major version upgrade, document the affected dependency, the current version, the upgraded major version, and the breaking change risk. Verify that the project builds and all tests pass after remediation.

**Environment Configuration**:
Runtime environment established by previous tasks (e.g., Java Home, .NET runtime).
Build tool established by previous tasks (e.g., Maven/Gradle, dotnet).

**App Scope**:
The app folders that this task will operate on

**Skills**:
- Skill Name: validate-cves-and-fix
  - Skill Location: builtin
