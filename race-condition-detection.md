# Race Condition Detection

Race conditions ocorrem quando o comportamento do sistema depende da sequência ou tempo de eventos não controlados. Em código assíncrono, são particularmente difíceis de detectar porque podem acontecer de forma imprevisível.

## Property-Based Testing com fast-check

O **fast-check** é uma biblioteca JavaScript/TypeScript que permite detectar race conditions usando um `scheduler` que controla a ordem de execução de tarefas assíncronas.

### Instalação

```bash
npm install --save-dev fast-check
# ou
yarn add -D fast-check
```

### Exemplo Básico

```typescript
import fc from 'fast-check';

test('should resolve promises in order', async () => {
  await fc.assert(
    fc.asyncProperty(fc.scheduler(), async (s) => {
      const results: number[] = [];
      
      const promise1 = s.schedule(Promise.resolve(1));
      const promise2 = s.schedule(Promise.resolve(2));
      const promise3 = s.schedule(Promise.resolve(3));
      
      s.scheduleWait(promise1).then(() => results.push(1));
      s.scheduleWait(promise2).then(() => results.push(2));
      s.scheduleWait(promise3).then(() => results.push(3));
      
      await s.waitAll();
      
      expect(results).toEqual([1, 2, 3]);
    })
  );
});
```

### Scheduler API

| Método | Descrição |
|--------|-----------|
| `schedule(promise)` | Agenda uma promise para execução |
| `scheduleWait(promise)` | Espera a promise resolver |
| `scheduleSequence(tasks)` | Executa uma sequência de tarefas |
| `waitAll()` | Espera todas as tarefas agendadas |
| `waitOne()` | Espera uma tarefa específica |
| `waitFor(promise)` | Espera por uma task não agendada |

### Exemplo com API Real

```typescript
import fc from 'fast-check';
import supertest from 'supertest';
import { prismaMock } from './mocks/prisma.mock';

const request = supertest(app);

test('should handle concurrent booking requests', async () => {
  await fc.assert(
    fc.asyncProperty(fc.scheduler(), async (s) => {
      const reservation = {
        id: 'randomId',
        slotId: 'slot123',
        tableId: 'table456',
        booked: false,
      };

      // Agenda as chamadas à API
      prismaMock.reservation.findFirst.mockImplementation(
        s.scheduleFunction(async () => reservation) as any
      );

      const req1 = request.post('/reservation').send({ slotId: 'slot123' });
      const req2 = request.post('/reservation').send({ slotId: 'slot123' });

      s.scheduleWait(req1);
      s.scheduleWait(req2);

      await s.waitAll();

      // Verifica que apenas uma reserva foi criada
      const bookings = await prismaMock.reservation.createMany;
      expect(bookings).toHaveBeenCalledTimes(1);
    })
  );
});
```

## Python com Hypothesis

### Instalação

```bash
pip install hypothesis
```

### Exemplo de Race Condition

```python
from hypothesis import given, settings, Phase
import asyncio

@given(scheduler=st.schedules())
@settings(max_examples=1000)
async def test_concurrent_writes(scheduler):
    """Testa escrita concorrente para detectar race conditions"""
    results = []
    lock = asyncio.Lock()
    
    async def write_value(value):
        async with lock:
            results.append(value)
        await asyncio.sleep(0.001)  # Simula processamento
    
    # Executa tarefas concorrentes
    tasks = [write_value(i) for i in range(10)]
    await asyncio.gather(*tasks)
    
    # Verifica ordem de execução
    assert len(results) == 10
```

## Estratégias de Detecção

### 1. Scheduler Arbitrário (fast-check)
- Controla explicitamente a ordem de execução
- Gera milhares de cenários diferentes
- Detecta ordens de resolução inesperadas

### 2. Model-Based Testing
- Compara comportamento real vs modelo esperado
- Útil para sistemas complexos com estados
- Combina com `scheduledModelRun`

### 3. Teste de Carga
- Simula múltiplas requisições simultâneas
- Use ferramentas como k6, Gatling, wrk
- Monitore resultados inconsistentes

## Boas Práticas

- **Use locks**: Proteja seções críticas com mutex/semáforos
- **Transações**: Use transações de banco para operações atômicas
- **Idempotência**: Garanta que operações podem ser repetidas com segurança
- **Testes automatizados**: Adicione PBT no CI/CD

## Ferramentas por Linguagem

| Linguagem | Ferramenta | Uso |
|-----------|------------|-----|
| JavaScript | fast-check | Scheduler para async |
| Python | hypothesis | Property-based testing |
| Rust | proptest | Property-based testing |
| Go | testing/quick | Property-based testing |
| Elixir | StreamData | Property-based testing |