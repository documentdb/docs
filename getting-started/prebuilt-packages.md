---
title: Pre-built Packages
description: Install the complete DocumentDB stack with a setup wizard, or choose individual PostgreSQL and gateway packages. Includes PGDG and source-build options.
---

# Pre-built Packages

Install a complete MongoDB-compatible service, or add only the components you need to a PostgreSQL instance you manage. Since v0.116, DocumentDB's packages include a setup wizard, administrator tools, and systemd integration.

Get first-party packages from [documentdb.io](https://documentdb.io/packages), extension packages from [PGDG](#pgdg-extension-packages), or [build packages for another target](#community-builds) with the provided scripts.

## Choose an installation path

| You want | Installation |
| --- | --- |
| A working DocumentDB instance with minimal setup | **[Stand-alone stack](#set-up-and-connect), recommended:** install `documentdb` and run the wizard. |
| MongoDB-compatible access to a PostgreSQL instance you manage | [Extension + gateway](#add-a-gateway): configure the components separately and retain your PostgreSQL service lifecycle. |
| DocumentDB through SQL, without a gateway | [Extension only](#extension-only-installation): install the extension and optional administrator tools. |

## First-party packages

The current release is [`v0.117-0`](https://github.com/documentdb/documentdb/releases/tag/v0.117-0), published on **2026-09-10**. First-party CI builds and tests the full stack for:

| Distribution | PostgreSQL | Architectures |
| --- | --- | --- |
| Ubuntu 24.04 (DEB) | 17, 18 | amd64, arm64 |
| RHEL-compatible 9 (RPM) | 17, 18 | x86_64, aarch64 |

Ubuntu 24.04 with PostgreSQL 18 is the recommended default. Other targets may be available through PGDG or the build scripts; they are not part of this hosted matrix.

## Set up and connect

These steps are for a **new stand-alone installation**. For an existing deployment, read [Upgrades and retired targets](#upgrades-and-retired-targets) first.

### 1. Install the stack

Use the [Package Finder](https://documentdb.io/packages) or [Linux Packages Quick Start](https://documentdb.io/docs/getting-started/packages/) to configure the repositories for your distribution and architecture. First-party packages depend on PostgreSQL and extensions from PGDG; EL9 also needs EPEL and CRB / CodeReady Builder.

With the repositories configured, install the stack on Ubuntu:

```bash
sudo apt install documentdb
```

On EL9, use `sudo dnf install documentdb`. Both `documentdb` (the PG18 default) and `documentdb-18` (used by the Package Finder) install the complete stack.

### 2. Run the setup wizard

> The gateway listens on all interfaces by default. Restrict access to port `10260` before setup on a network-accessible host; use a trusted certificate before allowing remote clients.

```bash
sudo documentdb-setup --pg-version 18 --use-new-postgres-instance --admin-user admin
```

The wizard prompts for the admin password, creates a private PostgreSQL 18 instance, configures extensions and the admin user, and starts the gateway. On systemd hosts it also enables the stack at boot.

### 3. Connect

Install [mongosh](https://www.mongodb.com/docs/mongodb-shell/install/) separately, then connect interactively:

```bash
mongosh localhost:10260 --authenticationMechanism SCRAM-SHA-256 \
  --tls --tlsAllowInvalidCertificates -u admin -p
```

Enter the password set in the wizard, then run:

```javascript
db.runCommand({ ping: 1 })
```

The local example accepts the generated self-signed certificate. A bare `-p` prompts for a password; non-interactive scripts must supply a password explicitly.

### Optional setup controls

| Requirement | How |
| --- | --- |
| Use PostgreSQL 17 | Install `documentdb-17` instead of `documentdb`, and use `--pg-version 17`. |
| Preview changes | Add `--dry-run` before applying setup. |
| Change the gateway port | Add `--listen-port 27017`, or another unused port. |
| Load sample data | Add `--load-sample-data` to load [StoreData](https://documentdb.io/docs/documentdb-local/#built-in-sample-data); requires `mongosh`. |
| Automate setup | Use `--yes` with `--admin-password-file /path/to/protected-file` (mode `0600`) or `--admin-password-stdin`. |
| Adopt an existing local PostgreSQL instance | Replace `--use-new-postgres-instance` with `--target-postgres-instance 18/main`; review the proposed changes and plan PostgreSQL restarts. |

Inspect state with `sudo documentdb-setup --pg-version 18 --status`. On systemd, restart this stack with `sudo systemctl restart documentdb-local@18.target`; adopted PostgreSQL retains its own lifecycle. See [Operating a package install](https://documentdb.io/docs/linux-packages/) for TLS and other operational guidance.

## What each release publishes

`N` is the PostgreSQL major version. The package manager installs the dependencies of your selected package.

| Package | Purpose |
| --- | --- |
| `documentdb` | Convenience meta package selecting `documentdb-18`; also provides the `documentdb-local.target` alias. |
| `documentdb-N` | Complete stand-alone stack for major N; owns its systemd lifecycle. |
| `postgresql-N-documentdb` (DEB) / `postgresqlN-documentdb` (RPM) | Extension libraries and SQL files only; does not configure existing PostgreSQL instances. |
| `documentdb-postgresql-tools` | `documentdb-tune`, `documentdb-createcluster`, `documentdb-register-gateway`, and `documentdb-gateway-admin`. |
| `documentdb-gateway` | MongoDB wire-protocol runtime and its service; does not configure PostgreSQL. |
| `documentdb-common` | Shared wizard, service templates, helpers, and sample data, pulled in by `documentdb-N`. |

## Extension-only installation

This **alternative** uses first-party packages on Ubuntu 24.04 with an administrator-managed PostgreSQL 18 cluster named `main`. Install only the extension and tools, not `documentdb` or `documentdb-N`:

```bash
sudo apt install postgresql-18-documentdb documentdb-postgresql-tools
sudo documentdb-tune --pg-version 18 --cluster main --dry-run
```

Review the proposed configuration, then apply it during a suitable restart window:

```bash
sudo documentdb-tune --pg-version 18 --cluster main --yes
sudo systemctl restart postgresql@18-main
sudo -u postgres /usr/bin/psql --cluster 18/main -d postgres -v ON_ERROR_STOP=1 \
  -c "CREATE EXTENSION IF NOT EXISTS documentdb CASCADE;" \
  -c "CREATE EXTENSION IF NOT EXISTS documentdb_extended_rum CASCADE;"
```

In v0.117, both extension statements are required after the restart; `CASCADE` does not create `documentdb_extended_rum`. Adjust the major and cluster name for your instance. For other layouts, follow the restart and connection commands printed by `documentdb-tune`.

For a **new** Debian/Ubuntu cluster, `sudo documentdb-createcluster 18 docdb --start` combines cluster creation, tuning, startup, and extension creation. Choose a cluster name that does not already exist.

### Add a gateway

After the extension-only steps, install and register a gateway. Store the admin password in a protected file (mode `0600`) and substitute its path below:

```bash
sudo apt install documentdb-gateway
sudo documentdb-register-gateway --target-postgres-instance 18/main \
  --admin-user admin --admin-password-file /path/to/admin-password --yes
sudo systemctl reload postgresql@18-main
sudo systemctl enable --now documentdb-gateway
```

Registration configures authentication and the admin user. Connect with `mongosh` as above. This path uses `documentdb-gateway.service`, not the stand-alone target, and requires PostgreSQL on the **same host**; remote TCP/password backends are not supported by package-managed registration.

## PGDG extension packages

DocumentDB is also included in the **PostgreSQL Global Development Group (PGDG) APT repository**, with packaging maintained by the Debian PostgreSQL team. This is an extension-only distribution: it does not include the first-party gateway, setup wizard, or administrator tools.

For example, as of **2026-09-10**, PGDG publishes DocumentDB **0.116** for Debian 13 (`trixie-pgdg`), PostgreSQL **15-18**, on **amd64 and arm64**. Versions and targets differ from first-party releases; consult the [PGDG package pool](https://apt.postgresql.org/pub/repos/apt/pool/main/d/documentdb/).

For a new Debian 13 installation, follow the [PGDG repository setup instructions](https://www.postgresql.org/download/linux/debian/#apt), then inspect the candidate for your chosen PostgreSQL major version:

```bash
sudo apt update
apt-cache policy postgresql-16-documentdb
```

If PGDG provides a candidate for your host, install it:

```bash
sudo apt install postgresql-16-documentdb
```

Configure PostgreSQL and create the extensions for that packaged version separately; the first-party helper commands above do not apply. Do not use another distribution's repository to obtain a missing package, or treat a provider switch as an in-place upgrade.

## Community builds

No package for your target? The supplied scripts build DEB or RPM packages with Docker; select the distribution and PostgreSQL major version rather than writing packaging files yourself.

Use a Linux build environment with Git, Docker, Bash 4+, and GNU utilities, on the target architecture. For example, build the extension for Debian 12 and PostgreSQL 18:

```bash
git clone --depth 1 --branch v0.117-0 https://github.com/documentdb/documentdb.git
cd documentdb
./packaging/build_packages.sh --os deb12 --pg 18 --output-dir packages
```

For the full stand-alone DEB set, make `dpkg-deb` available on the build host, then build the gateway and shared packages:

```bash
./packaging/gateway/build_gateway_packages.sh --os deb12 --pg 18 \
  --version 0.117.0 --output-dir packages
./packaging/build_extra_packages.sh --type deb --pg 18 \
  --version 0.117.0 --output-dir packages
```

Outputs go into `packages/`. RPM extras instead require host `rpmbuild` and applicable RPM macros. The [versioned packaging guide](https://github.com/documentdb/documentdb/blob/v0.117-0/packaging/README.md) lists targets, prerequisites, and clean-install checks.

Validate builds for targets outside the first-party CI matrix in your environment. PostgreSQL 15 is extension-only; package-managed gateway setup requires PostgreSQL 16 or newer.

## Download and verify

Use release assets when you need an exact first-party version rather than a repository install. `v0.117-0` includes 22 Linux packages, `SHA256SUMS`, and `manifest.txt`.

```bash
gh release download v0.117-0 -R documentdb/documentdb -D pkgs && cd pkgs && sha256sum -c SHA256SUMS
```

This downloads all formats, architectures, and PostgreSQL major versions. Install only the matching subset below, not every downloaded file.

## Install from downloaded assets

Enable the dependency repositories first: PGDG, plus EPEL and CRB / CodeReady Builder on EL9. Pass the matching local files to **one** `apt` or `dnf` command so dependencies resolve in the same transaction.

These examples install `documentdb-18` and its dependencies; the optional `documentdb` meta package is not needed.

### DEB (Ubuntu 24.04, PostgreSQL 18, amd64)

For arm64, replace `amd64` with `arm64`. Only the gateway and extension files are architecture-specific.

```bash
sudo apt install ./ubuntu24.04-documentdb-18_0.117.0_all.deb \
                 ./ubuntu24.04-documentdb-common_0.117.0_all.deb \
                 ./ubuntu24.04-documentdb-postgresql-tools_0.117.0_all.deb \
                 ./ubuntu24.04-documentdb-gateway_0.117.0_amd64.deb \
                 ./ubuntu24.04-postgresql-18-documentdb_0.117-0_amd64.deb
```

### RPM (RHEL-compatible 9, PostgreSQL 18, x86_64)

For arm64, replace `x86_64` with `aarch64`; leave `noarch` files unchanged.

```bash
sudo dnf install ./documentdb-18-0.117.0-1.noarch.rpm \
                 ./documentdb-common-0.117.0-1.noarch.rpm \
                 ./documentdb-postgresql-tools-0.117.0-1.noarch.rpm \
                 ./documentdb-gateway-0.117.0-1.el9.x86_64.rpm \
                 ./rhel9-postgresql18-documentdb-0.117.0-1.el9.x86_64.rpm
```

For PostgreSQL 17, replace the two PostgreSQL-specific files with their `17` equivalents; the shared packages stay the same. The `documentdb` meta package always selects PG18.

For extension-only use, install the extension and tools files, then [configure PostgreSQL](#extension-only-installation). For a full stack, [run the wizard](#2-run-the-setup-wizard) with the installed major: use `--pg-version 17` for PG17.

Filename prefixes such as `ubuntu24.04-` and `rhel9-` identify release assets, not package names. DEB extensions use version `0.117-0`, while other packages use `0.117.0`; RPMs use `0.117.0` with release `1` or `1.el9`.

### Offline / air-gapped

Release assets do not include the full dependency set. Stage PostgreSQL, its required extensions, and all other dependencies on a connected machine of the **same distro, release, and architecture**. Include dependencies already installed on the staging machine. Follow [Offline / air-gapped install](https://documentdb.io/docs/linux-packages/offline/) to serve the bundle as a local repository.

## Upgrades and retired targets

**Pre-GA:** in-place package upgrades from earlier releases are not supported yet. Use a clean host or a new, empty PostgreSQL instance and plan data migration separately. Removing packages preserves PostgreSQL data; do not reset or reuse an adopted instance as an upgrade workaround.

The hosted matrix excludes PostgreSQL 15/16, Debian 11/12/13, Ubuntu 22.04, and RHEL-compatible 8. Since v0.116, older packages for retired targets are not carried forward, including PG16 extensions formerly hosted for Ubuntu 24.04 and EL9. Existing installations keep running, but cannot update or reinstall those packages from documentdb.io.

For older installations, use matching [GitHub release assets](https://github.com/documentdb/documentdb/releases), including their dependencies, or consider PGDG and source builds where available. Changing provider or platform requires compatibility and recovery planning; it is not an automatic migration.

Retired URLs retain empty signed metadata so package-manager refreshes do not disrupt unrelated operations. If a host will no longer use first-party packages, remove its DocumentDB repository entry. This removes neither packages nor data; keep PGDG if other packages still use it.

```bash
# Debian / Ubuntu
sudo rm -f /etc/apt/sources.list.d/documentdb.list
sudo apt update

# RHEL-compatible
sudo rm -f /etc/yum.repos.d/documentdb.repo
sudo dnf clean all
```

## Container image

For macOS, Windows, or a container-based installation, follow [DocumentDB Local](../documentdb-local/index.md). Release v0.117 provides Linux amd64/arm64 images for PostgreSQL 15-18; use a versioned tag rather than `latest` when you need a reproducible version.
