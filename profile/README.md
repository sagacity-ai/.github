# Sagacity

**The reliability layer for Spring AI agents.**

AI agents do real work — charge cards, reserve inventory, send emails, update CRMs. When step 4 of 5 fails, nothing undoes steps 1–3 automatically, nobody approved the irreversible action in step 3, and there is no tamper-evident record of what happened.

Sagacity fixes all three — as a Spring Boot library, with no new infrastructure.

```java
@Workflow("refund-approval")
@Component
public class RefundWorkflow {

    @Stage(order = 1)
    @Compensable(by = "cancelValidation")       // undo if anything fails later
    public String validateRefund(String orderId) {
        return validation.check(orderId);        // returns "val-8821"
    }

    @Stage(order = 2)
    @Compensable(by = "reverseRefund")
    public String issueRefund(String validationId) {
        // 'validationId' injected automatically from stage 1's return value
        return payments.refund(validationId);
    }

    @Stage(order = 3)
    @Gate(approvalRequired = true,               // workflow pauses here
          reason = "Compliance must approve before customer is notified")
    public String notifyCompliance(String refundId) {
        return compliance.log(refundId);
    }

    @Stage(order = 4)
    public void sendConfirmation(String ref) {
        email.send(ref, "Your refund is confirmed");
    }

    @Compensation
    public void cancelValidation(CompensationContext ctx) { validation.cancel(ctx.result()); }

    @Compensation
    public void reverseRefund(CompensationContext ctx) { payments.reverse(ctx.result()); }
}
```

```java
// Run async — pauses at stage 3 until a human approves
WorkflowHandle handle = workflowRuntime.runAsync(refundWorkflow, "ORDER-88210");
workflowRuntime.approveGate(handle.runId(), "notifyCompliance");
handle.awaitCompletion(30, TimeUnit.MINUTES);
// Every stage journaled. If anything fails, stages 2 and 1 compensate in reverse.
```

## What it provides

| Feature | |
|---|---|
| Declarative workflows | `@Workflow`, `@Stage`, `@Gate`, `@Check` annotations |
| Automatic compensation | On failure, completed stages undo in reverse order |
| Human approval gates | Workflow pauses before irreversible actions |
| Pre-flight checks | Block a stage before it executes (`StageCheck` SPI) |
| Tamper-evident audit trail | SHA-256 hash-chained journal, verifiable via REST |
| EU AI Act Article 12 | Append-only, traceable, exportable |
| Embedded UI | `/sagacity/ui` — workflow runs, gate approvals, audit viewer |
| Universal JDBC | PostgreSQL, MySQL, MariaDB, Oracle, H2, SQLite |
| Zero new infrastructure | Library only — add a Maven dependency |

## Quick Start

```xml
<dependency>
    <groupId>io.github.sumitvairagar</groupId>
    <artifactId>sagacity-spring-boot-starter</artifactId>
    <version>0.3.0</version>
</dependency>

<!-- Optional: declarative workflow engine -->
<dependency>
    <groupId>io.github.sumitvairagar</groupId>
    <artifactId>sagacity-workflows</artifactId>
    <version>0.3.0</version>
</dependency>
```

## Repositories

| Repo | Description |
|------|-------------|
| [sagacity](https://github.com/sagacity-ai/sagacity) | Core library — Spring Boot Starter, workflow engine, compensation, audit journal |
| [sagacity-quickstart](https://github.com/sagacity-ai/sagacity-quickstart) | 5-minute runnable demo — 3 scenarios, no API key required |
| [sagacity-dashboard](https://github.com/sagacity-ai/sagacity-dashboard) | Cloud dashboard — hosted audit trail, team approval workflows |

## How it relates to Temporal

Temporal solves **durable execution** — surviving process crashes. Sagacity solves **compensation and evidence** — undoing side effects when business logic says "this should not have happened," gating irreversible actions behind human approval, and producing a tamper-evident record for compliance.

A refund is not a retry. They are complementary.

---

📖 [Documentation](https://sagacity-ai.github.io/sagacity/) · 📦 [Maven Central](https://central.sonatype.com/artifact/io.github.sumitvairagar/sagacity-spring-boot-starter) · 🎬 [@EngineerInAI](https://youtube.com/@EngineerInAI)

Apache License 2.0
