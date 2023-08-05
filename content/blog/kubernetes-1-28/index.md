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

TODO

1. ~~Fetch feature info~~
2. ~~Fetch deprecation info~~
3. Write about features
4. Write intro & editor's summary?
5. Publish


Brief summary of release

## The Hot and Cool {#hot}

Hihglight of the coolest new features

## Changes and Deprecations {#deprecations}

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


## API


### [#2340](https://github.com/kubernetes/enhancements/issues/2340) Consistent Reads from Cache {#2340}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `ConsistentListFromCache` **Default:** `false`

This enhancement will improve the performance of some API requests like `GET` or `LIST`, by reading information from the watch cache of etcd, instead of reading it from etcd itself.

This is possible thanks to `WatchProgressRequest` in etcd 3.4+, and will improve performance and scalability hugely on big deployments like 5k+ node clusters.

If you wanna read more about the technical details and how data consistency is ensured, [check the KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/2340-Consistent-reads-from-cache/README.md).


### [#3157](https://github.com/kubernetes/enhancements/issues/3157) Allow informers for getting a stream of data instead of chunking {#3157}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `WatchList` **Default:** `false`

This feature build on top of the previous ([#2340](https://github.com/kubernetes/enhancements/issues/2340)) to improve performance of the API server even further. In particular aiming to reduce memory usage when processing `LIST` requests.

If enabled, whoever wants data from the API server can stream (`WATCH`) the changes as they happen instead of pulling a complete `LIST` each time they need updated info.

If you wanna read more about the technical details, [check the KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-api-machinery/3157-watch-list).

### [#4020](https://github.com/kubernetes/enhancements/issues/4020) Unknown Version Interoperability Proxy {#4020}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `UnknownVersionInteroperabilityProxy` **Default:** `false`

During cluster upgrades, in a cluster with several API servers, it may happen that not every `kube-apiserver` has the data to handle every request. For example, if they are running on different versions.

Starting in Kubernetes 1.28 `kube-apiserver` can proxy resource requests to the right peers. This mechanism is called *Mixed Version Proxy*.

If you wanna read more about the rationale, and the edge cases, [check the KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-api-machinery/4020-unknown-version-interoperability-proxy) and the [documentation](https://github.com/Richabanker/website/blob/bb9c7d9d2457070d0d794a50bfeb07422fa86c98/content/en/docs/concepts/architecture/mixed-version-proxy.md).


### [#4008](https://github.com/kubernetes/enhancements/issues/4008) CRD Validation Ratcheting {#4008}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `CRDValidationRatcheting` **Default:** `false`

One of the main issues when working with [`CustomResourceDefinition`](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions) (CDR) is managing the changes to the definition schemas.

Take this example CDR schema:

```yaml
properties:
    fieldA:
        type: string
    fieldB:
        type: string
```

That we use to deploy the following resource:

```yaml
apiVersion: stable.example.com/v1
kind: MyCRD
fieldA: ""
```

If we update `fieldA` to now require `minLength: 2`, we will get validation errors when working with the existing resources.

For example, if we add a field to our existing resource:

```yaml
apiVersion: stable.example.com/v1
kind: MyCRD
fieldA: ""
fieldB: new field value
```

We will get a validation error, as `fieldA` is not compliant with `minLength: 2`.

This makes sense, but it is very painful and inconvenient. The resource is not compliant, ok, but it's already deployed… ¡Let me work with it!

Starting with Kubernetes 1.28 these kind of edits will be allowed. As long as you don't modify the fields with new validation rules, you'll be able to work with your custom resources without fear of getting unexpected validation errors.

**Related:** [#2876 CRD Validation Expression Language](#2876).

### [#2876](https://github.com/kubernetes/enhancements/issues/2876) CRD Validation Expression Language {#2876}

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `CustomResourceValidationExpressions` **Default:** `true`

This enhancement adds the means to [define validation rules](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema) for custom resources, right next to their definition:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
...
  schema:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          x-kubernetes-validator: 
            - rule: "minReplicas <= maxReplicas"
              message: "minReplicas cannot be larger than maxReplicas"
          type: object
          properties:
            minReplicas:
              type: integer
            maxReplicas:
              type: integer
```


Now validation can take place without the need of webhooks. Less webhooks makes it easier for admins to maintain the cluster.

Also, this information is declared in such a way that it's easy to read by other applications.

This feature has been in **Beta** since Kubernetes 1.25. In Kubernetes 1.28 new `reason` and `fieldPath` fields have been added.

- `reason`: A machine readable message to be returned in case the validation fails.
- `fieldPath`: The fieldPath of the field (i.e. `.foo.test.x`) to be returned when the validation fails. You may want to customize this value on some complex structures, or some cases where you are comparing several fields against each other.


### [#3488](https://github.com/kubernetes/enhancements/issues/3488) CEL for Admission Control {#3488}

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `ValidatingAdmissionPolicy` **Default:** `true`

Building on top of [(#2876) CRD Validation Expression Language](#2876), you can define validation rules for the Kubernetes admission controller using the new [Common Expression Language](https://kubernetes.io/docs/reference/using-api/cel/).

For example, you can define a policy that would fail if a request tries to create an object with more than 5 replicas:

```yaml
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicy
metadata:
  name: "demo-policy.example.com"
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups:   ["apps"]
      apiVersions: ["v1"]
      operations:  ["CREATE", "UPDATE"]
      resources:   ["deployments"]
  validations:
    - expression: "object.spec.replicas <= 5"
```

And then use it on a *binding*, so Kubernetes denies request that doesn't pass this rule.

```yaml
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: "demo-binding-test.example.com"
spec:
  policyName: "demo-policy.example.com"
  validationActions: [Deny]
  matchResources:
    namespaceSelector:
      matchLabels:
        environment: test
```

Read the Kubernetes documentation for more info on the [Validating Admission Policy](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/).


### [#3716](https://github.com/kubernetes/enhancements/issues/3716) CEL-based admission webhook match conditions {#3716}

**SIG group:** sig-api-machinery \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `AdmissionWebhookMatchConditions` **Default:** `true`

This enhancement adds the possibility to fine-tune what requests should be evaluated by a Kubernetes admission controller webhook. 

You can define filtering rules using the new [Common Expression Language (#3488)](#3488) on the new `matchConditions` field of a Webhook:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
webhooks:
…
  matchConditions:
  - name: 'exclude-kubelet-requests'
    expression: '!("system:nodes" in request.userInfo.groups)'
```

If several conditions are present, a request must pass all of them to be evaluated by a webhook.


## Apps


### [#3939](https://github.com/kubernetes/enhancements/issues/3939) Allow for recreation of pods once fully terminated in the job controller {#3939}

**SIG group:** sig-apps \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `JobPodReplacementPolicy` **Default:** `false`

Currently, as soon as a `Job` finishes (either successfully or not) enters a state of termination. Before it is fully terminated, a new `Job` may be created to replace it.

However, this is problematic for some workloads that need the jobs to be fully terminated.

You can now choose this alternate behaviour by setting the `podReplacementPolicy` field to `Failed` (the default being `FailedOrTerminating`).

```yaml
kind: Job
metadata:
  name: new
  ...
spec:
  podReplacementPolicy: Failed
  ...
```


### [#3850](https://github.com/kubernetes/enhancements/issues/3850) Backoff Limit Per Index For Indexed Jobs {#3850}

**SIG group:** sig-apps \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `JobBackoffLimitPerIndex` **Default:** `false`

Currently, you can set a `backoffLimit` on jobs to define the max number of retries before considering the whole `Job` as failed.

You can fine tune this for indexed jobs in Kubernetes 1.28, and set retry limits per index.

Each index can fail up to `backoffLimitPerIndex` times, before being condiered as failed.

You can also set a limit on indexes failed with the `maxFailedIndexes` field.

```yaml
apiVersion: v1
kind: Job
spec:
  parallelism: 10
  completions: 10
  completionMode: Indexed
  backoffLimitPerIndex: 1
  maxFailedIndexes: 10
…
```

In this example, each index can fail up to one time. The moment 10 indexes fail, the whole `Job` gets marked as Failed and the `Job` controller terminates all remaining pods.


### [#4026](https://github.com/kubernetes/enhancements/issues/4026) Add job creation timestamp to job annotations {#4026}

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `CronJobCreationAnnotation` **Default:** `true`

The `CronJob` controller now offers the timestamp when a job is expected t obe running.

```yaml
`batch.kubernetes.io/cronjob-scheduled-timestamp: "2016-05-19T03:00:00-07:00"`
```


### [#3329](https://github.com/kubernetes/enhancements/issues/3329) Retriable and non-retriable Pod failures for Jobs {#3329}

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `JobPodFailurePolicy` **Default:** `true`

This enhancement allows you to use `podFailurePolicy` to control how to manage failures for pods in jobs, and allow retries without counting for the `backoffLimit`.

With the following example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-pod-failure-policy-failjob
spec:
…
  backoffLimit: 6
  podFailurePolicy:
    rules:
    - action: FailJob
      onExitCodes:
        containerName: main
        operator: In
        values: [42]
```

For this specific app, if a job exits with error code `42` we know it's a bug and this job should fail instead of being retied (`action: FailJob`).

In this other case: 

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-pod-failure-policy-ignore
spec:
…
  backoffLimit: 0
  podFailurePolicy:
    rules:
    - action: Ignore
      onPodConditions:
      - type: DisruptionTarget
```

We ignore (`action: Ignore`) the cases where pods fail due to external disruptions (i.e. an admin drains the node and the pods gets evicted). The `Pod` will be retried without counting for the `backoffLimit`.

### [#4017](https://github.com/kubernetes/enhancements/issues/4017) Add Pod Index Label for StatefulSets and Indexed Jobs {#4017}

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `PodIndexLabel` **Default:** `true`

With this featured enabled, the `Job` and `StatefulSet` controllers provide the pod index as a label:

- For jobs: `batch.kubernetes.io/job-completion-index`
- For stateful sets: `apps.kubernetes.io/pod-index`

## Auth


### [#3299](https://github.com/kubernetes/enhancements/issues/3299) KMS v2 Improvements {#3299}

**SIG group:** sig-auth \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#2799](https://github.com/kubernetes/enhancements/issues/2799) Reduction of Secret-based Service Account Tokens {#2799}

**SIG group:** sig-auth \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3325](https://github.com/kubernetes/enhancements/issues/3325) Auth API to get self user attributes {#3325}

**SIG group:** sig-auth \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


## CLI


### [#3895](https://github.com/kubernetes/enhancements/issues/3895) kubectl delete: Add interactive(-i) flag {#3895}

**SIG group:** sig-cli \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#1440](https://github.com/kubernetes/enhancements/issues/1440) kubectl events {#1440}

**SIG group:** sig-cli \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


## Instrumentation


### [#3498](https://github.com/kubernetes/enhancements/issues/3498) Extend metrics stability {#3498}

**SIG group:** sig-instrumentation \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


## Network


### [#3836](https://github.com/kubernetes/enhancements/issues/3836) Kube-proxy improved ingress connectivity reliability {#3836}

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#2681](https://github.com/kubernetes/enhancements/issues/2681) Field status.hostIPs added for Pod {#2681}

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3668](https://github.com/kubernetes/enhancements/issues/3668) Reserve nodeport ranges for dynamic and static allocation {#3668}

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3458](https://github.com/kubernetes/enhancements/issues/3458) Remove transient node predicates from KCCM's service controller {#3458}

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3685](https://github.com/kubernetes/enhancements/issues/3685) Move EndpointSlice Reconciler into Staging {#3685}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3453](https://github.com/kubernetes/enhancements/issues/3453) Minimizing iptables-restore input size {#3453}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3178](https://github.com/kubernetes/enhancements/issues/3178) Cleaning up IPTables Chain Ownership {#3178}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#2595](https://github.com/kubernetes/enhancements/issues/2595) Expanded DNS configuration {#2595}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#1669](https://github.com/kubernetes/enhancements/issues/1669) Proxy Terminating Endpoints {#1669}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


## Nodes


### [#4138](https://github.com/kubernetes/enhancements/issues/4138) [KEP-3085] Add condition for sandbox creation (xposted from original issue) {#4138}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#4033](https://github.com/kubernetes/enhancements/issues/4033) Discover cgroup driver from CRI {#4033}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#4009](https://github.com/kubernetes/enhancements/issues/4009) Add CDI devices to device plugin API {#4009}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3983](https://github.com/kubernetes/enhancements/issues/3983) Add support for a drop-in kubelet configuration directory {#3983}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#753](https://github.com/kubernetes/enhancements/issues/753) Sidecar Containers {#753}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3063](https://github.com/kubernetes/enhancements/issues/3063) dynamic resource allocation {#3063}

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3085](https://github.com/kubernetes/enhancements/issues/3085) Pod conditions around readiness to start containers after completion of pod sandbox creation {#3085}

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#127](https://github.com/kubernetes/enhancements/issues/127) Support User Namespaces in pods {#127}

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3545](https://github.com/kubernetes/enhancements/issues/3545) Improved multi-numa alignment in Topology Manager {#3545}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#2400](https://github.com/kubernetes/enhancements/issues/2400) Node memory swap support {#2400}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3673](https://github.com/kubernetes/enhancements/issues/3673) Kubelet limit of Parallel Image Pulls {#3673}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `FOO` **Default:** `true`


### [#2403](https://github.com/kubernetes/enhancements/issues/2403) Extend podresources API to report allocatable resources {#2403}

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3743](https://github.com/kubernetes/enhancements/issues/3743) graduate the kubelet podresources endpoint to GA {#3743}

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#606](https://github.com/kubernetes/enhancements/issues/606) Support 3rd party device monitoring plugins {#606}

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


## Releases


### [#1731](https://github.com/kubernetes/enhancements/issues/1731) Publishing Kubernetes packages on community infrastructure {#1731}

**SIG group:** sig-release \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


## Storage


### [#1790](https://github.com/kubernetes/enhancements/issues/1790) Support recovery from volume expansion failure {#1790}

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#3762](https://github.com/kubernetes/enhancements/issues/3762) PersistentVolume last phase transition time {#3762}

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `FOO` **Default:** `false`


### [#2268](https://github.com/kubernetes/enhancements/issues/2268) Non-graceful node shutdown {#2268}

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


### [#3333](https://github.com/kubernetes/enhancements/issues/3333) Retroactive default StorageClass assignement {#3333}

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `FOO` **Default:** `true`


## Conclusion

