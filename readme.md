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

> Policies decide. Contracts define. Orchestrators execute.

---

## Execution Pipeline

```mermaid
flowchart LR
    A[Controller] --> B[Service]
    B --> C[Policy]
    C --> D[Execution Contract]
    D --> E[Orchestrator]
    E --> F[Database]
