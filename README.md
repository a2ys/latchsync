# MeshSync

> Room made local storage easy. MeshSync makes synchronization easy.

Every Android app that needs to work offline ends up writing the same thing. A Room database to hold pending operations. A WorkManager job to retry when the network is back. Network callbacks to detect when the network is back. Deduplication so the same request does not go twice. Conflict handling for when the server has moved on without you.

You write it once, forget it. You write it again in the next project. And again.

MeshSync is that code, extracted into a library you drop in and never think about again.

```kotlin
// This is where you are headed.
MeshSync.enqueue(
    SyncOperation(
        id = UUID.randomUUID().toString(),
        type = "PAYMENT",
        payload = payment.toJson()
    )
)
```

No WorkManager boilerplate. No Room DAOs. No network callbacks. MeshSync stores it locally, watches the network, and retries — even if your app is killed in between.

## The problem, concretely

You are building a payment app. The merchant taps "Collect Payment". The network drops. What happens?

If you have not thought about it — the request fails silently, and your user has no idea whether the payment went through or not. You have a problem.

The standard answer is: store it in Room, schedule a WorkManager job, add a NetworkCallback, handle retries, deduplicate the requests, resolve conflicts. That is six things for one feature. And you will write all six again in the next app, and the one after that.

MeshSync handles all six. You just enqueue.

## Features

- **Persistent queue** — survives process death, app restarts, and reboots
- **Automatic retry** — exponential backoff, battery-aware via WorkManager constraints
- **Connectivity-aware** — triggers sync the moment you are back online, not on a timer
- **Idempotency** — every operation carries a stable ID; safe to retry indefinitely
- **Conflict resolution** — pluggable strategies, last-write-wins by default
- **Observable state** — `Flow<SyncState>` so your UI always knows what is happening
- **P2P sync** *(coming soon)* — phone-to-phone sync over WiFi Direct, no internet required

## How it works

```
MeshSync.enqueue(op)
        │
  LocalQueue (Room)         ← persisted immediately
        │
  ConnectivityMonitor       ← watching for network
        │
  online? ──yes──  SyncWorker  ──  your endpoint
        │                               │
       no                           success? ──yes──  mark done
        │                               │
  RetryScheduler                        │
  (WorkManager backoff)    ─────────────┘
```

Everything internal stays under `internal/`. The only surface you ever touch is `MeshSync`, `SyncOperation`, and `SyncState`. Everything else is an implementation detail.

## Why not just use WorkManager + Room directly?

You can. MeshSync *is* WorkManager + Room — with the wiring written for you and the edge cases handled.

|  | Room + WorkManager | MeshSync |
|---|---|---|
| Local persistence | ✓ you write it | ✓ built in |
| Auto-retry | ✓ you write it | ✓ built in |
| Connectivity trigger | ✓ you write it | ✓ built in |
| Idempotency | ✗ you write it | ✓ built in |
| Conflict resolution | ✗ you write it | ✓ built in |
| Observable state | ✗ you write it | ✓ built in |
| P2P sync | ✗ not possible | ✓ coming soon |

If you enjoy writing the same wiring for every project, by all means. MeshSync is for everyone else.

## Roadmap

- [ ] Local operation queue (Room)
- [ ] Connectivity-aware sync trigger
- [ ] WorkManager retry with exponential backoff
- [ ] Idempotency + deduplication
- [ ] Conflict resolution strategies
- [ ] Transaction ordering guarantees
- [ ] `meshsync-p2p` — phone-to-phone sync over WiFi Direct / Nearby API
- [ ] Encryption at rest
- [ ] Sync telemetry

## Status

MeshSync is actively being built in public. Nothing is released yet — but everything is being documented as it happens. Watch the repo to follow along.

If you have hit pain points with offline sync in your own apps, open an issue and describe what you needed that did not exist. That is how the roadmap gets shaped.

Follow the build: [@unreal_sapien on X](https://x.com/unreal_sapien), [@a2ys on LinkedIn](https://linkedin.com/in/a2ys)

## License

```
Apache License 2.0

Copyright (c) 2024 Aayush Shukla (a2ys)
```
