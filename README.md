# Devel Frame

Framework de desenvolvimento de software com melhores práticas de engenharia.

## Documentação

### Guias Principais

| Arquivo | Descrição |
|---------|-----------|
| [AGENTS.md](./AGENTS.md) | Standards e diretrizes para agents de código |
| [tdd.md](./tdd.md) | Test-Driven Development (Red-Green-Refactor) |
| [12-factors.md](./12-Factors.md) | Metodologia Cloud Native (12 fatores) |

### Qualidade & Segurança

| Arquivo | Descrição |
|---------|-----------|
| [security-ai-code.md](./security-ai-code.md) | Segurança para código gerado por IA |
| [architecture-reliability.md](./architecture-reliability.md) | Circuit breakers, fallbacks, testes de falha |

### Performance

| Arquivo | Descrição |
|---------|-----------|
| [n1-query-detector.md](./n1-query-detector.md) | Detecção de queries N+1 |
| [race-condition-detection.md](./race-condition-detection.md) | Detecção de race conditions (fast-check) |
| [memory-leak-prevention.md](./memory-leak-prevention.md) | Prevenção de memory leaks |

## Tech Stack

- Node.js/TypeScript
- Python
- Rust
- Go

## Quick Start

```bash
# Install dependencies
npm install

# Build
npm run build

# Lint
npm run lint

# Test
npm test
```

## Estrutura

```
.
├── AGENTS.md                    # Guidelines para AI agents
├── tdd.md                       # TDD guide
├── 12-factors.md               # 12-Factor App
├── security-ai-code.md         # Security
├── architecture-reliability.md # Reliability
├── n1-query-detector.md        # N+1 queries
├── race-condition-detection.md # Race conditions
└── memory-leak-prevention.md   # Memory leaks
```

## Referências

Baseado em conteúdo do vídeo: [vai ter apagão de DEV em 2027 - Lucas Montano](https://youtu.be/T9V7EyB_B9w)