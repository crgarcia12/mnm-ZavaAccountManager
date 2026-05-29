# Architecture Diagram

This document summarizes the application's runtime architecture and key component relationships.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Bank Operations User"]
    end

    subgraph App["Application Layer - ASP.NET Web Forms on .NET Framework 4.8"]
        Pages["Web Forms Pages"]
        Auth["Forms Authentication"]
        Logic["Page Code Behind Logic"]
    end

    subgraph Data["Data Layer"]
        SqlClient["ADO.NET SqlClient"]
        DB[("SQL Server ZavaBankDB")]
    end

    subgraph External["External Services"]
        AuthGateway["ZavaAuthGateway"]
        Ledger["Zava Ledger API"]
    end

    Browser -->|"HTTPS/HTTP requests"| Pages
    Pages -->|"session and login checks"| Auth
    Pages -->|"account workflows"| Logic
    Logic -->|"SQL queries"| SqlClient
    SqlClient -->|"TDS"| DB
    Logic -->|"login/logout redirect"| AuthGateway
    Logic -->|"balance and transactions API calls"| Ledger
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | ASP.NET Web Forms | .NET Framework 4.8 | UI for customer/account operations |
| Application | Code-behind pages + Forms Auth | .NET Framework 4.8 | Business actions and access checks |
| Data Access | ADO.NET SqlClient | System.Data | Reads/writes customer/account/transaction records |
| Runtime | Mono + xsp4 (container path) | mono 6.12 | Linux container hosting option |

### Data Storage & External Services

The app stores operational data in SQL Server through a single connection string (`ZavaBankDb`). It also relies on external HTTP services for centralized authentication redirection and for account ledger balance/transaction retrieval.

### Key Architectural Decisions

- Uses server-side Web Forms with direct SQL access in code-behind rather than a separate repository/service layer.
- Uses forms authentication in `web.config`, with login/logout delegated to an external auth gateway URL.
- Implements resilient transaction display by falling back to local SQL transactions when remote ledger API calls fail.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        DefaultPage["Default.aspx"]
        LoginPage["Login.aspx"]
        LogoutPage["Logout.aspx"]
    end

    subgraph Business["Business Logic"]
        DefaultCode["Default.aspx.cs"]
        LoginCode["Login.aspx.cs"]
        LogoutCode["Logout.aspx.cs"]
    end

    subgraph DataAccess["Data Access"]
        SqlCommands["SqlConnection and SqlCommand"]
        SqlDb[("ZavaBankDB")]
    end

    subgraph Infra["Infrastructure"]
        FormsAuth["Forms Authentication"]
        LedgerApi["Ledger REST API"]
        Gateway["Auth Gateway"]
    end

    DefaultPage -->|"events"| DefaultCode
    LoginPage -->|"events"| LoginCode
    LogoutPage -->|"events"| LogoutCode
    DefaultCode -->|"query and update"| SqlCommands
    SqlCommands -->|"SQL"| SqlDb
    DefaultCode -->|"GET balance and transactions"| LedgerApi
    LoginCode -->|"redirect for sign in"| Gateway
    LogoutCode -->|"redirect for sign out"| Gateway
    FormsAuth -.->|"guards authenticated access"| DefaultPage
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| Default.aspx | Presentation | Web Forms Page | Main account management UI |
| Default.aspx.cs | Business Logic | Code-behind controller | Customer/account workflows, SQL and ledger calls |
| Login.aspx.cs | Business Logic | Code-behind controller | Redirects unauthenticated users to auth gateway |
| Logout.aspx.cs | Business Logic | Code-behind controller | Signs out and redirects to logout endpoint |
| SqlConnection/SqlCommand | Data Access | ADO.NET access components | Executes account/customer/transaction SQL |
| Forms Authentication | Infrastructure | Security middleware/config | Auth cookie handling and access control |
| Ledger REST API | Infrastructure | External service | Returns balance and transaction XML data |
| Auth Gateway | Infrastructure | External service | Centralized login/logout endpoints |
