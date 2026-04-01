---
title: "Steampipe Table: kubernetes_kots_version - Query KOTS Application Versions using SQL"
description: "Allows users to query KOTS application version history, including deployment status, release notes, and channel information."
folder: "KOTS"
---

# Table: kubernetes_kots_version - Query KOTS Application Versions using SQL

[KOTS](https://kots.io) is a kubectl plugin and admin console for managing Kubernetes Off-The-Shelf (KOTS) software. Each KOTS application maintains a version history that tracks every release available or deployed to the cluster.

## Table Usage Guide

The `kubernetes_kots_version` table provides insights into the version history of a KOTS application. It auto-discovers all namespaces where kotsadm is running and queries the version history API. Use this table to track deployed versions, identify pending updates, and review release history.

**Important Notes:**
- You **must** specify `app_slug` in a `where` clause to query this table.
- This table requires a running kotsadm instance in the cluster.
- The user must have permission to list pods (label `app=kotsadm`) and read the `kotsadm-authstring` secret in the target namespace(s).

## Examples

### List all versions for an application
Get the full version history for a specific KOTS application across all namespaces.

```sql+postgres
select
  version_label,
  sequence,
  status,
  created_on,
  deployed_at,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
order by
  sequence desc;
```

```sql+sqlite
select
  version_label,
  sequence,
  status,
  created_on,
  deployed_at,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
order by
  sequence desc;
```

### List versions in a specific namespace
Focus on version history for a particular namespace.

```sql+postgres
select
  version_label,
  sequence,
  status,
  deployed_at
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and namespace = 'production';
```

```sql+sqlite
select
  version_label,
  sequence,
  status,
  deployed_at
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and namespace = 'production';
```

### Find the currently deployed version
Identify which version is currently deployed.

```sql+postgres
select
  version_label,
  sequence,
  deployed_at,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and status = 'deployed';
```

```sql+sqlite
select
  version_label,
  sequence,
  deployed_at,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and status = 'deployed';
```

### Find versions that failed to deploy
Identify versions with deployment failures.

```sql+postgres
select
  version_label,
  sequence,
  status,
  created_on,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and status = 'failed';
```

```sql+sqlite
select
  version_label,
  sequence,
  status,
  created_on,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and status = 'failed';
```

### List required versions that have not been deployed
Find required releases that are still pending.

```sql+postgres
select
  version_label,
  sequence,
  status,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and is_required
  and status != 'deployed';
```

```sql+sqlite
select
  version_label,
  sequence,
  status,
  namespace
from
  kubernetes_kots_version
where
  app_slug = 'my-app'
  and is_required = 1
  and status != 'deployed';
```
