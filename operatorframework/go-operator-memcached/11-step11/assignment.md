---
slug: step11
id: eh3yfaykpyt1
type: challenge
title: Deleting the Memcached Custom Resource
tabs:
- title: Terminal 1
  type: terminal
  hostname: container
- title: Terminal 2
  type: terminal
  hostname: container
- title: Visual Editor
  type: code
  hostname: container
  path: /root/projects/memcached-operator
difficulty: basic
timelimit: 300
---
Our Memcached controller creates pods containing OwnerReferences in their `metadata` section. This ensures they will be removed upon deletion of the `memcached-sample` CR.

Observe the OwnerReference set on a Memcached's pod:

```
oc get pods -o yaml | grep ownerReferences -A10
```

Delete the memcached-sample Custom Resource:

```
oc delete memcached memcached-sample
```

Thanks to OwnerReferences, all of the pods should be deleted:

```
oc get pods
```