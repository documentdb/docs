---
title: YugabyteDB compatibility
description: Understand the current compatibility guidance for using DocumentDB with YugabyteDB.
---

# YugabyteDB compatibility

DocumentDB is implemented as PostgreSQL extensions and is tested with supported PostgreSQL versions. YugabyteDB is PostgreSQL-compatible, but it isn't currently documented as a supported runtime for DocumentDB.

## Guidance

Use a supported PostgreSQL environment when you deploy DocumentDB. For local development, use [`documentdb-local`](../documentdb-local/index.md). For self-managed Linux deployments, use the [pre-built packages](prebuilt-packages.md) that match your operating system, PostgreSQL major version, and CPU architecture.

If you want to experiment with YugabyteDB, treat it as unsupported until the DocumentDB project publishes compatibility guidance and test coverage for that runtime.
