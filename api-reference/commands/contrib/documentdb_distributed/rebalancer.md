---
title: Distributed rebalancer
description: Start, inspect, and stop DocumentDB distributed shard rebalancing through wire protocol commands.
type: commands
category: documentdb_distributed
---

# Distributed rebalancer

The `balancerStart`, `balancerStatus`, and `balancerStop` commands require [documentdb_distributed and its prerequisites](overview.md#prerequisites-and-installation). They manage background shard-rebalancing jobs across the cluster, rather than changing a collection's logical shard key. Run these wire protocol commands against the `admin` database.

## Availability

Availability is subject to provider gateway implementations. The provider gateway must plumb these wire commands to the distributed rebalancer implementation, including forwarding the `strategy` option when supplied.

The deployment must also enable the separately gated rebalancer capability, which is disabled by default in the audited implementation. Starting a rebalance requires change streams to be disabled in the deployment.

## Inspect status

```javascript
db.adminCommand({ balancerStatus: 1 })
```

The response includes `ok` and `mode`, which is `full` when jobs are scheduled or running and `off` otherwise. It can also include `runningJobs`, `otherJobs`, and `strategies`. Inspect individual job states as well: cancelling or failing jobs can still prevent a new rebalance even when `mode` is `off`.

The `strategies` entries identify available strategy names and whether each is the default. `runningJobs` and `otherJobs` contain shard-rebalancing job information, including task-state counts and task details when available.

## Start rebalancing

Start a background rebalance using the current default strategy:

```javascript
db.adminCommand({ balancerStart: 1 })
```

The command returns `{ ok: 1 }` after starting the background work. It does not wait for rebalancing to finish; monitor progress with `balancerStatus`.

| Parameter | Description |
| --- | --- |
| **`balancerStart`** | (Required) The command marker, conventionally `1`. |
| **`strategy`** | (Optional) A string selecting a shard-rebalancing strategy. Use a name from `strategies` in the status response. Supplying it changes the default rebalance strategy before starting the job. |

When you supply a strategy, include `strategy` alongside `balancerStart` in the command document. Strategy names come from the deployment rather than from a fixed DocumentDB list; use a `strategy_name` from the `strategies` entries returned by `balancerStatus`.

For example, when status lists at least one strategy, this selects the first advertised strategy and starts a rebalance with it. Run this as an alternative to the default-strategy start above, not while that rebalance is active:

```javascript
const status = db.adminCommand({ balancerStatus: 1 })
const strategy = status.strategies[0].strategy_name
db.adminCommand({ balancerStart: 1, strategy: strategy })
```

Only one rebalance can be active. A job in `scheduled`, `running`, `cancelling`, or `failing` state blocks a new start.

## Stop rebalancing

```javascript
db.adminCommand({ balancerStop: 1 })
```

The response includes `ok` and `wasActive`. `wasActive: true` means an active job was found and a stop was requested; it is not proof that every task has already stopped. Call `balancerStatus` again to confirm the final job states. If no job is active, the call succeeds with `wasActive: false`.

## Implementation

See [wire protocol and common fields](../../diagnostic/wire-protocol.md) for the source revision.

The `balancerStart`, `balancerStatus`, and `balancerStop` wire APIs map to the SQL UDFs `documentdb_api_distributed.rebalancer_start`, `documentdb_api_distributed.rebalancer_status`, and `documentdb_api_distributed.rebalancer_stop`, respectively.

Source: [distributed shard rebalancer](https://github.com/documentdb/documentdb/blob/f027bf1b2fc5ac1e83bfccf97ecc50b69835c758/internal/pg_documentdb_distributed/src/rebalancer/shard_rebalancer.c), `command_rebalancer_start`, `command_rebalancer_status`, and `command_rebalancer_stop`; configuration: `src/documentdb_distributed_init.c`, `documentdb_distributed.enable_shard_rebalancer_apis`.
