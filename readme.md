# Execution-Driven Architecture
> Controlling system behavior from input to output - deterministically.

---

## Overview

Most systems are designed around structure:

- Layers  
- Services  
- Patterns  

This approach focuses on something else:

> Execution flow - how a system actually runs

Instead of reacting to events and chaining handlers, this model defines:
- explicit intent
- deterministic execution
- clear boundaries

---

## Core Idea

Policies decide. Contracts define. Orchestrators execute.

---

## Execution Pipeline

```mermaid
flowchart LR
    A[Controller] --> B[Service]
    B --> C[Policy]
    C --> D[Execution Contract]
    D --> E[Orchestrator]
    E --> F[Database]
```

---

## Why This Matters

Traditional systems often suffer from:

- Event handler sprawl  
- Hidden execution paths  
- Hard-to-trace behavior  
- Coupled services  

This architecture replaces that with:

- Explicit execution flow  
- Deterministic behavior  
- Clear ownership  
- Predictable outcomes  

---

## Traditional vs Execution-Driven

### Traditional (Event-driven sprawl)

```text
Controller -> Handler -> Event -> Handler -> Handler -> ???
```

### Execution-Driven (Deterministic)

```text
Controller -> Service -> Policy -> Contract -> Orchestrator -> DB
```

---

## Contracts (Execution Intent)

Contracts define what must happen:

```csharp
public record AssignUserLicenseInput(...) : IOrchestratorInput;
```

They:

- originate from policies  
- contain all required data  
- drive execution directly  

---

## Orchestrators

Orchestrators are pure execution units.

They:

- execute infrastructure logic  
- coordinate data operations  
- do not contain business rules  

```csharp
public class AssignLicenseOrchestrator 
    : Orchestrator<AssignUserLicenseInput, AssignUserLicenseOutput>
{
    public override async Task<AssignUserLicenseOutput> ExecuteAsync(...)
    {
        // Persistence + execution logic
    }
}
```

---

## Policies

Policies are responsible for:

- evaluating rules  
- producing outcomes  
- generating execution contracts  

They do NOT:

- perform persistence  
- call infrastructure  

---

## Domain Separation

- Domains are isolated  
- No module depends on another module’s internals  
- Cross-domain interaction happens via:
  - infrastructure  
  - orchestration  

---

## Evolution Path

This model naturally evolves:

### Modular Monolith

```text
Policy -> Contract -> Orchestrator
```

### Distributed System

```mermaid
flowchart LR
    A[Policy] --> B[Contract]
    B --> C[Message Bus]
    C --> D[Orchestration Service]
    D --> E[Orchestrator]
    E --> F[Database]
```

Same contracts. Same orchestrators. Different execution transport.

---

## Testing Strategy

### Policy Tests (Unit)
Validate decision logic

### Orchestrator Tests (Integration)
Validate execution behavior

### Workflow Tests (Integration)

```text
Decision -> Execution -> Result
```

---

## Key Principle

The contract contains everything required for execution.

No:

- handler discovery  
- routing logic  
- implicit behavior  

---

## Architectural Inversion

Instead of:

```text
Event -> Handler -> Handler -> Handler
```

You get:

```text
Policy -> Contract -> Orchestrator
```

---

## The Real Problem

The hard part is not splitting systems.  
The hard part is coordinating execution between them.

---

## What This Enables

- Deterministic execution  
- Clear system behavior  
- Reduced complexity  
- Easier debugging  
- Seamless evolution to distributed systems  

---

## Positioning

This approach aligns with modern needs:

- High-throughput systems  
- Multi-context data operations  
- Controlled execution flows  
- Real-world system coordination  

---

## Summary

This architecture shifts focus from structure to execution.

- Policies define intent  
- Contracts carry intent  
- Orchestrators execute intent  

---

# DataArc FAQ

## Why not just use `DbContext`?

Use `DbContext` directly when you are working inside **one persistence boundary** and the workflow is simple.

For example:

```csharp
dbContext.Set<Order>().Add(order);
await dbContext.SaveChangesAsync();
```

DataArc becomes useful when the operation is no longer just a single-context persistence call.

The gap appears when you need to:

- Coordinate multiple `DbContext` instances
- Run ordered execution pipelines
- Perform bulk operations as part of real workflows
- Handle parallel EF Core pipelines safely
- Coordinate transaction behavior
- Produce execution summaries and telemetry
- Avoid service-layer coordination scripts

A `DbContext` knows only itself. DataArc coordinates execution across contexts.

**Summary**

> `DbContext` is excellent for single-context persistence. DataArc is for coordinated execution when one `DbContext` is no longer enough.

---

## Why not repositories?

Repositories are still fine.

DataArc does not require you to abandon repositories. In fact, the clean adoption path is to use DataArc inside repositories:

```csharp
public sealed class OrderRepository : IOrderRepository
{
    private readonly IAsyncCommand _command;
    private readonly IAsyncQuery _query;

    public OrderRepository(IAsyncCommand command, IAsyncQuery query)
    {
        _command = command;
        _query = query;
    }

    public Task CreateAsync(Order order)
    {
        return _command
            .UseCommandContext<OrderDbContext>()
            .Add(order)
            .ExecuteAsync();
    }
}
```

The repository remains the application-facing abstraction. DataArc becomes the execution engine underneath it.

The problem DataArc avoids is not repositories. The problem is **repository coordination sprawl**, where services begin manually coordinating many repositories, contexts, transactions, bulk operations, and side effects.

**Summary**

> Repositories abstract data access. DataArc coordinates data execution. Use repositories for simple boundaries; use DataArc when execution crosses boundaries or needs orchestration.

---

## Why not MediatR?

MediatR is a message dispatcher. It routes requests to handlers.

DataArc is not a replacement for every MediatR use case. The difference is that DataArc focuses on **data execution coordination**, not general request dispatch.

MediatR often leads to a structure like:

- One request
- One handler
- One pipeline
- Many small classes

That can be useful, but it can also create handler sprawl when the real problem is persistence execution.

DataArc gives command/query separation without forcing every operation into a request/handler model:

```csharp
await _command
    .UseCommandContext<OrderDbContext>()
    .Add(order)
    .ExecuteAsync();
```

For larger workflows, DataArc uses orchestrators:

```csharp
await _orchestrator.ExecuteAsync(input);
```

**Summary**

> MediatR coordinates messages between handlers. DataArc coordinates execution across persistence boundaries. DataArc gives CQRS-style command/query execution without forcing request/response handler sprawl.

---

## Why not EF bulk extensions?

EF bulk libraries are useful for fast inserts, updates, deletes, and merges.

DataArc’s purpose is not only to make bulk operations faster. The purpose is to make bulk operations fit into real application workflows.

Most bulk libraries focus on isolated operations such as:

```csharp
await dbContext.BulkInsertAsync(items);
```

That solves a single performance problem.

DataArc is aimed at broader execution coordination:

- Bulk operations inside ordered workflows
- Multi-context execution
- Parallel pipelines
- Transaction coordination
- Execution summaries
- Schema and DDL integration
- Orchestrated infrastructure flows

EF bulk extensions can solve an isolated bulk operation. DataArc solves the broader execution-coordination problem.

**Summary**

> EF bulk libraries optimize individual bulk operations. DataArc integrates bulk operations into coordinated, multi-context execution workflows.

---

## What is an execution context?

An execution context is a persistence boundary that DataArc is allowed to execute through.

With EF Core, that is usually a `DbContext` that also implements DataArc’s execution context contract:

```csharp
public class OrdersDbContext 
    : DbContext, IExecutionContext<OrdersDbContext>
{
    public OrdersDbContext(DbContextOptions<OrdersDbContext> options)
        : base(options)
    {
    }

    public DbSet<Order> Orders { get; set; }
}
```

Then it is registered with DataArc:

```csharp
services
    .AddDataArcCore()
    .ConfigureExecutionContexts(ctx =>
    {
        ctx.AddDbContext<OrdersDbContext>(options =>
            options.UseSqlServer(connectionString));
    });
```

To EF Core, it is a `DbContext`.

To DataArc, it is an execution context.

**Summary**

> An execution context is a registered persistence boundary. In EF Core, it is a `DbContext` that implements `IExecutionContext<TContext>` so DataArc can coordinate execution through it.

---

## When do I need an orchestrator?

Use normal repositories or command/query builders for simple CRUD.

Use an orchestrator when a workflow becomes bigger than one repository, one context, or one simple persistence operation.

An orchestrator is useful when you need to:

- Coordinate multiple `DbContext` instances
- Sequence multiple operations as one workflow
- Combine inserts, updates, bulk operations, and queries
- Coordinate transaction behavior
- Share execution state between steps
- Produce one execution result or summary
- Keep services from becoming coordination scripts

Example orchestrator scenario:

```text
1. Seed parent data in Context A
2. Bulk insert children in Context B
3. Run validation query in Context C
4. Commit or fail the workflow as one coordinated execution path
```

That is orchestrator territory.

**Summary**

> Use an orchestrator when a workflow spans multiple persistence boundaries or requires ordered, coordinated execution. Repositories handle access; orchestrators coordinate execution.

## Final Thought

Most architectures define structure.  
Very few control execution.

This one does.
