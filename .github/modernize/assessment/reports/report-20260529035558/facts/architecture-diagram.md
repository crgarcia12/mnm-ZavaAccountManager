# Architecture Diagram

This document summarizes the current application architecture and key internal component relationships for ZavaAccountManager.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - ASP.NET Web Forms on .NET Framework 4.8"]
        Pages["ASPX Pages"]
        CodeBehind["Page CodeBehind"]
        Auth["Forms Authentication"]
    end
    subgraph Data["Data Layer"]
        ADO["ADO.NET SqlClient"]
        SQLDB[("SQL Server ZavaBankDB")]
    end
    subgraph External["External Services"]
        Ledger["Ledger Service REST API"]
        AuthGateway["Auth Gateway URLs"]
    end

    Browser -->|"HTTP requests"| Pages
    Pages -->|"events and binding"| CodeBehind
    CodeBehind -->|"auth checks"| Auth
    CodeBehind -->|"SQL queries"| ADO
    ADO -->|"T SQL"| SQLDB
    CodeBehind -->|"balance and transactions calls"| Ledger
    CodeBehind -->|"login logout redirects"| AuthGateway
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | ASP.NET Web Forms | .NET Framework 4.8 | Server rendered UI for account management |
| Business Logic | C# code behind classes | .NET Framework 4.8 | Handles page events and account operations |
| Data Access | System.Data.SqlClient | .NET Framework library | Executes SQL against banking database |
| Integration | HttpWebRequest and XML parsing | .NET Framework library | Calls ledger service endpoints |

### Data Storage & External Services

The application stores customer and account data in a SQL Server database using direct ADO.NET commands. It also integrates with an external ledger service over HTTP for balance and transaction history and relies on configured login and logout gateway URLs for authentication flow.

### Key Architectural Decisions

- Uses server side Web Forms event model with code behind orchestration instead of API first controller patterns.
- Uses direct SQL command execution with `SqlConnection`, `SqlCommand`, and `SqlDataAdapter` rather than an ORM.
- Uses synchronous HTTP calls from page code for ledger integration and falls back to database reads when remote transaction retrieval fails.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation
        DefaultPage["Default.aspx"]
        LoginPage["Login.aspx"]
        LogoutPage["Logout.aspx"]
        MasterPage["Site.Master"]
    end

    subgraph Business["Business Logic"]
        DefaultCode["Default.aspx.cs"]
        LoginCode["Login.aspx.cs"]
        LogoutCode["Logout.aspx.cs"]
    end

    subgraph DataAccess["Data Access"]
        SqlClient["SqlClient Commands"]
        ZavaDb[("ZavaBankDB")]
    end

    subgraph Infra["Infrastructure"]
        FormsAuth["Forms Auth Middleware"]
        LedgerClient["Ledger HTTP Client"]
    end

    DefaultPage -->|"events"| DefaultCode
    LoginPage -->|"events"| LoginCode
    LogoutPage -->|"events"| LogoutCode
    MasterPage -->|"layout"| DefaultPage
    DefaultCode -->|"queries and updates"| SqlClient
    SqlClient -->|"database operations"| ZavaDb
    DefaultCode -->|"balance transactions"| LedgerClient
    FormsAuth -.->|"intercepts anonymous access"| DefaultPage
    LoginCode -.->|"sign in flow"| FormsAuth
    LogoutCode -.->|"sign out flow"| FormsAuth
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| Default.aspx | Presentation | Web Forms Page | Main account management UI |
| Login.aspx | Presentation | Web Forms Page | User authentication entry point |
| Logout.aspx | Presentation | Web Forms Page | Sign out flow |
| Site.Master | Presentation | Master Page | Shared page layout and navigation |
| Default.aspx.cs | Business Logic | Code behind class | Coordinates account queries, updates, and external calls |
| Login.aspx.cs | Business Logic | Code behind class | Handles login behavior |
| Logout.aspx.cs | Business Logic | Code behind class | Handles logout behavior |
| SqlClient Commands | Data Access | ADO.NET data access | Executes SQL statements |
| Forms Auth Middleware | Infrastructure | Authentication component | Controls access to authenticated pages |
| Ledger HTTP Client | Infrastructure | External integration client | Retrieves account balance and transaction data |
