# Sagacity

**The SAGA pattern for Spring AI agents.**

AI agents do real work — charge cards, reserve inventory, send emails. When step 4 of 5 fails, nothing undoes steps 1–3 automatically. Sagacity fixes that with declarative compensation and a tamper-evident audit trail.

```java
@Tool(description = "Reserve inventory for a product")
@Compensable(by = "releaseInventory")
public String reserveInventory(String sku, int qty) { ... }

@Compensation
public void releaseInventory(CompensationContext ctx) {
    inventory.release(ctx.result()); // runs automatically on failure
}
```

## Repositories

| Repo | Description |
|------|-------------|
| [sagacity](https://github.com/sagacity-ai/sagacity) | Core Java library — Spring Boot Starter, compensation engine, hash-chained audit journal |
| [sagacity-quickstart](https://github.com/sagacity-ai/sagacity-quickstart) | 5-minute runnable demo — clone, add API key, run |

## Quick Start

```xml
<dependency>
    <groupId>io.github.sumitvairagar</groupId>
    <artifactId>sagacity-spring-boot-starter</artifactId>
    <version>0.1.0</version>
</dependency>
```

📖 [Full documentation](https://sagacity-ai.github.io/sagacity/) · ⭐ [Star on GitHub](https://github.com/sagacity-ai/sagacity) · 🎬 [@EngineerInAI](https://youtube.com/@EngineerInAI)

---

Apache License 2.0
