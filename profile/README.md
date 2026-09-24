# Sagacity

**Human oversight and audit for Spring AI agents.**

Your AI agents are making decisions that affect real people — approving transactions, sending emails, charging cards, updating records. Right now, nobody approved the irreversible action before it ran. Nothing undoes completed steps when something later fails. There is no compliance-grade record for your regulator.

Sagacity is the governance layer that sits between your Spring AI agent and the actions it takes.

```java
@Workflow("refund-approval")
@Component
public class RefundWorkflow {

    @Stage(order = 1)
    @Compensable(by = "cancelValidation")       // undone automatically if anything fails
    public String validateRefund(String orderId) { ... }

    @Stage(order = 2)
    @Compensable(by = "reverseRefund")
    public String issueRefund(String validationId) { ... }

    @Stage(order = 3)
    @Gate(approvalRequired = true,               // workflow pauses here
          reason = "Compliance must approve before customer is notified")
    public String notifyCompliance(String refundId) { ... }

    @Stage(order = 4)
    public void sendConfirmation(String ref) { ... }
}
```

```java
// Workflow pauses at stage 3. Approve from the embedded UI or via REST.
// If rejected — stages 2 and 1 compensate in reverse. Full audit trail either way.
WorkflowHandle handle = workflowRuntime.runAsync(refundWorkflow, "ORDER-88210");
```

## What it provides

| Feature | |
|---|---|
| **Human approval gates** | Workflow pauses before irreversible actions. Approve or reject from the embedded UI or REST API. |
| **Automatic compensation** | On failure or rejection, completed stages undo in reverse order. |
| **Tamper-evident audit trail** | SHA-256 hash-chained journal. Every decision recorded. Verifiable. |
| **EU AI Act Article 12** | Append-only, traceable, exportable. |
| **Durable gates** | Gate approvals survive JVM restarts — state persisted to your existing database. |
| **Embedded UI** | `/sagacity/ui` — workflow runs, gate approvals, audit viewer. Zero config. |
| **Universal JDBC** | PostgreSQL, MySQL, MariaDB, Oracle, H2, SQLite. |
| **Zero new infrastructure** | Add a Maven dependency. Nothing else. |

## Quick Start

```xml
<dependency>
    <groupId>io.github.sumitvairagar</groupId>
    <artifactId>sagacity-spring-boot-starter</artifactId>
    <version>0.4.0</version>
</dependency>

<!-- Workflow engine with @Stage, @Gate, @Check -->
<dependency>
    <groupId>io.github.sumitvairagar</groupId>
    <artifactId>sagacity-workflows</artifactId>
    <version>0.4.0</version>
</dependency>
```

## Repositories

| Repo | Description |
|------|-------------|
| [sagacity](https://github.com/sagacity-ai/sagacity) | Core library — Spring Boot Starter, workflow engine, compensation, audit journal |
| [sagacity-quickstart](https://github.com/sagacity-ai/sagacity-quickstart) | 5-minute runnable demo — 3 scenarios, no API key required |
| [sagacity-dashboard](https://github.com/sagacity-ai/sagacity-dashboard) | Cloud dashboard — hosted audit trail, team approval workflows (Sagacity Cloud) |

---

📖 [Documentation](https://sagacity-ai.github.io/sagacity/) · 📦 [Maven Central](https://central.sonatype.com/artifact/io.github.sumitvairagar/sagacity-spring-boot-starter) · 🎬 [@EngineerInAI](https://youtube.com/@EngineerInAI)

Apache License 2.0
