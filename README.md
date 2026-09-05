# Clean Architecture Template

[![Build](https://github.com/evangelosvlachos96-dotcom/clean-architecture-template/actions/workflows/build.yml/badge.svg)](https://github.com/evangelosvlachos96-dotcom/clean-architecture-template/actions/workflows/build.yml)
[![CodeQL](https://github.com/evangelosvlachos96-dotcom/clean-architecture-template/actions/workflows/codeql.yml/badge.svg)](https://github.com/evangelosvlachos96-dotcom/clean-architecture-template/actions/workflows/codeql.yml)

> A `dotnet new` solution template for enterprise applications built on Clean Architecture, ASP.NET Core 10 and .NET Aspire, with an Angular, React or Web API-only front end and a choice of SQLite, PostgreSQL or SQL Server.

Developed by **Evangelos Vlachos**.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Solution Structure](#solution-structure)
- [Technologies](#technologies)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Install the template](#install-the-template)
  - [Create a new solution](#create-a-new-solution)
  - [Run the app](#run-the-app)
  - [Test](#test)
- [Scaffolding Use Cases](#scaffolding-use-cases)
- [Working on the Template Itself](#working-on-the-template-itself)
- [Architectural Decisions](#architectural-decisions)

## Overview

The template gives you a complete, runnable solution in one command: a layered back end following Clean Architecture, an optional SPA front end, an Aspire AppHost that orchestrates everything locally, and unit, integration, functional and acceptance test projects already wired up. It is meant as a starting point you own, not a framework you depend on.

## Architecture

Dependencies point inward. The Domain has no dependencies. Application depends on Domain. Infrastructure and Web depend on Application. The AppHost composes the runnable pieces.

| Layer | Project | Contents |
|---|---|---|
| **Domain** | `src/Domain` | Entities, value objects, enums, domain events, exceptions. No external dependencies. |
| **Application** | `src/Application` | Use cases as MediatR commands and queries, validators, DTOs, pipeline behaviours, interfaces for infrastructure. |
| **Infrastructure** | `src/Infrastructure` | EF Core DbContext and migrations, identity, external service implementations. |
| **Web** | `src/Web` | ASP.NET Core minimal API endpoints, OpenAPI via Scalar, and the Angular or React client app. |
| **AppHost** | `src/AppHost` | .NET Aspire orchestration: database container, Web project, dashboard. |
| **ServiceDefaults** | `src/ServiceDefaults` | Shared Aspire defaults: health checks, OpenTelemetry, resilience. |
| **Shared** | `src/Shared` | Small helpers shared across projects. |

Cross-cutting behaviours in the Application layer handle validation, authorization, logging, performance tracking and unhandled exceptions for every request.

## Solution Structure

```
.
├── src
│   ├── AppHost/               # Aspire AppHost
│   ├── Application/           # Use cases (CQRS with MediatR)
│   ├── Domain/                # Core business model
│   ├── Infrastructure/        # EF Core, identity, external services
│   ├── ServiceDefaults/       # Aspire service defaults
│   ├── Shared/
│   └── Web/                   # Minimal API + ClientApp (Angular) / ClientApp-React
├── tests
│   ├── Application.FunctionalTests/
│   ├── Application.UnitTests/
│   ├── Domain.UnitTests/
│   ├── Infrastructure.IntegrationTests/
│   ├── TestAppHost/
│   └── Web.AcceptanceTests/   # Playwright
├── templates/ca-use-case/     # `dotnet new ca-usecase` item template
├── docs/decisions/            # Architecture Decision Records
├── build/                     # Local repack script
├── .template.config/          # `dotnet new ca-sln` template definition
├── CleanArchitecture.nuspec   # NuGet package definition for the template
└── CleanArchitecture.slnx
```

## Technologies

- [ASP.NET Core 10](https://docs.microsoft.com/en-us/aspnet/core/introduction-to-aspnet-core) and [.NET Aspire](https://aspire.dev)
- [Entity Framework Core 10](https://docs.microsoft.com/en-us/ef/core/) with SQLite, PostgreSQL or SQL Server
- [Angular 21](https://angular.dev/) or [React 19](https://react.dev/)
- [MediatR](https://github.com/jbogard/MediatR), [AutoMapper](https://automapper.org/), [FluentValidation](https://fluentvalidation.net/)
- [NUnit](https://nunit.org/), [Shouldly](https://docs.shouldly.org/), [Moq](https://github.com/devlooped/moq), [Respawn](https://github.com/jbogard/Respawn), [Playwright](https://playwright.dev/)
- [Scalar](https://scalar.com/) for OpenAPI documentation

## Getting Started

### Prerequisites

- [.NET 10.0 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) or later
- [Node.js](https://nodejs.org/) LTS, only if you use the Angular or React front end
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or [Podman](https://podman.io/), only if you use PostgreSQL or SQL Server. SQLite, the default, needs no container.

### Install the template

From the NuGet package, once you have published it:

```bash
dotnet new install CleanArchitecture.Template
```

Or straight from a local clone of this repository:

```bash
dotnet new install ./
```

### Create a new solution

Pick the client framework with `--client-framework` (`-cf`) and the database with `--database` (`-db`):

```bash
dotnet new ca-sln --client-framework [angular|react|none] --database [postgresql|sqlite|sqlserver] --output YourProjectName
```

| Option | Values | Default |
|---|---|---|
| `--client-framework` | `angular`, `react`, `none` | `angular` |
| `--database` | `postgresql`, `sqlite`, `sqlserver` | `sqlite` |

Examples:

```bash
# Angular SPA + Web API + PostgreSQL
dotnet new ca-sln -cf angular -db postgresql -o YourProjectName

# React SPA + Web API + SQL Server
dotnet new ca-sln -cf react -db sqlserver -o YourProjectName

# Web API only + SQLite
dotnet new ca-sln -cf none -db sqlite -o YourProjectName
```

Run `dotnet new ca-sln --help` to see every option.

### Run the app

```bash
dotnet run --project .\src\AppHost
```

The Aspire dashboard opens automatically and shows the application URLs, logs and traces.

### Test

```bash
dotnet test
```

Functional and integration tests spin up their own database through Aspire's `TestAppHost`. Acceptance tests use Playwright and need browsers installed with `pwsh bin/Debug/net10.0/playwright.ps1 install` from the `Web.AcceptanceTests` folder.

## Scaffolding Use Cases

The solution ships with an item template for new commands and queries. Run it from `src/Application`:

```bash
# New command
dotnet new ca-usecase --name CreateTodoList --feature-name TodoLists --usecase-type command --return-type int

# New query
dotnet new ca-usecase -n GetTodos -fn TodoLists -ut query -rt TodosVm
```

If `ca-usecase` is not found, install the template as described above.

## Working on the Template Itself

To test template changes locally, pack and install it in one step:

```powershell
.\build\repack.ps1
```

The script builds the NuGet package from the nuspec into `artifacts`, uninstalls any previous version and installs the new one. `.template.config/template.json` defines the `ca-sln` template and its options. The GitHub Actions workflows build the solution, run CodeQL, exercise every client and database combination, and publish the package on release when a `NUGET_API_KEY` secret is configured.

## Architectural Decisions

Key design decisions are documented as [Architecture Decision Records](docs/decisions/):

- [ADR-001: Use EF Core in the Application layer](docs/decisions/ADR-001-Use-EFCore-In-Application-Layer.md)
- [ADR-002: Aspire for orchestration and testing](docs/decisions/ADR-002-Aspire-For-Orchestration-And-Testing.md)
- [ADR-003: MediatR contracts in the Domain](docs/decisions/ADR-003-MediatR-Contracts-In-Domain.md)
