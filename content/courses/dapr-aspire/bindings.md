---
date: '2025-11-01T06:00:00+05:30'
title: '🖇️ Bindings'
summary: 'DAPR Bindings - Input and Output Bindings'
tags: ['.NET Aspire', 'DAPR', 'CRON', 'Storage']
weight: 6
---

## Input Bindings

### CRON

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: scheduler
  namespace: default
spec:
  type: bindings.cron
  version: v1
  metadata:
  - name: schedule
    value: "@every 20s"
  - name: route
    value: /api/bindings/cron
```

## Output Bindings

### Local Storage

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: storage
spec:
  type: bindings.localstorage
  version: v1
  metadata:
  - name: rootPath
    value: "../Storage"
```
