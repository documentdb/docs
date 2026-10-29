---
title: Role management commands
description: Supported custom-role creation, querying, deletion, and inheritance commands.
type: commands
category: security
---

# Role management commands

Run role-management commands on `admin`. Custom-role functionality requires role CRUD to be enabled and appropriate caller privileges. `dropRole`, `rolesInfo`, `grantRolesToRole`, and `revokeRolesFromRole` additionally require cluster version 0.116 or later.

## createRole

```javascript
db.getSiblingDB("admin").createRole({
  role: "bookings_reader",
  privileges: [{
    resource: { db: "bookings", collection: "flights" },
    actions: ["find"]
  }],
  roles: []
})
```

| Parameter | Description |
| --- | --- |
| **`createRole`** | (Required) A string specifying the non-empty custom-role name; built-in and reserved names are rejected. |
| **`roles`** | (Required) An array specifying the inherited-role list, possibly empty. Each entry is a role-name string or `{ role: "<name>", db: "admin" }`. |
| **`privileges`** | (Required) An array of documents specifying the privilege definitions, possibly empty. |
| **`privileges[].resource`** | (Required within each privilege) A document that requires a non-empty `db` string and a `collection` string. An empty collection name represents database-wide collection scope. Other resource forms are rejected. |
| **`privileges[].actions`** | (Required within each privilege) An array of strings naming supported actions: `find`, `insert`, `update`, `remove`, and `changeStream`. Unknown actions are rejected. |

Inheritance must reference supported existing roles and cannot introduce cycles. Cluster-level MongoDB resource documents and `authenticationRestrictions` are not supported by this parser.

## rolesInfo

```javascript
db.getSiblingDB("admin").getRole("bookings_reader", {
  showPrivileges: true,
  showBuiltInRoles: false
})
```

| Parameter | Description |
| --- | --- |
| **`rolesInfo`** | (Required) A BSON int32, string, document, or array selecting the roles to list. `1` lists roles; otherwise use a role name, `{ role: "<name>", db: "admin" }`, or an array of names/documents. |
| **`showBuiltInRoles`** | (Optional) A boolean that includes built-in roles when listing; defaults to `false`. |
| **`showPrivileges`** | (Optional) A boolean that includes privilege details; defaults to `false`. String modes such as `"asUserFragment"` are not accepted. |

## dropRole

```javascript
db.getSiblingDB("admin").dropRole("bookings_reader")
```

`dropRole` is the required custom-role name string. Built-in and reserved roles cannot be dropped. No other command-specific options are implemented.

## grantRolesToRole and revokeRolesFromRole

After creating `bookings_reader` above, create a dispatcher role and add, then remove, its inherited read access:

```javascript
db.getSiblingDB("admin").createRole({
  role: "bookings_dispatcher",
  privileges: [{
    resource: { db: "bookings", collection: "flights" },
    actions: ["update"]
  }],
  roles: []
})
db.getSiblingDB("admin").grantRolesToRole(
  "bookings_dispatcher", [{ role: "bookings_reader", db: "admin" }]
)
db.getSiblingDB("admin").revokeRolesFromRole(
  "bookings_dispatcher", [{ role: "bookings_reader", db: "admin" }]
)
```

| Parameter | Description |
| --- | --- |
| **`grantRolesToRole`** | (Required) A string specifying the existing custom role to modify. |
| **`revokeRolesFromRole`** | (Required) A string specifying the existing custom role to modify. |
| **`roles`** | (Required) An array specifying the non-empty parent-role list, using strings or `{ role, db }` documents. |

Built-in roles cannot be modified; grant operations also enforce inheritance-cycle and membership limits.

The drop example is independent: do not drop `bookings_reader` before running the inheritance examples.

## Provider-dependent role mutations

`updateRole`, `grantPrivilegesToRole`, and `revokePrivilegesFromRole` can be implemented by some providers. Their availability and options depend on the provider. Use `createRole` to define resource privileges with the options documented above.

Common fields such as `writeConcern` and `comment` do not alter the implemented role operations. See [wire protocol and common fields](../diagnostic/wire-protocol.md).

Source: `pg_documentdb/src/commands/roles.c`, the role parsers and `command_update_role`, `grant_privileges_to_role`, and `revoke_privileges_from_role`.
