---
title: Compatibility commands
description: Describe the simple responses of DocumentDB compatibility wire commands.
type: commands
category: diagnostic
---

# Compatibility commands

These commands have gateway handlers but do not implement all behavior implied by their MongoDB names. Except where noted, no command-specific option changes their response.

## Status and information commands

| Command | Invocation | Implemented behavior |
| --- | --- | --- |
| `ping` | `{ ping: 1 }` | Returns `{ ok: 1 }`; checks gateway responsiveness, not database health. |
| `buildInfo` | `{ buildInfo: 1 }` | Returns configured MongoDB-compatible `version`, `versionArray`, `bits: 64`, and `maxBsonObjectSize`. `buildinfo` is an alias. Use [`hello`](hello.md) for the engine version. |
| `getCmdLineOpts` | `{ getCmdLineOpts: 1 }` | Returns an empty `argv` array, not the gateway's actual startup arguments. |
| `getLog` | `{ getLog: "global" }` | Returns empty `log` and `totalLinesWritten: 0`; does not expose server logs or implement log-name selection. |
| `hostInfo` | `{ hostInfo: 1 }` | Returns current time with placeholder memory, OS, and CPU fields. |
| `isdbgrid` | `{ isdbgrid: 1 }` | Returns `isdbgrid: 1` and the configured hostname. Not a complete shard-health check. |
| `whatsmyuri` | `{ whatsmyuri: 1 }` | Returns only `{ ok: 1 }`; no client URI. |

For example:

```javascript
db.runCommand({ ping: 1 })
db.serverBuildInfo()
```

## getDefaultRWConcern

Run on `admin`:

```javascript
db.adminCommand({ getDefaultRWConcern: 1 })
```

The response always reports implicit `majority` read and write concerns, with `wtimeout: 0`. This is a fixed compatibility response, not evidence that arbitrary per-command concern options are implemented.

## Other no-ops

`enableSharding` returns success without enabling a database-level sharding state. `refreshSessions` and `killAllSessionsByPattern` return success without refreshing or terminating sessions. Their command arguments are not used to perform those operations. See [session commands](../sessions/session-commands.md) for commands that actually terminate sessions.

See [wire protocol and common fields](wire-protocol.md). Source: gateway `processor/constant.rs` and `processor/process.rs`.
