---
title: Custom Roles
description: Built-in DocumentDB roles that grant least-privilege access to find, insert, update, and remove operations on collections.
type: commands
category: security
---

# Custom Roles

> **Added in v0.110-0.**

DocumentDB ships with four built-in PostgreSQL roles that mirror the four
classes of CRUD operations performed against a collection. Granting one of these
roles to a database user limits the user to that subset of operations, and is
the recommended building block for least-privilege RBAC on DocumentDB. Each
role is created automatically when the `documentdb` extension is installed or
upgraded to v0.110-0.

| Role | Allowed operations |
| --- | --- |
| `documentdb_api_find_role` | `find`, `aggregate`, `count`, `distinct`, `getMore` |
| `documentdb_api_insert_role` | `insert` (and bulk insert variants) |
| `documentdb_api_update_role` | `update` (and bulk update variants) |
| `documentdb_api_remove_role` | `delete` |

## Granting a role

You grant the roles with standard PostgreSQL `GRANT` syntax against the
DocumentDB database. The following example creates a read-only application
user that can run queries but cannot modify data:

```sql
-- Create the database user.
CREATE USER reporting_app WITH PASSWORD 'change-me';

-- Allow it to run find / aggregate / count / distinct / getMore.
GRANT documentdb_api_find_role TO reporting_app;
```

To create a write-only ingestion user that can only insert documents:

```sql
CREATE USER ingest_app WITH PASSWORD 'change-me';
GRANT documentdb_api_insert_role TO ingest_app;
```

To grant full CRUD access, grant all four roles:

```sql
GRANT documentdb_api_find_role,
      documentdb_api_insert_role,
      documentdb_api_update_role,
      documentdb_api_remove_role
TO app_user;
```

## Revoking a role

Use standard `REVOKE` syntax to take a role away from a user:

```sql
REVOKE documentdb_api_remove_role FROM ingest_app;
```

## Composing the roles

The four roles are independent — granting `documentdb_api_update_role` does not
implicitly grant `documentdb_api_find_role`. Combine them as needed for the
access pattern you want to allow.

For coarser-grained role management at the MongoDB protocol level, see the
related role-management commands `createRole`, `rolesInfo`, and `dropRole`,
which were introduced in earlier DocumentDB releases.
