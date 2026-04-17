# SOLID Principles for TypeScript Implementation

Use SOLID to keep TypeScript code modular, testable, and easy to extend.
Treat each principle as a concrete implementation rule during code generation and review.

## Single Responsibility Principle (SRP)

Each class or module should have one reason to change.
In TypeScript, this usually means one orchestrator plus focused collaborators.

```typescript
interface PlaybackEvent {
  readonly creativeId: string;
  readonly playedAt: string;
}

interface PlaybackBatch {
  readonly deviceId: string;
  readonly events: ReadonlyArray<PlaybackEvent>;
}

interface BatchStore {
  get(key: string): Promise<PlaybackBatch | null>;
  set(key: string, batch: PlaybackBatch): Promise<void>;
}

interface BatchReporter {
  report(batch: PlaybackBatch): Promise<boolean>;
}

class PlaybackBatchService {
  constructor(
    private readonly store: BatchStore,
    private readonly reporter: BatchReporter,
    private readonly maxBatchSize: number
  ) {}

  async append(deviceId: string, event: PlaybackEvent): Promise<void> {
    const key = this.makeStorageKey(deviceId);
    const current = (await this.store.get(key)) ?? this.makeEmptyBatch(deviceId);
    const updated = this.addEvent(current, event);
    const maybeFlushed = await this.flushIfNeeded(updated);
    await this.store.set(key, maybeFlushed);
  }

  private makeStorageKey(deviceId: string): string {
    return `playback:${deviceId}`;
  }

  private makeEmptyBatch(deviceId: string): PlaybackBatch {
    return { deviceId, events: [] };
  }

  private addEvent(batch: PlaybackBatch, event: PlaybackEvent): PlaybackBatch {
    return { ...batch, events: [...batch.events, event] };
  }

  private async flushIfNeeded(batch: PlaybackBatch): Promise<PlaybackBatch> {
    if (batch.events.length < this.maxBatchSize) return batch;
    const reported = await this.reporter.report(batch);
    return reported ? { ...batch, events: [] } : batch;
  }
}
```

## Open/Closed Principle (OCP)

Extend behavior by adding implementations, not by editing a central conditional.
Use registries and typed strategies.

```typescript
type TransportKind = "http" | "udp";

interface BatchTransport {
  readonly kind: TransportKind;
  send(batch: PlaybackBatch): Promise<void>;
}

class TransportRouter {
  constructor(
    private readonly transports: ReadonlyMap<TransportKind, BatchTransport>
  ) {}

  async dispatch(kind: TransportKind, batch: PlaybackBatch): Promise<void> {
    const transport = this.transports.get(kind);
    if (!transport) throw new Error(`Unsupported transport: ${kind}`);
    await transport.send(batch);
  }
}

// Add new behavior by registering another implementation.
const router = new TransportRouter(
  new Map<TransportKind, BatchTransport>([
    ["http", new HttpTransport(httpClient)],
    ["udp", new UdpTransport(udpSocket)],
  ])
);
```

## Liskov Substitution Principle (LSP)

Any implementation of an interface should preserve its contract.
If the interface returns `Result`, implementations should not throw for expected failures.

```typescript
type ReportResult = { success: true } | { success: false; reason: string };

interface SafeBatchReporter {
  report(batch: PlaybackBatch): Promise<ReportResult>;
}

// WRONG: Breaks substitutability by throwing for an expected failure.
class ThrowingReporter implements SafeBatchReporter {
  async report(batch: PlaybackBatch): Promise<ReportResult> {
    if (batch.events.length === 0) throw new Error("Batch is empty");
    return { success: true };
  }
}

// CORRECT: Preserves the contract.
class ContractSafeReporter implements SafeBatchReporter {
  async report(batch: PlaybackBatch): Promise<ReportResult> {
    if (batch.events.length === 0) {
      return { success: false, reason: "Batch is empty" };
    }
    return { success: true };
  }
}
```

## Interface Segregation Principle (ISP)

Prefer small capability interfaces over large "do everything" interfaces.

```typescript
// WRONG: Consumers are forced to depend on methods they do not need.
interface MediaGateway {
  fetchMedia(id: string): Promise<MediaAsset>;
  reportPlayback(batch: PlaybackBatch): Promise<void>;
  syncConfiguration(): Promise<void>;
}

// CORRECT: Split by capability.
interface MediaReader {
  fetchMedia(id: string): Promise<MediaAsset>;
}

interface PlaybackReporter {
  reportPlayback(batch: PlaybackBatch): Promise<void>;
}

interface ConfigSync {
  syncConfiguration(): Promise<void>;
}

class PlaybackOnlyReporter implements PlaybackReporter {
  async reportPlayback(batch: PlaybackBatch): Promise<void> {
    // Implementation
  }
}
```

## Dependency Inversion Principle (DIP)

Depend on abstractions in core logic.
Create concrete implementations at the composition root.

```typescript
interface HttpClient {
  post(url: string, body: unknown): Promise<void>;
}

class HttpBatchReporter implements BatchReporter {
  constructor(
    private readonly http: HttpClient,
    private readonly endpoint: string
  ) {}

  async report(batch: PlaybackBatch): Promise<boolean> {
    try {
      await this.http.post(this.endpoint, batch);
      return true;
    } catch {
      return false;
    }
  }
}

// Composition root.
const store: BatchStore = new IndexedDbBatchStore(indexedDbClient);
const reporter: BatchReporter = new HttpBatchReporter(httpClient, "/pop/report");
const service = new PlaybackBatchService(store, reporter, 100);
```

## SOLID Review Checklist

1. Does each class/module have a single reason to change?
2. Can new behavior be added without editing an existing central switch/if chain?
3. Are interface implementations preserving expected contracts (no surprise throws)?
4. Are interfaces small and capability-based?
5. Are core services depending on interfaces, with wiring done at composition boundaries?
