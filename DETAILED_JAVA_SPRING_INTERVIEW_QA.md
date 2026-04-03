# Detailed Java + Spring Boot Interview Q&A (with Code)

This document provides detailed interview-ready answers with practical code snippets.

---

## 1) What is the Circuit Breaker pattern and how is it implemented?

A Circuit Breaker protects a service from repeatedly calling a failing downstream dependency.

**States**
- **CLOSED**: calls flow normally.
- **OPEN**: calls are blocked immediately (fast fail).
- **HALF_OPEN**: limited trial calls allowed; success closes breaker, failures reopen it.

**Spring Boot implementation (Resilience4j)**

```java
@RestController
@RequiredArgsConstructor
public class PaymentController {
    private final PaymentClient paymentClient;

    @GetMapping("/pay/{id}")
    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
    public String pay(@PathVariable String id) {
        return paymentClient.charge(id);
    }

    public String fallback(String id, Throwable ex) {
        return "Payment service unavailable for id=" + id;
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
```

---

## 2) Explain the Saga pattern for distributed transactions.

Saga breaks a business transaction into **local transactions** across services.
If one step fails, previously completed steps are rolled back via **compensating actions**.

**Types**
- **Choreography**: services emit/consume events.
- **Orchestration**: central orchestrator tells each step what to do.

**Example flow (Order Saga)**
1. Order created
2. Reserve inventory
3. Charge payment
4. Confirm order

If payment fails: release inventory + mark order failed.

---

## 3) How do you implement tolerance in Spring Boot microservices?

Use multiple resilience patterns together:
- **Timeout** (`TimeLimiter`)
- **Retry** (transient failures)
- **Circuit Breaker**
- **Bulkhead** (isolate thread pools)
- **Rate limiter**
- **Fallback methods**

```java
@Retry(name = "inventory", fallbackMethod = "inventoryFallback")
@TimeLimiter(name = "inventory")
public CompletableFuture<Boolean> reserveAsync(String sku) {
    return CompletableFuture.supplyAsync(() -> inventoryClient.reserve(sku));
}

public CompletableFuture<Boolean> inventoryFallback(String sku, Throwable ex) {
    return CompletableFuture.completedFuture(false);
}
```

---

## 4) What is the difference between `@RestController` and `@Controller`?

- `@Controller`: returns view names (Thymeleaf/JSP), unless `@ResponseBody` is added.
- `@RestController`: shorthand for `@Controller + @ResponseBody`; returns JSON/XML directly.

```java
@Controller
class ViewController {
    @GetMapping("/home")
    public String home() { return "home"; } // renders template
}

@RestController
class ApiController {
    @GetMapping("/api/home")
    public Map<String, String> home() { return Map.of("status", "ok"); }
}
```

---

## 5) Explain lazy initialization in Spring Boot and its impact on startup time.

Lazy initialization delays bean creation until first use.

**Impact**
- Faster startup
- Lower initial memory use
- First request may be slower
- Misconfigurations can appear later (runtime)

```yaml
spring:
  main:
    lazy-initialization: true
```

Per bean:
```java
@Lazy
@Service
class HeavyService {}
```

---

## 6) How would you optimize Spring Boot application startup time?

1. Enable lazy init selectively
2. Remove unused auto-configurations
3. Use `spring-context-indexer`
4. Reduce classpath/dependencies
5. Use AOT/native image where suitable
6. Tune logging (avoid debug at boot)
7. Use lightweight health checks and deferred init

```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class
})
public class App {}
```

---

## 8) Explain the purpose of Spring Boot Actuator metrics

Actuator exposes production metrics/health endpoints for observability.

- `/actuator/health`
- `/actuator/metrics`
- `/actuator/prometheus`

Use cases:
- SLO/SLA monitoring
- Alerting
- Capacity planning
- Performance tuning

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

---

## 9) How do you implement distributed tracing using Spring Cloud Sleuth and Zipkin?

Sleuth adds trace/span IDs to logs and propagates context via headers.
Zipkin aggregates traces for end-to-end visualization.

```yaml
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
```

Log pattern should include `%X{traceId}` and `%X{spanId}`.

---

## 10) Difference: monolith vs SOA vs microservices

- **Monolith**: one deployable unit, shared DB often, simple ops initially.
- **SOA**: coarse-grained services, often ESB-centric integration.
- **Microservices**: small independently deployable services, decentralized data/ownership.

Microservices improve independent scaling/deployment but add operational complexity.

---

## 11) Difference between Callable and Runnable

- `Runnable#run()` returns `void`, cannot throw checked exception.
- `Callable<V>#call()` returns value and can throw checked exception.

```java
ExecutorService pool = Executors.newFixedThreadPool(2);
Future<Integer> f = pool.submit(() -> 42); // Callable
pool.execute(() -> System.out.println("Runnable"));
```

---

## 12) Explain JMM and happens-before

Java Memory Model defines visibility, ordering, and atomicity guarantees.

**Happens-before** means writes by one thread are visible to another if ordering rule exists.
Examples:
- Monitor unlock happens-before subsequent lock on same monitor
- Volatile write happens-before subsequent volatile read
- Thread start/join establish happens-before edges

---

## 13) Why store passwords in `char[]` instead of `String`?

- `String` is immutable, remains in memory until GC.
- `char[]` can be wiped immediately.

```java
char[] pwd = readPassword();
try {
    // authenticate
} finally {
    Arrays.fill(pwd, '\0');
}
```

---

## 14) How does CAS work?

CAS (Compare-And-Swap) atomically updates a value if current value matches expected value.
Used in lock-free algorithms.

```java
AtomicInteger ai = new AtomicInteger(10);
boolean updated = ai.compareAndSet(10, 11);
```

If contention happens, CAS retries in a loop (spin).

---

## 15) Difference: String vs StringBuilder vs StringBuffer

- `String`: immutable, thread-safe by immutability.
- `StringBuilder`: mutable, fast, not thread-safe.
- `StringBuffer`: mutable, synchronized, slower than builder.

Use `StringBuilder` in most single-threaded concatenation scenarios.

---

## 16) Pattern matching enhancements in Java 23

Key evolution areas (across recent Java versions):
- Pattern matching for `instanceof`
- Pattern matching for `switch`
- Record patterns
- Guarded patterns (`when` clauses in switch)

```java
static String fmt(Object o) {
    return switch (o) {
        case Integer i when i > 0 -> "positive int " + i;
        case Integer i -> "int " + i;
        case String s -> "string " + s;
        default -> "other";
    };
}
```

---

## 17) How to handle distributed transactions in microservices?

Preferred approaches:
- Saga (orchestration/choreography)
- Outbox pattern + reliable event publishing
- Idempotency keys
- Retries with deduplication
- Compensation instead of 2PC in most cloud-native systems

---

## 18) Role of API Gateway in Spring Boot microservices

API Gateway provides:
- Single entry point
- Routing and load balancing
- Authentication/authorization
- Rate limiting
- Request/response transformation
- Centralized logging/tracing

Spring Cloud Gateway is a common choice.

---

## 19) Explain service discovery with Eureka

Eureka lets instances register themselves and discover others dynamically.

- Service starts → registers with Eureka server.
- Client asks Eureka for instances.
- Client-side load balancing chooses one instance.

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

---

## 20) How Spring Cloud Config Server manages externalized config

Config Server centralizes configuration from Git/Vault/files.
Clients fetch config at startup (and optionally refresh at runtime).

```yaml
spring:
  application:
    name: order-service
  config:
    import: optional:configserver:http://localhost:8888
```

Benefits: versioned configs, environment profiles, centralized secrets strategy.

---

## 21) Difference between `synchronized` and `ReentrantLock`

- `synchronized`: JVM-managed monitor, simpler syntax.
- `ReentrantLock`: explicit lock/unlock, tryLock, timeout, fairness, condition variables.

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

---

## 22) Purpose of `volatile` keyword

`volatile` ensures **visibility** and prevents certain reorderings.
It does **not** make compound actions atomic.

```java
class FlagHolder {
    private volatile boolean running = true;
    void stop() { running = false; }
    void loop() { while (running) { /* work */ } }
}
```

---

## 23) How does ThreadLocal work internally?

Each `Thread` has a `ThreadLocalMap`.
`ThreadLocal` keys map to values scoped to that thread.

Important: call `remove()` in pooled threads to prevent stale data leaks.

```java
private static final ThreadLocal<String> ctx = new ThreadLocal<>();

ctx.set("req-123");
try {
   // use ctx.get()
} finally {
   ctx.remove();
}
```

---

## 24) Difference between fail-fast and fail-safe iterators

- **Fail-fast** (`ArrayList`, `HashMap` iterators): throw `ConcurrentModificationException` on structural modification.
- **Fail-safe** (`CopyOnWriteArrayList`, concurrent collections): iterate over snapshot/weakly consistent view.

---

## 25) Diamond problem in Java 8 and resolution

Java avoids multiple inheritance of state (classes), but allows multiple interface inheritance.
If two interfaces define same default method, implementing class must override it.

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // explicit conflict resolution
    }
}
```

---

## 26) G1 GC vs CMS

- **CMS**: low pause collector for old gen; deprecated and removed in modern JDKs.
- **G1**: region-based, predictable pause goals, compacts incrementally, default in many JDKs.

G1 generally preferred for server workloads today.

---

## 27) What is a functional interface and relation to lambdas?

Functional interface has exactly one abstract method.
Lambdas provide implementation of that method.

```java
@FunctionalInterface
interface MathOp { int apply(int a, int b); }

MathOp add = (a, b) -> a + b;
System.out.println(add.apply(2, 3));
```

---

## 28) Effectively final variables in lambdas

A local variable captured by lambda must be final or effectively final (assigned once).

```java
int base = 10; // effectively final
Function<Integer, Integer> f = x -> x + base;
```

This avoids unsafe capture of mutable local state from stack frames.

---

## 29) How to detect and fix memory leaks in Java?

**Detect**
- Enable GC logs
- Capture heap dumps on OOM
- Analyze with MAT/YourKit/JFR/JMC
- Track retained-size dominators

**Common leak causes**
- Static collections holding references
- Unclosed resources
- ThreadLocal misuse in pools
- Listener/cache entries never removed

**Fix patterns**
- Weak references where appropriate
- Bounded caches with eviction
- try-with-resources
- `ThreadLocal.remove()`

```java
try (BufferedReader br = Files.newBufferedReader(Path.of("file.txt"))) {
    // read safely
}
```

---

## Quick revision checklist

- Resilience: circuit breaker, retry, timeout, bulkhead, saga, idempotency
- Spring Cloud: gateway, config, discovery, tracing
- Concurrency: JMM, happens-before, CAS, volatile, locks, ThreadLocal
- Core Java: functional interfaces, lambdas, string classes, iterators, GC

