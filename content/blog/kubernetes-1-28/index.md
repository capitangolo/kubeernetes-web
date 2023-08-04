---
title: "Kubernetes 1.28 - What's new?"
date: 2023-08-01T08:00:00+02:00
author:
 - Víctor Jiménez Cerrada
description: "What's new in Kubernetes 1.28?"
tags:
 - kubernetes
 - kubernetes 1.28
 - releases
categories:
 - Kubernetes Releases
draft: false
---

## TODO

1. Fetch feature info
2. Fetch deprecation info
3. Write about features
4. Write intro & editor's summary?
5. Publish


## Intro

Brief summary of release

## The Hot and Cool

Hihglight of the coolest new features

## Changes and Deprecations

Things that are deprecated and will stop working:
- The `NetworkPolicyStatus` feature.
- `kubectl version` now returns `kubectl version --short`, `--short` flag is removed.
- Remove `KUBECTL_EXPLAIN_OPENAPIV3`.

Things that have changed or are removed and there's an alternative:
- Metrics:
    - `scheduler_scheduler_goroutines` ➜ `scheduler_goroutines`.
- azureFile in-tree storage plugin, use the CSI driver instead.
- `node-role.kubernetes.io/master`: `--non-blocking-taints` no longer in the default value, add it manually if needed.
- `k8s.io/kubernetes/pkg/kubelet/cri/streaming`  ➜ `k8s.io/kubelet/pkg/cri/streaming`.
- KMSv1 ➜ Use KMSv2, or set `--feature-gates=KMSv1=true`.
- Test suites replaced by the new table driven e2e tests:
  - `NetworkPolicyLegacy`.
  - `TestPerPodSchedulingMetrics`.
- kube-scheduler: `--lock-object-namespace` and `--lock-object-name`  ➜ `--leader-elect-resource-namespace` and `--leader-elect-resource-name`.
- `pvc.Status`: `resizeStatus` ➜ `AllocatedResourceStatus`

Things that will stop working soon and you need to start planning for a replacement:
- CephFS in-tree volume plugin: Use the [CephFS CSI driver](https://github.com/ceph/ceph-csi/) instead.
- RBD in-tree volume plugin: Use the [RBD CSI driver](https://github.com/ceph/ceph-csi/) instead.
- [k8s.io/code-generator](k8s.io/code-generator): `generate_groups.sh` and `generate_internal_groups.sh` ➜ `kube_codegen.sh`.
- kube-controller-manager: `--volume-host-cidr-denylist` and `--volume-host-allow-local-loopback` flags.
- Kubelet: `--azure-container-registry-config`  ➜ `--image-credential-provider-config` and `--image-credential-provider-bin-dir`.
- For custom scheduler plugin developers: In `EnqueueExtension`, `EnqueueExtension` [changed some return values](https://github.com/kubernetes/kubernetes/pull/118551).
- Metrics:
    - `apiserver_flowcontrol_request_concurrency_in_use`  ➜ `apiserver_flowcontrol_current_executing_seats`.
    - `apiserver_storage_db_total_size_in_bytes` ➜ `apiserver_storage_size_bytes metric`.

Other minor changes you may want to read more about:
- metrics server bumped to `v0.6.3`.
- metrics server metric-resolution is now `15s`.
- Klog text output now uses JSON as encoding for structs, maps and slices.
- etcd updated to `3.5.9-0`.
- cri-tools updated to `v1.27.0`.
- distroless-iptables bumped to `0.2.6`.

API version changes:
- KubeSchedulerConfiguration: `v1beta2` ➜ `v1`.
- SelfSubjectReview: `v1beta1` ➜ `v1`.
- ValidatingAdmissionPolicy: `v1alpha1` ➜ `v1beta1`.
- ValidatingAdmissionPolicyBinding: `v1alpha1` ➜ `v1beta1`.


## Kubernetes 1.28 API

### [#2340](https://github.com/kubernetes/enhancements/issues/2340) Consistent Reads from Cache

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#4020](https://github.com/kubernetes/enhancements/issues/4020) Unknown Version Interoperability Proxy

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#4008](https://github.com/kubernetes/enhancements/issues/4008) CRD Validation Ratcheting

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3157](https://github.com/kubernetes/enhancements/issues/3157) Allow informers for getting a stream of data instead of chunking

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3716](https://github.com/kubernetes/enhancements/issues/3716) CEL-based admission webhook match conditions

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3488](https://github.com/kubernetes/enhancements/issues/3488) CEL for Admission Control

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#2876](https://github.com/kubernetes/enhancements/issues/2876) CRD Validation Expression Language

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


## Apps in Kubernetes 1.28


### [#3939](https://github.com/kubernetes/enhancements/issues/3939) Allow for recreation of pods once fully terminated in the job controller

**SIG group:** sig-apps \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3850](https://github.com/kubernetes/enhancements/issues/3850) Backoff Limit Per Index For Indexed Jobs

**SIG group:** sig-apps \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#4026](https://github.com/kubernetes/enhancements/issues/4026) Add job creation timestamp to job annotations

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3329](https://github.com/kubernetes/enhancements/issues/3329) Retriable and non-retriable Pod failures for Jobs

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#4017](https://github.com/kubernetes/enhancements/issues/4017) Add Pod Index Label for StatefulSets and Indexed Jobs

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


## Kubernetes 1.28 Auth


### [#3299](https://github.com/kubernetes/enhancements/issues/3299) KMS v2 Improvements

**SIG group:** sig-auth \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#2799](https://github.com/kubernetes/enhancements/issues/2799) Reduction of Secret-based Service Account Tokens

**SIG group:** sig-auth \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3325](https://github.com/kubernetes/enhancements/issues/3325) Auth API to get self user attributes

**SIG group:** sig-auth \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


## CLI in Kubernetes 1.28


### [#3895](https://github.com/kubernetes/enhancements/issues/3895) kubectl delete: Add interactive(-i) flag

**SIG group:** sig-cli \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#1440](https://github.com/kubernetes/enhancements/issues/1440) kubectl events

**SIG group:** sig-cli \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


## Kubernetes 1.28 Instrumentation


### [#3498](https://github.com/kubernetes/enhancements/issues/3498) Extend metrics stability

**SIG group:** sig-instrumentation \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


## Network in Kubernetes 1.28


### [#3836](https://github.com/kubernetes/enhancements/issues/3836) Kube-proxy improved ingress connectivity reliability

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#2681](https://github.com/kubernetes/enhancements/issues/2681) Field status.hostIPs added for Pod

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3668](https://github.com/kubernetes/enhancements/issues/3668) Reserve nodeport ranges for dynamic and static allocation

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3458](https://github.com/kubernetes/enhancements/issues/3458) Remove transient node predicates from KCCM's service controller

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3685](https://github.com/kubernetes/enhancements/issues/3685) Move EndpointSlice Reconciler into Staging

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3453](https://github.com/kubernetes/enhancements/issues/3453) Minimizing iptables-restore input size

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3178](https://github.com/kubernetes/enhancements/issues/3178) Cleaning up IPTables Chain Ownership

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#2595](https://github.com/kubernetes/enhancements/issues/2595) Expanded DNS configuration

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#1669](https://github.com/kubernetes/enhancements/issues/1669) Proxy Terminating Endpoints

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


## Kubernetes 1.28 Nodes


### [#4138](https://github.com/kubernetes/enhancements/issues/4138) [KEP-3085] Add condition for sandbox creation (xposted from original issue)

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#4033](https://github.com/kubernetes/enhancements/issues/4033) Discover cgroup driver from CRI

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#4009](https://github.com/kubernetes/enhancements/issues/4009) Add CDI devices to device plugin API

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3983](https://github.com/kubernetes/enhancements/issues/3983) Add support for a drop-in kubelet configuration directory

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#753](https://github.com/kubernetes/enhancements/issues/753) Sidecar Containers

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3063](https://github.com/kubernetes/enhancements/issues/3063) dynamic resource allocation

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3085](https://github.com/kubernetes/enhancements/issues/3085) Pod conditions around readiness to start containers after completion of pod sandbox creation

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#127](https://github.com/kubernetes/enhancements/issues/127) Support User Namespaces in pods

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3545](https://github.com/kubernetes/enhancements/issues/3545) Improved multi-numa alignment in Topology Manager

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#2400](https://github.com/kubernetes/enhancements/issues/2400) Node memory swap support

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3673](https://github.com/kubernetes/enhancements/issues/3673) Kubelet limit of Parallel Image Pulls

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `True`


### [#2403](https://github.com/kubernetes/enhancements/issues/2403) Extend podresources API to report allocatable resources

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3743](https://github.com/kubernetes/enhancements/issues/3743) graduate the kubelet podresources endpoint to GA

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#606](https://github.com/kubernetes/enhancements/issues/606) Support 3rd party device monitoring plugins

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


## Releases in Kubernetes 1.28


### [#1731](https://github.com/kubernetes/enhancements/issues/1731) Publishing Kubernetes packages on community infrastructure

**SIG group:** sig-release \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


## Kubernetes 1.28 Storage


### [#1790](https://github.com/kubernetes/enhancements/issues/1790) Support recovery from volume expansion failure

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#3762](https://github.com/kubernetes/enhancements/issues/3762) PersistentVolume last phase transition time

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `False`


### [#2268](https://github.com/kubernetes/enhancements/issues/2268) Non-graceful node shutdown

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`


### [#3333](https://github.com/kubernetes/enhancements/issues/3333) Retroactive default StorageClass assignement

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `True`



## Conclusion
