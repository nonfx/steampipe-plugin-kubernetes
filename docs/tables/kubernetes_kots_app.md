---
title: "Steampipe Table: kubernetes_kots_app - Query KOTS Applications using SQL"
description: "Allows users to query KOTS applications installed in a Kubernetes cluster, including their status, version, and configuration details."
folder: "KOTS"
---

# Table: kubernetes_kots_app - Query KOTS Applications using SQL

[KOTS](https://kots.io) is a kubectl plugin and admin console for managing Kubernetes Off-The-Shelf (KOTS) software. The KOTS admin console provides a web-based UI for managing the lifecycle of applications distributed with Replicated.

## Table Usage Guide

The `kubernetes_kots_app` table provides insights into KOTS applications installed in the cluster. It auto-discovers all namespaces where the KOTS admin console (kotsadm) is running, connects via port-forwarding, and queries the admin console API. Use this table to get an overview of installed applications, their deployment state, license type, and version information.

**Important Notes:**
- This table requires a running kotsadm instance in the cluster. It communicates with kotsadm via port-forwarding to the admin console API on port 3000.
- The Kubernetes user/service account requires the following RBAC permissions in each target namespace:
  - `list` and `get` on `pods` (core API) — to discover kotsadm pods by label `app=kotsadm` and resolve the port-forward target.
  - `create` on `pods/portforward` (core API) — to establish the port-forward tunnel to kotsadm on port 3000.
  - `get` on `secrets` (core API) — to read the `kotsadm-authstring` secret for API authentication.
- For auto-discovery across all namespaces (no `namespace` filter), these permissions must be cluster-wide. For tighter access, use namespace-scoped `Role` + `RoleBinding` instead. See the [KOTS Applications section](/plugins/turbot/kubernetes#kots-applications) in the main docs for RBAC examples and scoping guidance.

## Examples

### List all KOTS applications across all namespaces
Discover all KOTS-managed applications installed in your cluster, including their current state and deployed version.

```sql+postgres
select
  name,
  slug,
  namespace,
  state,
  current_version_label,
  license_type
from
  kubernetes_kots_app;
```

```sql+sqlite
select
  name,
  slug,
  namespace,
  state,
  current_version_label,
  license_type
from
  kubernetes_kots_app;
```

### List applications in a specific namespace
Focus on KOTS applications running in a particular namespace.

```sql+postgres
select
  name,
  slug,
  state,
  current_version_label,
  created_at
from
  kubernetes_kots_app
where
  namespace = 'my-app-namespace';
```

```sql+sqlite
select
  name,
  slug,
  state,
  current_version_label,
  created_at
from
  kubernetes_kots_app
where
  namespace = 'my-app-namespace';
```

### Find applications that are not in a ready state
Identify applications that may be experiencing issues.

```sql+postgres
select
  name,
  namespace,
  state,
  current_version_label
from
  kubernetes_kots_app
where
  state != 'ready';
```

```sql+sqlite
select
  name,
  namespace,
  state,
  current_version_label
from
  kubernetes_kots_app
where
  state != 'ready';
```

### List airgapped applications
Find applications deployed in airgap mode.

```sql+postgres
select
  name,
  namespace,
  current_version_label,
  license_type
from
  kubernetes_kots_app
where
  is_airgap;
```

```sql+sqlite
select
  name,
  namespace,
  current_version_label,
  license_type
from
  kubernetes_kots_app
where
  is_airgap = 1;
```
