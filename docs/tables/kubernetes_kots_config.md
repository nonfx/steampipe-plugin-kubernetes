---
title: "Steampipe Table: kubernetes_kots_config - Query KOTS Application Config using SQL"
description: "Allows users to query KOTS application configuration values, including config groups, item types, values, and defaults."
folder: "KOTS"
---

# Table: kubernetes_kots_config - Query KOTS Application Config using SQL

[KOTS](https://kots.io) is a kubectl plugin and admin console for managing Kubernetes Off-The-Shelf (KOTS) software. KOTS applications can expose configurable settings that operators set during installation or updates. These config values control application behavior and are organized into groups.

## Table Usage Guide

The `kubernetes_kots_config` table provides insights into the configuration of a KOTS application. Each row represents a single config item within a config group. Use this table to audit configuration values, compare settings across namespaces, or verify that required config items are properly set.

**Important Notes:**
- You **must** specify `app_slug` in a `where` clause to query this table.
- If `sequence` is not specified, the config for the latest available version is returned.
- This table requires a running kotsadm instance in the cluster. It communicates with kotsadm via port-forwarding to the admin console API on port 3000.
- The Kubernetes user/service account requires the following RBAC permissions in each target namespace:
  - `list` and `get` on `pods` (core API) — to discover kotsadm pods by label `app=kotsadm` and resolve the port-forward target.
  - `create` on `pods/portforward` (core API) — to establish the port-forward tunnel to kotsadm on port 3000.
  - `get` on `secrets` (core API) — to read the `kotsadm-authstring` secret for API authentication.
- For auto-discovery across all namespaces (no `namespace` filter), these permissions must be cluster-wide. For tighter access, use namespace-scoped `Role` + `RoleBinding` instead. See the [KOTS Applications section](/plugins/turbot/kubernetes#kots-applications) in the main docs for RBAC examples and scoping guidance.

## Examples

### List all config items for an application
Get all config values for a KOTS application across all namespaces.

```sql+postgres
select
  item_name,
  item_type,
  item_value,
  item_default,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app';
```

```sql+sqlite
select
  item_name,
  item_type,
  item_value,
  item_default,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app';
```

### List config for a specific version sequence
Get config values for a specific version of the application.

```sql+postgres
select
  item_name,
  item_value,
  item_default,
  group_name
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and sequence = 5;
```

```sql+sqlite
select
  item_name,
  item_value,
  item_default,
  group_name
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and sequence = 5;
```

### Find config items that differ from their defaults
Identify configuration values that have been customized from the default.

```sql+postgres
select
  item_name,
  item_value,
  item_default,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_value != item_default;
```

```sql+sqlite
select
  item_name,
  item_value,
  item_default,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_value != item_default;
```

### List password-type config items
Find sensitive configuration items to audit.

```sql+postgres
select
  item_name,
  group_name,
  item_hidden,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_type = 'password';
```

```sql+sqlite
select
  item_name,
  group_name,
  item_hidden,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_type = 'password';
```

### List read-only config items
Find config items that cannot be modified by the operator.

```sql+postgres
select
  item_name,
  item_value,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_read_only;
```

```sql+sqlite
select
  item_name,
  item_value,
  group_name,
  namespace
from
  kubernetes_kots_config
where
  app_slug = 'my-app'
  and item_read_only = 1;
```
