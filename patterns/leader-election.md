# 👑 Leader Election

> In a distributed system, sometimes exactly ONE node must act as the "leader". How do you pick it?

---

## Why Leader Election?

Some tasks must run on exactly one node:

- Sending scheduled emails (don't send twice)
- Coordinating distributed transactions
- Acting as primary in a database cluster
- Processing a specific partition of a queue

**Problem:** Nodes can crash. The leader must be detected as dead and a new one elected.

---

## Algorithms

### Bully Algorithm

- Node with highest ID wins
- When a node thinks the leader is dead → sends election messages to all higher-ID nodes
- If no response → declares itself leader
- ✅ Simple | ❌ Many messages, assumes ordered IDs

### Raft Consensus

- Nodes vote for a leader candidate
- Candidate needs majority vote (quorum)
- Leader sends heartbeats; if missed → new election
- ✅ Well-proven, used in production systems
- Used by: etcd, CockroachDB, TiKV

### ZooKeeper / etcd-based Leader Election (Practical)

```
All nodes race to create an ephemeral node:
  /election/leader

First node to create it → wins leadership
Others watch the node for deletion (leader crash)
On deletion → race again
```

- ✅ Battle-tested, you don't implement the algorithm yourself
- ✅ Ephemeral nodes auto-delete when the session dies (crash detection built in)
- ❌ Adds ZooKeeper/etcd as dependency

---

## Split-Brain Problem

What if the network partitions and both halves elect a leader?

- Two leaders making conflicting decisions = data corruption
- Solution: **quorum** — leader must maintain majority (N/2 + 1) connections
- If minority partition can't reach quorum → steps down

---

## Real-World Usage

| System         | Leader Election                                 |
| -------------- | ----------------------------------------------- |
| Kafka          | ZooKeeper (legacy) / KRaft (new) for controller |
| Elasticsearch  | Zen Discovery / Raft-based                      |
| Redis Sentinel | Sentinel quorum for primary election            |
| PostgreSQL HA  | Patroni uses etcd/ZooKeeper                     |

---

## 🎨 Diagram

See: [`../diagrams/leader-election.excalidraw`](../diagrams/leader-election.excalidraw)
