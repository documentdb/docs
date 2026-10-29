---
title: User management commands
description: Supported createUser, updateUser, dropUser, usersInfo, and user role-membership fields.
type: commands
category: security
---

# User management commands

Run user-management commands on `admin`. They require user CRUD support and the caller's administrative privileges. Native password operations also require native authentication to be enabled.

## createUser

```javascript
db.getSiblingDB("admin").createUser({
  user: "bookings_operator",
  pwd: passwordPrompt(),
  roles: [{ role: "readAnyDatabase", db: "admin" }]
})
```

| Parameter | Description |
| --- | --- |
| **`createUser`** | (Required) A string specifying the non-empty, non-reserved username within PostgreSQL's role-name length limit. |
| **`pwd`** | (Required) A string specifying the non-empty password for native users; server password constraints can apply. |
| **`roles`** | (Required) An array of documents specifying the role definitions containing `role` and optionally `db` or `$db`; database qualifiers must be `"admin"`. |
| **`customData`** | (Optional) A document whose only recognized field is `IdentityProvider`. It selects a deployment-provided external-identity path, not arbitrary user metadata; external-identity provisioning does not use the native password requirement. |

For native user creation, accepted built-in role sets are `readAnyDatabase`, `readWriteAnyDatabase` where available, or `readWriteAnyDatabase` together with `clusterAdmin`. Alternatively, specify one supported custom role. A custom role cannot be mixed with built-in roles, and multiple custom roles are rejected by this creation path. This is not arbitrary MongoDB role assignment.

`mechanisms`, `digestPassword`, and `authenticationRestrictions` are not recognized by the default parser.

## updateUser

```javascript
db.getSiblingDB("admin").updateUser("bookings_operator", { pwd: passwordPrompt() })
```

| Parameter | Description |
| --- | --- |
| **`updateUser`** | (Required) A string specifying the existing username. |
| **`pwd`** | (Optional) A non-empty string specifying the new native-user password. |

The parser does not provide generic `customData` updates.

## dropUser

```javascript
db.getSiblingDB("admin").dropUser("bookings_operator")
```

`dropUser` is the required username string. There are no additional command-specific options.

## usersInfo

```javascript
db.getSiblingDB("admin").getUser("bookings_operator", { showPrivileges: true })
```

| Parameter | Description |
| --- | --- |
| **`usersInfo`** | (Required) A BSON int32, string, or document selecting the users to list. `1` lists all users; a string selects one. Document forms are `{ user: "<name>", db: "admin" }` or `{ forAllDBs: true }`. Arrays are not supported. |
| **`showPrivileges`** | (Optional) A boolean that requests privileges; defaults to `false` and privilege output depends on the server's users-info privilege capability. |

`forAllDBs` does not expand the authentication namespace: users are provisioned in `admin`. The parser does not recognize `showCredentials`, `showAuthenticationRestrictions`, or `filter`.

The backend also recognizes a string `getUser` selector, but `getUser` is not a standalone wire command recognized by the default name parser. The shell's `getUser()` helper sends `usersInfo` and extracts the selected user from the response.

## grantRolesToUser and revokeRolesFromUser

Create `bookings_reader` using the [role-command example](role-commands.md#createrole) before granting it. Run the drop-user example independently so that `bookings_operator` still exists.

```javascript
db.getSiblingDB("admin").grantRolesToUser(
  "bookings_operator", [{ role: "bookings_reader", db: "admin" }]
)
db.getSiblingDB("admin").revokeRolesFromUser(
  "bookings_operator", [{ role: "bookings_reader", db: "admin" }]
)
```

| Parameter | Description |
| --- | --- |
| **`grantRolesToUser`** | (Required) A string specifying the existing username. |
| **`revokeRolesFromUser`** | (Required) A string specifying the existing username. |
| **`roles`** | (Required) An array specifying the non-empty role list. Entries may be role-name strings or `{ role: "<name>", db: "admin" }` documents. |

Membership limits and supported built-in/custom-role rules are enforced by the backend. These commands do not make `updateUser.roles` a supported field.

See [wire protocol and common fields](../diagnostic/wire-protocol.md) for shared request metadata.

## Implementation

Source: `pg_documentdb/src/commands/users.c`, the `ParseCreateUserSpec`, `ParseUpdateUserSpec`, `ParseDropUserSpec`, `ParseGetUserSpec`, `ParseGrantRolesToUserSpec`, and `ParseRevokeRolesFromUserSpec` parsers.
