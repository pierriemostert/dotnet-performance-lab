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

Controller -> Service -> Policy -> Execution Contract -> Orchestrator -> Database

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

Traditional (Event-driven sprawl)

Controller -> Handler -> Event -> Handler -> Handler -> ???

Execution-Driven (Deterministic)

Controller -> Service -> Policy -> Contract -> Orchestrator -> DB

---

## Contracts (Execution Intent)

Contracts define what must happen.

Example:

public record AssignUserLicenseInput(...) : IOrchestratorInput;

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

Example:

public class AssignLicenseOrchestrator 
    : Orchestrator<AssignUserLicenseInput, AssignUserLicenseOutput>
{
    public override async Task<AssignUserLicenseOutput> ExecuteAsync(...)
    {
        // Persistence + execution logic
    }
}

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

Modular Monolith

Policy -> Contract -> Orchestrator

Distributed System

Policy -> Contract -> Message Bus -> Orchestration Service -> Orchestrator -> Database

Same contracts. Same orchestrators. Different execution transport.

---

## Testing Strategy

Policy Tests (Unit)
- Validate decision logic

Orchestrator Tests (Integration)
- Validate execution behavior

Workflow Tests (Integration)

Decision -> Execution -> Result

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

Event -> Handler -> Handler -> Handler

You get:

Policy -> Contract -> Orchestrator

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

## Final Thought

Most architectures define structure.  
Very few control execution.

This one does.
