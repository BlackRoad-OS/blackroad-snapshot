<div align="center">

# 📸 blackroad-snapshot

### Point-in-time snapshots for the BlackRoad OS ecosystem

[![npm version](https://img.shields.io/npm/v/@blackroad/snapshot?style=for-the-badge&color=FF1D6C)](https://www.npmjs.com/package/@blackroad/snapshot)
[![npm downloads](https://img.shields.io/npm/dm/@blackroad/snapshot?style=for-the-badge&color=9C27B0)](https://www.npmjs.com/package/@blackroad/snapshot)
[![License](https://img.shields.io/badge/License-Proprietary-F5A623?style=for-the-badge)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-blackroad.io-FF1D6C?style=for-the-badge)](https://blackroad.io)

Capture, restore, and audit the state of any BlackRoad agent, service, or data pipeline — at any point in time.

[**Get Started →**](#installation) · [**API Reference →**](#api-reference) · [**Stripe Billing →**](#stripe-integration) · [**Examples →**](#examples)

</div>

---

## Index

1. [Overview](#overview)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Core Concepts](#core-concepts)
   - [Snapshots](#snapshots)
   - [Snapshot Chains](#snapshot-chains)
   - [Restore Points](#restore-points)
5. [API Reference](#api-reference)
   - [createSnapshot](#createsnapshot)
   - [restoreSnapshot](#restoresnapshot)
   - [listSnapshots](#listsnapshots)
   - [deleteSnapshot](#deletesnapshot)
   - [diffSnapshots](#diffsnapshots)
6. [Stripe Integration](#stripe-integration)
   - [Plans & Pricing](#plans--pricing)
   - [Webhook Setup](#webhook-setup)
   - [Metered Billing](#metered-billing)
7. [Configuration](#configuration)
8. [E2E Testing](#e2e-testing)
9. [Examples](#examples)
10. [Security](#security)
11. [Contributing](#contributing)
12. [License](#license)

---

## Overview

`@blackroad/snapshot` is the official point-in-time snapshot engine for the **BlackRoad OS** platform. It provides a consistent, auditable record of system state across agents, services, and data pipelines — enabling rollback, diffing, compliance auditing, and billing metering.

**Key capabilities:**

| Capability | Description |
|------------|-------------|
| 📸 **Capture** | Instantly freeze any serialisable state to an immutable snapshot |
| ⏪ **Restore** | Roll back any agent or service to a prior known-good state |
| 🔍 **Diff** | Compute structural diffs between any two snapshots |
| 🔗 **Chain** | Linked snapshot chains with PS-SHA-∞ integrity verification |
| 💳 **Metered Billing** | Stripe-native usage reporting per snapshot and per restore |
| 🔒 **Audit Trail** | Tamper-evident logs for compliance and forensics |

---

## Installation

**Requirements:** Node.js ≥ 18 · npm ≥ 9 · BlackRoad API key

```bash
# npm
npm install @blackroad/snapshot

# yarn
yarn add @blackroad/snapshot

# pnpm
pnpm add @blackroad/snapshot
```

---

## Quick Start

```typescript
import { BlackroadSnapshot } from '@blackroad/snapshot';

const snap = new BlackroadSnapshot({
  apiKey: process.env.BLACKROAD_API_KEY,
  projectId: 'my-project',
});

// Capture a snapshot
const snapshot = await snap.createSnapshot({
  label: 'pre-deploy',
  data: myApplicationState,
});

console.log(`Snapshot captured: ${snapshot.id}`);

// Restore later
await snap.restoreSnapshot(snapshot.id);
```

---

## Core Concepts

### Snapshots

A **snapshot** is an immutable, content-addressed record of a serialisable state object at a specific point in time. Each snapshot carries:

- A globally unique `id` (UUIDv4)
- A `timestamp` (ISO 8601 UTC)
- A `checksum` (PS-SHA-∞)
- Optional human-readable `label`
- Optional `metadata` map

### Snapshot Chains

Snapshots can be linked into **chains** — ordered sequences that capture the evolution of a resource over time. Chains enable efficient delta storage and rollback traversal.

```
snap-001  →  snap-002  →  snap-003 (latest)
   ↑
  restore point
```

### Restore Points

Any snapshot in a chain can be promoted to a **restore point**, making it a named, protected checkpoint that cannot be automatically garbage-collected.

---

## API Reference

### `createSnapshot`

Captures the current state and returns a `Snapshot` object.

```typescript
const snapshot = await snap.createSnapshot(options: CreateSnapshotOptions): Promise<Snapshot>
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `data` | `unknown` | ✅ | The serialisable state to snapshot |
| `label` | `string` | | Human-readable label |
| `metadata` | `Record<string, string>` | | Arbitrary key-value metadata |
| `chainId` | `string` | | Append to an existing chain |
| `ttlDays` | `number` | | Auto-delete after N days |

**Returns:** [`Snapshot`](#snapshot-object)

---

### `restoreSnapshot`

Retrieves a snapshot by ID and returns the original data.

```typescript
const data = await snap.restoreSnapshot(snapshotId: string): Promise<unknown>
```

---

### `listSnapshots`

Returns a paginated list of snapshots for the current project.

```typescript
const page = await snap.listSnapshots(options?: ListOptions): Promise<SnapshotPage>
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | `number` | Results per page (default: `20`, max: `100`) |
| `cursor` | `string` | Pagination cursor |
| `label` | `string` | Filter by label |
| `chainId` | `string` | Filter by chain |
| `before` | `string` | ISO 8601 timestamp upper bound |
| `after` | `string` | ISO 8601 timestamp lower bound |

---

### `deleteSnapshot`

Permanently deletes a snapshot. Restore-point snapshots require `{ force: true }`.

```typescript
await snap.deleteSnapshot(snapshotId: string, options?: { force?: boolean }): Promise<void>
```

---

### `diffSnapshots`

Returns a structured diff between two snapshot IDs.

```typescript
const diff = await snap.diffSnapshots(fromId: string, toId: string): Promise<SnapshotDiff>
```

---

### Snapshot Object

```typescript
interface Snapshot {
  id: string;            // "snap_a1b2c3d4..."
  projectId: string;
  chainId: string | null;
  label: string | null;
  checksum: string;      // PS-SHA-∞ hash
  sizeBytes: number;
  createdAt: string;     // ISO 8601 UTC
  expiresAt: string | null;
  metadata: Record<string, string>;
  isRestorePoint: boolean;
}
```

---

## Stripe Integration

`@blackroad/snapshot` reports metered usage directly to **Stripe** so that your platform charges customers based on actual snapshot activity.

### Plans & Pricing

| Plan | Snapshots/mo | Retentions | Restores/mo | Price |
|------|-------------|------------|-------------|-------|
| **Starter** | 1,000 | 7 days | 100 | $9/mo |
| **Growth** | 25,000 | 30 days | 2,500 | $49/mo |
| **Scale** | 250,000 | 90 days | 25,000 | $199/mo |
| **Enterprise** | Unlimited | Custom | Unlimited | Custom |

> Overage on Starter and Growth plans is billed at $0.002 per additional snapshot.

---

### Webhook Setup

Register the BlackRoad Stripe webhook in your [Stripe Dashboard](https://dashboard.stripe.com/webhooks):

**Endpoint URL:**
```
https://api.blackroad.io/v1/billing/stripe/webhook
```

**Events to enable:**

| Event | Purpose |
|-------|---------|
| `customer.subscription.created` | Provision new project quota |
| `customer.subscription.updated` | Adjust quota on plan change |
| `customer.subscription.deleted` | Suspend project on cancellation |
| `invoice.payment_failed` | Notify account owner |
| `invoice.payment_succeeded` | Confirm billing cycle |

---

### Metered Billing

When metered billing is active, the SDK automatically reports usage after each operation. You can also report usage manually:

```typescript
import { BlackroadSnapshot, reportUsage } from '@blackroad/snapshot';

// Manual usage reporting (advanced)
await reportUsage({
  apiKey: process.env.BLACKROAD_API_KEY,
  projectId: 'my-project',
  operation: 'snapshot.create',
  quantity: 1,
  idempotencyKey: `snap-${Date.now()}`,
});
```

> **Note:** Automatic reporting is enabled by default. Set `billing: { autoReport: false }` in the constructor to disable it.

---

## Configuration

Full configuration reference for the `BlackroadSnapshot` constructor:

```typescript
const snap = new BlackroadSnapshot({
  // Required
  apiKey: string,
  projectId: string,

  // Optional
  region: 'us-east-1' | 'eu-west-1' | 'ap-southeast-1',  // default: 'us-east-1'
  baseUrl: string,                // override API base URL
  timeout: number,                // request timeout in ms (default: 30_000)
  maxRetries: number,             // max retry attempts (default: 3)

  billing: {
    autoReport: boolean,          // enable metered usage reporting (default: true)
    stripeCustomerId: string,     // associate with a Stripe customer
  },

  storage: {
    compression: 'none' | 'gzip' | 'brotli',  // default: 'gzip'
    encryption: boolean,          // client-side encryption (default: false)
    encryptionKey: string,        // required if encryption: true
  },

  logging: {
    level: 'debug' | 'info' | 'warn' | 'error' | 'silent',  // default: 'warn'
  },
});
```

---

## E2E Testing

The package ships with a built-in test harness for end-to-end validation of your snapshot workflows.

### Running the test suite

```bash
# Install dev dependencies
npm install

# Run unit tests
npm test

# Run E2E tests against the live API (requires BLACKROAD_API_KEY)
BLACKROAD_API_KEY=<key> npm run test:e2e

# Run E2E tests against the sandbox
BLACKROAD_API_KEY=<sandbox-key> BLACKROAD_BASE_URL=https://sandbox.api.blackroad.io npm run test:e2e
```

### Writing E2E tests

```typescript
import { BlackroadSnapshot } from '@blackroad/snapshot';
import { describe, it, expect, beforeAll, afterAll } from 'vitest';

describe('snapshot e2e', () => {
  let snap: BlackroadSnapshot;
  let snapshotId: string;

  beforeAll(() => {
    snap = new BlackroadSnapshot({
      apiKey: process.env.BLACKROAD_API_KEY!,
      projectId: process.env.BLACKROAD_TEST_PROJECT_ID!,
    });
  });

  it('creates a snapshot', async () => {
    const result = await snap.createSnapshot({
      label: 'e2e-test',
      data: { hello: 'world', ts: Date.now() },
    });

    expect(result.id).toMatch(/^snap_/);
    expect(result.checksum).toBeTruthy();
    snapshotId = result.id;
  });

  it('restores the snapshot', async () => {
    const data = await snap.restoreSnapshot(snapshotId) as { hello: string };
    expect(data.hello).toBe('world');
  });

  afterAll(async () => {
    if (snapshotId) await snap.deleteSnapshot(snapshotId);
  });
});
```

### Sandbox Environment

Use the **BlackRoad sandbox** to test without affecting production data or Stripe billing:

| Setting | Value |
|---------|-------|
| API Base URL | `https://sandbox.api.blackroad.io` |
| Stripe test keys | Use keys prefixed `sk_test_` |
| Snapshot quota | Unlimited (no billing events emitted) |

---

## Examples

### Agent State Checkpoint

```typescript
// Checkpoint an AI agent before a risky operation
const checkpoint = await snap.createSnapshot({
  label: `agent-checkpoint-${agent.id}`,
  data: agent.exportState(),
  metadata: { agentVersion: agent.version, triggeredBy: 'pre-deploy' },
});

try {
  await agent.runRiskyOperation();
} catch (err) {
  // Roll back on failure
  const savedState = await snap.restoreSnapshot(checkpoint.id);
  agent.importState(savedState);
  throw err;
}
```

### Database Migration Safety Net

```typescript
const dbSnap = await snap.createSnapshot({
  label: 'pre-migration-v3.2.0',
  data: await db.exportSchema(),
  chainId: 'db-migrations',
});

await runMigration('v3.2.0');
```

### Compliance Audit Trail

```typescript
// Capture immutable state for compliance
const auditSnap = await snap.createSnapshot({
  label: `compliance-daily-${new Date().toISOString().slice(0, 10)}`,
  data: await gatherComplianceData(),
  metadata: {
    regulation: 'SOC2',
    auditor: 'internal',
    retentionPolicy: '7years',
  },
});

// Promote to restore point (protected from garbage collection)
await snap.promoteToRestorePoint(auditSnap.id);
```

---

## Security

- All data in transit is encrypted using **TLS 1.3**
- Snapshots at rest are encrypted using **AES-256-GCM**
- Integrity is verified with **PS-SHA-∞** (BlackRoad's proprietary infinite-cascade hashing)
- API keys are scoped per-project and support fine-grained permissions
- To report a security vulnerability, see [SECURITY.md](./SECURITY.md)

---

## Contributing

This repository is proprietary and maintained by the BlackRoad OS team. External contributions are not accepted at this time.

For bug reports or feature requests, open an issue or contact [support@blackroad.io](mailto:support@blackroad.io).

---

## License

© 2024–2026 BlackRoad OS, Inc. All Rights Reserved.

This software is proprietary and confidential. See [LICENSE](./LICENSE) for full terms.

---

<div align="center">

**Part of [BlackRoad OS](https://blackroad.io)** — 30,000+ AI Agents · 17 Organizations · 1,800+ Repos

[blackroad.io](https://blackroad.io) · [docs.blackroad.io](https://docs.blackroad.io) · [status.blackroad.io](https://status.blackroad.io) · [support@blackroad.io](mailto:support@blackroad.io)

*© BlackRoad OS, Inc. All rights reserved.*

</div>
