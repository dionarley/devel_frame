# AGENTS.md - Coding Standards & Guidelines

## Build, Lint & Test Commands

### General Commands
```bash
# Build the project
npm run build          # Node/TypeScript
cargo build          # Rust
go build ./...       # Go
mvn package         # Java

# Run linters
npm run lint         # ESLint
cargo clippy        # Rust
golangci-lint run   # Go
mvn checkstyle:check # Java

# Type checking
npm run typecheck   # TypeScript
cargo check         # Rust
go vet ./...        # Go

# Run all tests
npm test            # Node
cargo test         # Rust
go test ./...      # Go
mvn test           # Java

# Run single test - most common patterns
npm test -- --testNamePattern="specific_test_name"
npm test -- -t "specific_test_name"
npm test --filter="TestName"
cargo test specific_test_name
go test -run TestName ./...
pytest -k "test_name"
```

### Single Test Patterns by Framework
```bash
# Jest
npm test -- -t "test name"

# Vitest
vitest run -t "test name"

# RSpec (Ruby)
bundle exec rspec spec/path/to/spec.rb -e "test name"

# Pytest
pytest tests/test_file.py::test_function_name

# Cargo
cargo test test_function_name
```

## Code Style Guidelines

### General Principles
- Keep functions small and focused (single responsibility)
- Maximum function length: 50 lines
- Maximum file length: 300 lines
- Use meaningful variable names (no single letters except loop indices)
- Avoid magic numbers - use named constants
- Use early returns to reduce nesting

### Imports & Organization
```typescript
// Order: external → internal → relative
import React from 'react'           // external
import { Button } from '@/components' // internal alias
import { useAuth } from './hooks'     // relative

// Group by type with blank lines between
import { useState, useEffect } from 'react'
import { z } from 'zod'
import { ApiClient } from './api'
import { Button, Card } from '@/components'
```

### Formatting
- Use 2-space indentation (or 4-space per team preference)
- Maximum line length: 100 characters
- Trailing commas in multiline objects/arrays
- Prefer object shorthand when possible
- Use template literals over concatenation

### TypeScript/JavaScript Types
- Never use `any` - always type explicitly
- Use strict TypeScript: `"strict": true` in tsconfig
- Prefer interfaces over types for objects
- Use union types for finite states
- Generic types for reusable components
- Explicit return types for public APIs

```typescript
// Good
interface User {
  id: string
  name: string
  email: string
}

type UserStatus = 'active' | 'inactive' | 'pending'

function getUser(id: string): Promise<User> {
  // ...
}

// Avoid
function bad(user: any): any { /* ... */ }
```

### Naming Conventions
- **Files**: kebab-case (`user-service.ts`) or PascalCase (`UserService.ts`)
- **Functions**: camelCase, verb-prefixed (`getUserById`, `calculateTotal`)
- **Classes/Interfaces**: PascalCase (`UserService`, `ApiClient`)
- **Constants**: SCREAMING_SNAKE_CASE
- **Booleans**: isPrefix, hasPrefix, shouldPrefix (`isActive`, `hasPermission`)
- **Enums**: PascalCase members (`UserRole.Admin`)

### Error Handling
- Never silently catch errors
- Use custom error types for domain errors
- Include context in error messages
- Log errors at the appropriate boundary (API, worker)
- Never expose internal error details to clients

```typescript
// Good
class NotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`)
    this.name = 'NotFoundError'
  }
}

// Always handle async errors
async function fetchUser(id: string) {
  try {
    return await api.get(`/users/${id}`)
  } catch (error) {
    if (error instanceof NotFoundError) {
      throw error
    }
    logger.error('Failed to fetch user', { id, error })
    throw new ApiError('Failed to fetch user')
  }
}
```

## Security Guidelines

### Secrets Management
- NEVER commit secrets, keys, or credentials
- Use environment variables for secrets
- Use secrets management services (AWS Secrets Manager, Vault)
- Scan for leaked secrets in PRs
- Pin dependencies with exact versions

### Input Validation
- Validate all inputs at API boundaries
- Sanitize user inputs before use in queries
- Use parameterized queries to prevent SQL injection
- Encode output to prevent XSS

### AI-Generated Code
- Sempre faça review de código gerado por IA
- Execute scanners de segurança em CI/CD
- Ratreabilidade: identifique código gerado por IA
-**Ver**: [security-ai-code.md](./security-ai-code.md)

## Performance & Architecture

### N+1 Query Detection
- Evite N+1 queries - use eager loading (joinedload, include)
- Implemente middleware para monitorar número de queries por requisição
- Configure threshold (ex: 10 queries) e alerte se exceder
- Use ferramentas como nplusone (Python), DexterJS (Node.js)
- **Ver**: [n1-query-detector.md](./n1-query-detector.md)

### Race Conditions
- Use proper locking mechanisms (mutex, semaphores)
- Implemente idempotency keys para operações críticas
- Use database transactions para operações multi-step
- Use property-based testing (fast-check, Hypothesis) para detectar race conditions
- **Ver**: [race-condition-detection.md](./race-condition-detection.md)

### Memory Management
- Limpe recursos (close connections, streams)
- Evite memory leaks em processos de longa duração
- Use WeakMap/WeakSet para referências que podem ser coletadas
- Use profiling tools (clinic.js, pprof, Chrome heap snapshot)
- **Ver**: [memory-leak-prevention.md](./memory-leak-prevention.md)

### Reliability
- Implemente circuit breakers para serviços externos
- Adicione timeouts para todas as chamadas externas
- Handle graceful degradation
- Planeje para queda de banco (cache, write queue, read replica)
- Implemente retry com exponential backoff + jitter
- Adicione health checks para dependências
- **Ver**: [architecture-reliability.md](./architecture-reliability.md)

## Testing Guidelines

### Test Structure
- One describe block per function/component
- Clear test descriptions: "should [expected behavior]"
- AAA pattern: Arrange, Act, Assert

### TDD - Red Green Refactor
- **RED**: Write failing test FIRST before code exists
- **GREEN**: Write minimum code to pass the test
- **REFACTOR**: Clean up without changing behavior
- NEVER write code without a failing test
- **Ver**: [tdd.md](./tdd.md)

### Test Coverage
- Aim for 80%+ coverage on business logic
- Test happy path and error paths
- Test edge cases and boundary values
- Mock external dependencies

### 12-Factor App (Cloud Native)
- One codebase, many deploys
- Dependencies explicit (package.json, not implicit)
- Config in environment, NOT in code
- Backing services as attached resources
- Build/Release/Run strictly separate
- Stateless processes
- Port binding (self-contained)
- Concurrency (scale out)
- Disposability (fast startup/shutdown)
- Dev/Prod parity (Docker)
- Logs to stdout
- **Ver**: [12-factors.md](./12-factors.md)

## Code Review Checklist

- [ ] No debug statements (console.log, print)
- [ ] No commented-out code
- [ ] Error handling present
- [ ] Types are explicit
- [ ] Naming is clear and consistent
- [ ] Functions are small and focused
- [ ] Tests cover the changes
- [ ] No security issues introduced