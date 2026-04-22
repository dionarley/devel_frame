# N+1 Query Detector

O problema N+1 ocorre quando você faz uma query para buscar uma lista de registros e depois faz uma query adicional para cada registro para buscar seus relacionamentos. Por exemplo, buscar 100 usuários e depois 100 queries adicionais para buscar os posts de cada um = 101 queries ao banco.

## Implementação

### Node.js - Middleware de Contagem

```typescript
import { Pool } from 'pg'
import { PrismaClient } from '@prisma/client'

const queryCount = new Map<string, number>()

export function createQueryCounter() {
  return (req: Request, res: Response, next: NextFunction) => {
    const start = Date.now()
    const originalQuery = Pool.prototype.query
    
    Pool.prototype.query = function(...args) {
      const count = queryCount.get(req.path) || 0
      queryCount.set(req.path, count + 1)
      return originalQuery.apply(this, args)
    }
    
    res.on('finish', () => {
      const total = queryCount.get(req.path) || 0
      if (total > 10) {
        console.warn(`[N+1 ALERT] ${req.path}: ${total} queries`)
      }
      queryCount.delete(req.path)
    })
    
    next()
  }
}
```

### Python - SQLAlchemy Event Hook

```python
from sqlalchemy import event
from sqlalchemy.engine import Engine
from collections import Counter
import re

class QueryCounter:
    def __init__(self):
        self.queries = []
        self._listening = False
    
    def start(self, engine: Engine):
        @event.listens_for(engine, "before_cursor_execute")
        def receive_before_cursor_execute(conn, cursor, statement, parameters, context, executemany):
            if self._listening:
                self.queries.append(statement)
        self._listening = True
    
    def detect_n_plus_one(self, threshold=5):
        normalized = []
        for q in self.queries:
            sql = re.sub(r"'[^']*'", "?", q)
            sql = re.sub(r"= \d+", "= ?", sql)
            normalized.append(sql)
        
        counts = Counter(normalized)
        return {sql: count for sql, count in counts.items() if count >= threshold}
    
    def reset(self):
        self.queries = []

# Uso em testes
def test_n_plus_one(query_counter, db_session):
    query_counter.start(engine)
    users = db_session.query(User).all()
    for user in users:
        _ = user.posts  # Acessa relacionamento
    
    violations = query_counter.detect_n_plus_one(threshold=3)
    assert len(violations) == 0, f"N+1 detected: {violations}"
    query_counter.reset()
```

### Ferramentas Prontas

| Linguagem | Ferramenta | Descrição |
|-----------|------------|-----------|
| Python | `nplusone` | Detecta lazy loading automático |
| Node.js | `DexterJS` | Observabilidade com detecção N+1 |
| Python | `fastapi-sqlalchemy-monitor` | Middleware para FastAPI |
| Ruby | `bullet` | Detecção N+1 para Rails |
| Go | `ent` | ORM com detecção integrada |

## Boas Práticas

- **Eager Loading**: Use `joinedload()` ou `includes()` para carregar relacionamentos de uma vez
- **Threshold**: Defina um limite (ex: 10 queries por request) e alerte acima disso
- **Monitoramento**: Adicione em staging/canary para detectar padrões reais de uso
- **Testes**: Crie testes que falham se N+1 for detectado

## Exemplo com Prisma

```typescript
// Problema N+1
const users = await prisma.user.findMany()
for (const user of users) {
  const posts = await prisma.post.findMany({ where: { userId: user.id } })
}

// Solução com eager loading
const users = await prisma.user.findMany({
  include: { posts: true }
})
```