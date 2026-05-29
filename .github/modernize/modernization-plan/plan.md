# Modernization Plan: ZavaAccountManager Azure Modernization

**Project**: ZavaAccountManager

---

## Technical Framework

- **Language**: C# (.NET Framework 4.8)
- **Framework**: ASP.NET Web Forms
- **Build Tool**: MSBuild
- **Database**: SQL Server (connection string in `web.config`)
- **Key Dependencies**: `System.Data.SqlClient`, `System.Web`

---

## Overview

> This migration modernizes configuration and data access dependencies for
> Azure readiness. The application currently uses hardcoded service URLs and
> SQL connection settings in `web.config`. The new architecture will:
>
> - Externalize non-secret application settings to centralized Azure config
> - Migrate database connectivity to Azure SQL with managed identity auth
> - Remediate dependency CVEs before deployment readiness
>
> The migration follows a phased approach: configuration modernization,
> database modernization, and security remediation.

---

## Migration Impact Summary

| Application | Original Service | New Azure Service | Authentication | Comments |
|-------------|------------------|-------------------|----------------|----------|
| ZavaAccountManager | Hardcoded app settings in `web.config` | Azure App Configuration | Managed Identity | Addresses issue: Hardcoded URLs detected |
| ZavaAccountManager | SQL conn string + `System.Data.SqlClient` | Azure SQL Database | Managed Identity | Addresses issues: SQL database connection detected; Connection strings without configuration builders detected; System.Data.SqlClient dependency detected |
