---
title: 1.0 Release Candidates
description: The DocumentDB 1.0 release candidates have ended. DocumentDB 1.0 is released; upgrading hosts and volumes that ran a release candidate isn't supported.
---

# 1.0 release candidates

The 1.0 release candidates have ended: [DocumentDB 1.0](release-notes-1.0.md) was released on 2026-10-19, from the same commit as v1.0-RC2.

Upgrading hosts and data volumes that ran a release candidate to 1.0 isn't supported, and the installer refuses such hosts. Install 1.0 on a fresh host or a new volume, following [Pre-built Packages](prebuilt-packages.md) or [DocumentDB Local](../documentdb-local/index.md), and move any data you need with `mongodump` and `mongorestore`.
