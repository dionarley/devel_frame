# Memory Leak Prevention

Memory leaks ocorrem quando objetos não são liberados da memória mesmo quando não são mais necessários. Em produção, podem degradar lentamente a performance até causar崩溃 (OOM).

## Detecção em Node.js

### 1. process.memoryUsage()

```javascript
// Monitoramento básico
setInterval(() => {
  const mem = process.memoryUsage();
  console.log({
    heapUsed: Math.round(mem.heapUsed / 1024 / 1024) + ' MB',
    heapTotal: Math.round(mem.heapTotal / 1024 / 1024) + ' MB',
    rss: Math.round(mem.rss / 1024 / 1024) + ' MB',
    external: Math.round(mem.external / 1024 / 1024) + ' MB'
  });
}, 30000);
```

### 2. Heap Snapshots com Chrome DevTools

**Técnica dos 3 Snapshots:**
1. Tire um snapshot inicial
2. Execute a ação suspeito
3. Tire outro snapshot
4. Repita a ação e tire o terceiro
5. Compare os snapshots - objetos que crescem são vazamentos

### 3. heapdump (Produção)

```bash
npm install heapdump
```

```javascript
const heapdump = require('heapdump');

app.get('/heapdump', (req, res) => {
  heapdump.writeSnapshot('/tmp/' + Date.now() + '.heapsnapshot', (err, filename) => {
    if (err) return res.status(500).send(err.message);
    res.send('Snapshot: ' + filename);
  });
});
```

### 4. clinic.js

```bash
npm install -g clinic
clinic heaprofiler -- node server.js
```

Gera relatório HTML com alocações de heap ao longo do tempo.

## Padrões Comuns de Vazamento

### 1. Event Listeners Esquecidos

```javascript
// PROBLEMA - listener nunca removido
server.on('request', (req) => {
  db.query(req.body.id); // fecha over req
});

// SOLUÇÃO - usar weakMap ou remover explicitamente
const requestHandlers = new WeakMap();
server.on('request', handler);
server.off('request', handler);
```

### 2. Closures que prendem objetos

```javascript
// PROBLEMA
app.post('/watch', (req, res) => {
  const interval = setInterval(() => {
    checkCondition(req.body.id);  // prende req.body para sempre
  }, 1000);
  res.json({ intervalId: interval });
});

// SOLUÇÃO
const watchers = new Map();
app.post('/watch', (req, res) => {
  const { id } = req.body;
  if (watchers.has(id)) clearInterval(watchers.get(id));
  const interval = setInterval(() => checkCondition(id), 1000);
  watchers.set(id, interval);
  res.json({ intervalId: id });
});

app.delete('/watch/:id', (req, res) => {
  const { id } = req.params;
  if (watchers.has(id)) {
    clearInterval(watchers.get(id));
    watchers.delete(id);
  }
  res.json({ success: true });
});
```

### 3. Singletons com estado mutável

```javascript
// PROBLEMA - array cresce infinitamente
class Cache {
  static instance = null;
  data = [];
  
  static get() {
    if (!this.instance) this.instance = new Cache();
    return this.instance;
  }
  
  add(item) {
    this.data.push(item);  // nunca limpa
  }
}

// SOLUÇÃO - limites eTTL
class Cache {
  data = new Map();
  maxSize = 1000;
  
  add(key, value) {
    if (this.data.size >= this.maxSize) {
      const first = this.data.keys().next().value;
      this.data.delete(first);
    }
    this.data.set(key, { value, timestamp: Date.now() });
  }
}
```

## Python - tracemalloc e memory_profiler

```python
import tracemalloc

tracemalloc.start()

# Seu código aqui
result = expensive_operation()

current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024:.1f} MB")
print(f"Peak: {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```

```bash
pip install memory_profiler
# Execute com: python -m memory_profiler script.py
```

## Java - VisualVM e jmap

```bash
# Heap dump em produção
jmap -dump:format=b,file=heap.bin <pid>

# Analisar com VisualVM ou MAT (Eclipse Memory Analyzer)
```

## Ferramentas por Linguagem

| Linguagem | Ferramenta | Uso |
|-----------|------------|-----|
| Node.js | Chrome DevTools | Heap snapshots |
| Node.js | clinic.js | Profiling automatizado |
| Node.js | heapdump | Dump programático |
| Python | tracemalloc | Rastreamento alocações |
| Python | memory_profiler | Linha a linha |
| Java | VisualVM | Profiling visual |
| Java | jmap | Heap dumps |
| Go | pprof | Análise de heap |

## Boas Práticas

- **Limite de cache**: Sempre tenha tamanho máximo
- **WeakMap/WeakSet**: Use para referências que podem ser coletadas
- **Cleanup em testes**: Verifique memória antes/depois de testes
- **Monitoramento produção**: Alertas quando memória excede threshold
- **Timeouts**: Configure para operações de longa duração