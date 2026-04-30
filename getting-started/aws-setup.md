---
title: AWS setup
description: Learn deployment options for running open-source DocumentDB on AWS.
---

# Run DocumentDB on AWS

You can run open-source DocumentDB on AWS infrastructure for development, testing, or self-managed deployments.

## Choose a deployment option

| Option | Use when |
| --- | --- |
| [`documentdb-local`](../documentdb-local/index.md) on an EC2 instance | You want the fastest way to test DocumentDB on AWS infrastructure. |
| [Pre-built Linux packages](prebuilt-packages.md) on EC2 | You want a self-managed DocumentDB deployment with explicit control over PostgreSQL and extension installation. |
| Amazon DocumentDB | You want the AWS managed MongoDB-compatible service. This service is separate from the open-source DocumentDB project. |

## Avoid product confusion

Open-source DocumentDB and Amazon DocumentDB are different products. Both provide MongoDB-compatible experiences for supported workloads, but they have different implementations, release cycles, and feature sets. Use the [API reference](../api-reference/) to confirm which commands and operators are available in open-source DocumentDB.
