---
title: "Kubernetes 1.28 - What's new?"
slug: "kubernetes-1-28"
date: 2023-08-08T08:00:00+02:00
author:
 - Víctor Jiménez Cerrada
description: "Sidecar containers, Job optimizations, Better Proxys"
tags:
 - kubernetes
 - kubernetes 1.28
 - releases
categories:
 - Kubernetes Releases
draft: false
---

**Kubernetes 1.28** will be out soon, and it brings 44 new or improved enhancements:

- **19** are new or improved **Alpha** enhancements that you can start testing.
- **14** are **Beta** enhancements, enabled by default from now on.
- **11** enhancements are considered **Stable**, and ready for prime time.

This version bring exciting quality of life improvements. Let's tap into them. 

## The Hot and Cool {#hot}

Four trends and enhancements catched our eye.

### [#753 Sidecar Container](#753)

It's been years since **the sidecar pattern** started being promoted as a useful practice. Now it finally is a first-class citizen in Kubernetes.

Sidecars are auxiliary containers that sit along your main workloads to perform a bunch of tasks. For example, providing metrics, pre-fetching secrets, implementing service meshes, and more.

In Kubernetes 1.28 implementing a sidecar container is easier and more reliable. Kubernetes workloads will be more reliable as a result, making dev, admins, and users happier.

[Read about the technical details of this feature](#753).


### Optimization on Jobs

Jobs got a lot of love in this release.

Kubernetes jobs can start a big number of repetitive paralell tasks at once, which is ideal for Machine Learning workloads. You can see how Kubernetes is becoming a big player in this area, thanks to their continuous improvements in tools like jobs.

We already discussed [#753 Sidecar Container](#753). This feature has a few  surprises for job users, like making sure that sidecars won't block a job completion.

The [#3329 Retriable and non-retriable Pod failures for Jobs](#3329) and [#3850	Backoff Limit Per Index For Indexed Jobs](#3850) enhancements will provide more granularity to handle failures on jobs. Some failures are temporary or expected, and handling them differently will prevent the failure of a whole job.

Finally [#3939 Allow for recreation of pods once fully terminated in the job controller](#3939), provides more control options when handling jobs that have finished. For sure, helpful to avoid some edge cases and race conditions.

Again, improvements on Kubernetes jobs keep coming version after version, showing how much interest the (Machine Learning) community have on this feature.


### Rolling Upgrades

Three new enhancements will make upgrades more reliable, and will reduce downtime. Definitely a quality of live improvement for administrators, for whom leaving the app in maintenance mode is a big fear.

With [#4020 Unknown Version Interoperability Proxy](#4020), rolling upgrades of the cluster components will be better handled. Rolling upgrades meaning that not all the same components are upgraded at once, rather one by one, keeping old and new coexisting. In this case, when traffic is sent to a Kubernetes component that is down, it will be redirected to a peer that is ready.

Finally, [#3836 Kube-proxy improved ingress connectivity reliability](#3836) and [#1669 Proxy terminating endpoints](#1669) will reduce the amount of killed connections when doing rolling upgrades. When a `Pod` is terminated to leave room for the new version, all its connections are killed too, leaving customers unhappy. With these enhancements, these connections will be let alone, letting the `Pod` terminate gracefully.

Maintenance is a bit less scary with Kubernetes 1.28.


### [#1731 Publishing Kubernetes Packages on Community Infrastructure](#1731)

The Kubernetes project keeps on their efforts to decouple from Google's infrastructure, the Kubernetes project is providing community owned repositories for their deb and yum packages.

Definitely a sign of maturity that will make the community stronger and will drive more contributors to the project.


## API


### [#2340](https://github.com/kubernetes/enhancements/issues/2340) Consistent Reads from Cache {#2340}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `ConsistentListFromCache` **Default:** `false`

This enhancement will improve the performance of some API requests like `GET` or `LIST`, by reading information from the watch cache of etcd, instead of reading it from etcd itself.

This is possible thanks to `WatchProgressRequest` in etcd 3.4+, and will improve performance and scalability hugely on big deployments like 5k+ node clusters.

If you wanna read more about the technical details and how data consistency is ensured, [tap into the KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/2340-Consistent-reads-from-cache/README.md).


### [#3157](https://github.com/kubernetes/enhancements/issues/3157) Allow Informers for Getting a Stream of Data Instead of Chunking {#3157}

**SIG group:** sig-api-machinery \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `WatchList` **Default:** `false`

This feature build on top of the previous ([#2340](#2340) Consistent Reads from Cache) to improve performance of the API server even further. In particular aiming to reduce memory usage when processing `LIST` requests.

If enabled, whoever wants data from the API server can stream (`WATCH`) the changes as they happen instead of pulling a complete `LIST` each time they need updated info.

If you wanna read more about the technical details, [tap into the KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-api-machinery/3157-watch-list).

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


### [#3716](https://github.com/kubernetes/enhancements/issues/3716) CEL-Based Admission Webhook Match Conditions {#3716}

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


### [#3939](https://github.com/kubernetes/enhancements/issues/3939) Allow for Recreation of Pods Once Fully Terminated in The Job Controller {#3939}

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


### [#4026](https://github.com/kubernetes/enhancements/issues/4026) Add Job Creation Timestamp to Job Annotations {#4026}

**SIG group:** sig-apps \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `CronJobCreationAnnotation` **Default:** `true`

The `CronJob` controller now offers the timestamp when a job is expected t obe running.

```yaml
`batch.kubernetes.io/cronjob-scheduled-timestamp: "2016-05-19T03:00:00-07:00"`
```


### [#3329](https://github.com/kubernetes/enhancements/issues/3329) Retriable and Non-Retriable Pod Failures for Jobs {#3329}

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
**Feature Gate:** `KMSv2` **Default:** `true`

This enhancement comprises the work to create a v2 of the Key Management Service (KMS), to solve some of the main issues of v1.

The problems being:
1. **Performance:** When starting a cluster, it needs to fetch and decrypt all the resources to fill the `kube-apiserver` cache. This may take some time, and hit some API rate limits, delaying the cluster startup.
2. **Key rotation:** In v1, this is manual (error prone) process.
3. **Health check and status:** In v1, it is required to perform encrypted calls to KMS plugins on performance checks.
4. **Observability:** The lack of trace IDs makes it hard to correlate and investigate events across logs.

V2 addresses this by:
1. A new key hierarchy has been implemented to reduce network request to the remote vault.
2. Extra metadata in KMS plugins allows `kube-apiserver` to rotate keys without a restart.
3. A new status API can be used by KMS plugins to provide health information to the API server.
4. A new `UID` field is available on `EncryptRequest` and `DecryptRequest`.

If you wanna tap into more detail, check this article in the Kubernetes.io blog: [Kubernetes 1.25: KMS V2 Improvements](https://kubernetes.io/blog/2022/09/09/kms-v2-improvements/).


### [#2799](https://github.com/kubernetes/enhancements/issues/2799) Reduction of Secret-based Service Account Tokens {#2799}

**SIG group:** sig-auth \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `LegacyServiceAccountTokenTracking` **Default:** `true`
**Feature Gate:** `LegacyServiceAccountTokenCleanUp` **Default:** `true`

Since Kubernetes 1.22, pods' service accounts can obtain tokens from the [TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/) instead of auto generating them.

This enhancement comprises the work to reduce the old style auto-generated tokens.

In a first step (Kubernetes 1.24) the auto creation of secrets for service accounts was disabled, and users were encouraged to use the new API.

Now, Kubernetes 1.28 will purge unused auto-generated tokens for these accounts. Tokens are considered unused after one year by default.


### [#3325](https://github.com/kubernetes/enhancements/issues/3325) Auth API to Get Self User Attributes {#3325}

**SIG group:** sig-auth \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `APISelfSubjectReview` **Default:** `true`

This API provides information about who are you authenticated as, either if you are a user, or a `ServiceAccount`:

```json
{
  "apiVersion": "authentication.k8s.io/v1alpha1",
  "kind": "SelfSubjectReview",
  "status": {
    "userInfo": {
      "name": "jane.doe",
      "uid": "b6c7cfd4-f166-11ec-8ea0-0242ac120002",
      "groups": [
        "viewers",
        "editors",
        "system:authenticated"
      ],
      "extra": {
        "provider_id": ["token.company.example"]
      }
    }
  }
}
```

You can also use a CLI command to get this information:

```bash
kubectl alpha auth whoami
```

This can be useful to troubleshoot authentication issues.


## CLI


### [#3895](https://github.com/kubernetes/enhancements/issues/3895) Kubectl Delete: Add Interactive(-i) Flag {#3895}

**SIG group:** sig-cli \
**Stage:** Net New to **Alpha**

When this new `-i` or `--interactive` flag is present on `kubectl delete`, the following will happen:

- The user will be shown a preview of the objects to be deleted.
- The user will be able to confirm or cancel the deletion.

```bash
$ kubectl delete --interactive -f test_deployment.yaml 
You are about to delete the following 2 resource(s):
Deployment/dockerd
Deployment/dockerd2
Do you want to continue? (y/n): y
deployment.apps "dockerd" deleted
deployment.apps "dockerd2" deleted
```


### [#1440](https://github.com/kubernetes/enhancements/issues/1440) Kubectl Events {#1440}

**SIG group:** sig-cli \
**Stage:** Graduating to **Stable**

The new `kubectl events` is created to replace `kubectl get events` to scape some of the limitations of the `kubectl get` command.

Listing events is a bit more different that other type of objects. For example you may want more control on the `--watch` option, or you may want to do advanced filtering.

In Kubernetes 1.28 this new command is now stable.


## Instrumentation


### [#3498](https://github.com/kubernetes/enhancements/issues/3498) Extend Metrics Stability {#3498}

**SIG group:** sig-instrumentation \
**Stage:** Graduating to **Beta**

Originally metrics were classified only between `stable` and `alpha`.

You could check for this classification, and you would know that if a metric is `stable` it would be safe to use. However, if it was `alpha`, you knew its definition could change alogn the way breaking things.

This enhancement adds two new stability levels:

- `Internal`: Low-level metrics which do not correspod to features, and don't really provide actionable information to admins.
- `Beta`: Metrics that are a bit more mature than `alpha`, but are not `stable` yet.

The `beta` metrics offer more security than `alpha` ones in the sense that they won't be removed without being deprecated first.

They also offer forward compatibility, meaning that the alerts and queries that you create with them will still work in the future.


## Network


### [#3836](https://github.com/kubernetes/enhancements/issues/3836) Kube-proxy Improved Ingress Connectivity Reliability {#3836}

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `KubeProxyDrainingTerminatingNodes` **Default:** `false`

This enhancement adds some features to `kube-proxy` to better manage the health of connections.

In particular:

1. Once a node is terminating, `kube-proxy` won't immediately terminate all the connections and will let them terminate gracefully.
2. A new `/livez` path is added where vendors and users can define a `livenessProbe` to determine `kube-proxy` health. This method is more specific than just checking if the node is terminating.
3. Provide guidelines for vendors to implement those health checks (Aligning them into a standard is not a goal at this stage).

**Related:**
- [#3458 Remove transient node predicates from KCCM's service controller](#3458).
- [#1669 Proxy Terminating Endpoints](#1669).


### [#2681](https://github.com/kubernetes/enhancements/issues/2681) Field status.hostIPs Added for Pod {#2681}

**SIG group:** sig-network \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `PodHostIPs` **Default:** `false`

Currently pods can check their host IP on the `status.hostIP` field. However, this field can only store one IP, and for some time Kubernetes allows for a [dual stack setup (both IPv4 and IPv6)](https://kubernetes.io/docs/concepts/services-networking/dual-stack/
).

A new `status.hostIPs` (note the final s) field has been added in Kubernetes 1.28 that will contain an array with all the host IPs.

```bash 
MY_HOST_IPS=fd00:10:20:0:3::3,10.20.3.3
```


### [#3668](https://github.com/kubernetes/enhancements/issues/3668) Reserve NodePort Ranges for Dynamic and Static Allocation {#3668}

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `ServiceNodePortStaticSubrange` **Default:** `true`

When using a `NodePort` service, you may want to allocate a port statically, manually defining which port to use within the `service-node-port-range` (default `30000-32767`).

However, you may find that your port has been already assigned dynamically to another service.

This new feature reserves the first ports in `service-node-port-range` to be assigned statically. 

The number of reserved ports is defined by this formula `min(max(16, nodeport-size / 32), 128)`, which returns a number between 16 and 128.

For example, for the default range (`30000-32767`), it returns `86` ports. The range `30000-30085` will be reserved for static allocations, the rest for dynamic allocations.

If you wanna learn more, check the Kubernetes.io blog [Kubernetes 1.27: Avoid Collisions Assigning Ports to NodePort Services](https://kubernetes.io/blog/2023/05/11/nodeport-dynamic-and-static-allocation/).


### [#3458](https://github.com/kubernetes/enhancements/issues/3458) Remove Transient Node Predicates from KCCM's Service Controller {#3458}

**SIG group:** sig-network \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `StableLoadBalancerNodeSet` **Default:** `true`

This enhancement will change the way in which nodes are removed from the load balancer's node.

Currently nodes are removed as soon as they are marked as terminating, either because they get the `ToBeDeletedByClusterAutoscaler` taint, or when the node is `NotReady`.

This triggers a chain of events that is not desirable in some cases:

- Connections are terminated immediately.
- load balancers are re-synced, which can cause too many API calls that triggers some rate-limits on cloud providers.

In two cases this behavior is not desired:

- When a node is temporarily not ready.
- When a node is terminating, and we want for connections to end gracefully.

For those cases you can enable this feature. Then nodes won't be removed from the load balancer's node until fully deleted in the API server.

**Related:**
- [#3836 Kube-proxy improved ingress connectivity reliability](#3836).
- [#1669 Proxy Terminating Endpoints](#1669).

### [#3685](https://github.com/kubernetes/enhancements/issues/3685) Move EndpointSlice Reconciler into Staging {#3685}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable**

This enhancement comprises the work done to expose the `EndpointSlice` reconciler logic, so it can be used in custom `Endpoint` controllers.

This means refactor the code to move it from `pkg/controller/endpointslice` into a separate Go module `staging/src/k8s.io/endpointslice`. This new module will be easy to import into other projects.

If you want to read the implementation details, [tap into the KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-network/3685-endpointslice-reconciler-to-staging).


### [#3453](https://github.com/kubernetes/enhancements/issues/3453) Minimizing iptables-restore Input Size {#3453}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `MinimizeIPTablesRestore` **Default:** `true`

This enhancement aims to improve `kube-proxy` performance in iptables mode. It will do so by changing the way `iptables-restore` works.

Now the calls to `iptables-restore` won't need to include the rules that haven't changed.

If you want to read the implementation details, [tap into the KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-network/3453-minimize-iptables-restore/README.md).


### [#3178](https://github.com/kubernetes/enhancements/issues/3178) Cleaning up IPTables Chain Ownership {#3178}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `IPTablesOwnershipCleanup` **Default:** `true`

Some components of Kubernetes, like `kubelet` and `kube-proxy`, create `iptables` chains for their internal usage.

Those weren't meant to be used by third-party components, but they were sometimes.

With the [deletion of dockershim](https://kubernetes.io/blog/2022/02/17/dockershim-faq/) and other code cleaning some of these chains are not needed and are being removed.

If you wanna read more about the rationale behind this change, check out the Kubernetes.io blog [Kubernetes’s IPTables Chains Are Not API](https://kubernetes.io/blog/2022/09/07/iptables-chains-not-api/).


### [#2595](https://github.com/kubernetes/enhancements/issues/2595) Expanded DNS Configuration {#2595}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `ExpandedDNSConfig` **Default:** `true`

This enhancement reduces some limits for the DNS configuration.

In particular allows for more DNS search paths, and longer DNS search paths lists.

- `MaxDNSSearchPaths` increases from `6` to `32`.
- `MaxDNSSearchListChars` increases from `256` to `2048`.


### [#1669](https://github.com/kubernetes/enhancements/issues/1669) Proxy Terminating Endpoints {#1669}

**SIG group:** sig-network \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `ProxyTerminatingEndpoints` **Default:** `true`

This enhancement improves how `kube-proxy` sends traffic to `Endpoint`s that are terminating. It prioritizes `Ready` endpoints that are not terminating, and if all endpoints are terminating, it prioritizes those that are still `Ready` or `Serving`.

This aims to improve reliability in situations like rolling updates, where all endpoints may be terminating at some point. 

If you wanna read more about this feature, check out the Kubernetes.io blog [Advancements in Kubernetes Traffic Engineering](https://kubernetes.io/blog/2022/12/30/advancements-in-kubernetes-traffic-engineering/).

**Related:**
- [#3458 Remove transient node predicates from KCCM's service controller](#3458).
- [#3836 Kube-proxy improved ingress connectivity reliability](#3836).


## Nodes

### [#4138](https://github.com/kubernetes/enhancements/issues/4138) [KEP-3085] Add Condition for Sandbox Creation {#4138}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `PodReadyToStartContainersCondition` **Default:** `false`

Starting with Kubernetes 1.28 there will be a new `Pod` `status.conditions` available: `PodReadyToStartContainers`.

This condition will be `true` when the `Pod`:
- Has a sandbox in the container runtime.
- The sandbox has networking configured.

This extra info may be useful in some cases:
- For cluster admins, to track metrics and calculate how long it takes to start pods.
- In future implementations of Kubernetes controllers, this information may be useful to tweak and optimize their behavior.


### [#4033](https://github.com/kubernetes/enhancements/issues/4033) Discover cgroup Driver from CRI {#4033}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `KubeletCgroupDriverFromCRI` **Default:** `false`

Currently you configure the cgroup driver in two places:
- Your container runtime.
- In `kubelet`.

If this configuration mismatches can lead to hard to troubleshoot issues.

Starting with Kubernetes 1.28 you can let `kubelet` detect what is the cgroup driver that the container is using.

For this detection to work, the container runtime must supports the `RuntimeConfig` CRI call. If not, `kubelet` will fall back to the `cgroupDriver` configuration setting.


### [#3983](https://github.com/kubernetes/enhancements/issues/3983) Add Support for a Drop-in kubelet Configuration Directory {#3983}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `ElasticIndexedJob` **Default:** `false`
**Env variable:** `KUBELET_CONFIG_DROPIN_DIR_ALPHA`

In Kubernetes 1.28 you can set up a configuration directoy, similar to what is possible right now with `kubelet`.

You can add the `--config-dir` CLI option, with files ended in `.conf` and content like:

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "192.168.0.8"
```

This can be useful to separate a single configuration into several logical groups. For example having separate config files for authentication and DNS.

Keep in mind that for this feature to work, you must also set the environment variable `KUBELET_CONFIG_DROPIN_DIR_ALPHA` to any value.

### [#753](https://github.com/kubernetes/enhancements/issues/753) Sidecar Containers {#753}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `SidecarContainers` **Default:** `false`

The [sidecar pattern has been around for a long time](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/#example-1-sidecar-containers), and it's been used for service meshes, gathering metrics, fetching secrets, and more. However, implementing it hasn't been trivial, or hassle free.

Up until now there wasn't a way to say "hey, this is a sidecar container". As a result, sidecar containers could be killed before the main container finishes, or stay alive after messing up with jobs.

Now you can set a new `restartPolicy` field to `Always` on an init container, and Kubernetes will handle it differently:


- The `kubelet` won't wait until the container has finished, instead it will only wait until the startup has completed.
- It will be restarted, unless the sidecar fails during startup and the `Pod` `restartPolicy` is `Never`. In this case the whole pod will fail.
- The sidecar container will keep running for as long as the main container runs.
- The sidecar container will be terminated once all regular containers complete. In this way, they won't prevent jobs from completing after the main container has finished.

```yaml
kind: Pod
…
spec:
  initContainers:
  - name: vault-agent
    image: hashicorp/vault:1.12.1
  - name: istio-proxy
    image: istio/proxyv2:1.16.0
    args: ["proxy", "sidecar"]
    restartPolicy: Always
  containers:
…
```


### [#4009](https://github.com/kubernetes/enhancements/issues/4009) Add CDI Devices to Device Plugin API {#4009}

**SIG group:** sig-node \
**Stage:** Net New to **Alpha** \
**Feature Gate:** `DevicePluginCDIDevices` **Default:** `false`

This enhancement builds upon [#3063 dynamic resource allocation](#3063), and allows device plugin authors to forward request to the container runtimes.


### [#3063](https://github.com/kubernetes/enhancements/issues/3063) Dynamic Resource Allocation {#3063}

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `DynamicResourceAllocation ` **Default:** `false`

When dynamic resource allocation is enabled, pods can define what resources they need beyond CPU and memory.

This is useful in edge setups, where there may be deployed sensors or other devices; and in ML workloads, where they need access to a GPU.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-gpu
spec:
  containers:
  - name: container-with-gpu
    image: ubuntu:22.04
    resources:
      claims:
      - name: gpu-1
  resourceClaims:
  - name: gpu-1
    source:
      resourceClaimTemplateName: resources-gpu
```

Dynamic resource allocation is the name of this new API for requesting and sharing resources: `resource.k8s.io/v1alpha2`. Resources will need their own specific driver.

Although this feature was introduced in Kubernetes 1.26, it continues to evolve on each version as an Alpha.

**Related:** [#4009 Add CDI devices to device plugin API](#4009)


### [#127](https://github.com/kubernetes/enhancements/issues/127) Support User Namespaces in Pods {#127}

**SIG group:** sig-node \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `UserNamespacesStatelessPodsSupport` **Default:** `false`

By adding support for user namespaces in Kubernetes, you'll be able to run processes in pods with a different user than in the host.

For example, a process in a `Pod` that runs as root may be running as an unprivileged user in the host. If that process is compromised, and escapes the container, the damage it can make will be limited.

For now, this is a Linux only feature, and it relies on idmap mounts on the filesystems used. You will need:

- At least a Linux 6.3 Kernel.
- A filesystem that supports idmap (i.e. ext4, btrfs, xfs, fat, …).
- A container runtime with support for idmap (i.e. CRI-O 1.25, containerd 1.7)


### [#3545](https://github.com/kubernetes/enhancements/issues/3545) Improved Multi-Numa Alignment in Topology Manager {#3545}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `TopologyManagerPolicyOptions` **Default:** `true`
**Feature Gate:** `TopologyManagerPolicyBetaOptions` **Default:** `true`

When execution latency is a concern, you need to be aware of what CPU cores your workloads are running on.

Server tend to have multi socket systems, and modern AMD CPUs are made of several chips on the same package. In those system those cores don't share an L2 cache, but rather an L3 or L4 one. This means that if your workload jumps from one core to another, you need extra time to move their context as well.

This enhancement makes Kubernetes aware of these topologies, so it tries to place workloads in cores that are closer to each other.

You enable this behavior by setting the `prefer-closest-numa-nodes` policy option on the `TopologyManager`.

If you want to read more on the implementation details and the mathematics of this, [tap into the KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/3545-improved-multi-numa-alignment/README.md). You can also get extra info [in the Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/#topology-manager-policy-options)


### [#2400](https://github.com/kubernetes/enhancements/issues/2400) Node Memory Swap Support {#2400}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \
**Feature Gate:** `NodeSwap` **Default:** `true`

Kubernetes support for swap on Linux nodes enters the `beta` stage.

To enable it you need to:
- Disable the `failSwapOn` setting.
- Enable the `NodeSwap` feature gate.
- Define the `swapBehavior`.

```yaml
---
apiVersion: "kubeadm.k8s.io/v1beta3"
kind: InitConfiguration
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
failSwapOn: false
featureGates:
  NodeSwap: true
memorySwap:
  swapBehavior: LimitedSwap
```

You can choose two different strategies for `swapBehavior`:

- `UnlimitedSwap`: The default option. Kubernetes workloads can use as much swap memory as they want (within the system limits).
- `LimitedSwap`: Only Pods with [Burstable QoS](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/#burstable) can use swap memory.

If you are interested, you can get more details on [this article from Kubernetes 1.22](https://kubernetes.io/blog/2021/08/09/run-nodes-with-swap-alpha/). Stay tuned, an updated blog article is being prepared for shortly after the Kubernetes 1.28 release.


### [#3673](https://github.com/kubernetes/enhancements/issues/3673) Kubelet Limit of Parallel Image Pulls {#3673}

**SIG group:** sig-node \
**Stage:** Graduating to **Beta** \

By default, `kubelet` downloads images one after another. You can disable the `kubelet` config `serialize-image-pulls` option, so images can be downloaded in paralell.

This enhancement adds control over this paralell download of images.

You can set the `maxParallelImagePulls` option to define how many images a node can download at the same time.

By default this option is set to `0`, meaning there is no limit.


### [#2403](https://github.com/kubernetes/enhancements/issues/2403) Extend PodResources API to Report Allocatable Resources {#2403}

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `KubeletPodResourcesGetAllocatable` **Default:** `true`

This endpoint to fetch information on the allocatable compute resources on a node is finally stable.

The information offered by the `GetAllocatableResources` gRPC endpoint, combined with the on from the pod resources API endpoint, allows to calculate a node's capacity.

Get more information [on the Kubernetes documentation](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/#grpc-endpoint-getallocatableresources).


### [#3743](https://github.com/kubernetes/enhancements/issues/3743) Graduate the Kubelet PodResources Endpoint to GA {#3743}

**SIG group:** sig-node \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `KubeletPodResources` **Default:** `true`

This enhancement comprises the work to bring the [device plugin monitoring service](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/) ([#606 Support 3rd party device monitoring plugins](https://github.com/kubernetes/enhancements/issues/606)) to stable in Kubernetes 1.28.

It allows to monitor what resources are provided by device plugins, the mapping to what pods are using them, and other metrics.


## Releases


### [#1731](https://github.com/kubernetes/enhancements/issues/1731) Publishing Kubernetes Packages on Community Infrastructure {#1731}

**SIG group:** sig-release \
**Stage:** Net New to **Alpha**

Following on the [recent change on image repositories](https://kubernetes.io/blog/2023/03/10/image-registry-redirect/), and aiming to decouple the project from Google's infrastructure, the Kubernetes project is providing community owned repositories for their deb and yum packages.

The new repository `pkgs.k8s.io` adds to the existing `apt.kubernetes.io` and `yum.kubernetes.io`. The old ones will be deprecated at some point in the future.

Stay tuned to the [Kubernetes.io blog](https://kubernetes.io/blog/), a new post will be published around release time with instructions to migrate to the new repos.


## Storage


### [#1790](https://github.com/kubernetes/enhancements/issues/1790) Support Recovery From Volume Expansion Failure {#1790}

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `RecoverVolumeExpansionFailure ` **Default:** `false`

For some time it's been possible to [recover from a failure when expanding a volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#recovering-from-failure-when-expanding-volumes). For example, if you requested a size bigger than the CSI driver allows, you were able to recover by requesting a smaller size (bigger than the original size before the increase).

In Kubernetes 1.28, the `pvc.Status.ResizeStatus` which contained information about the resizing process has changed. It has been renamed to `pvc.Status.AllocatedResourceStatus`, and now is a map able to contain multiple resources and their status.


### [#3762](https://github.com/kubernetes/enhancements/issues/3762) PersistentVolume Last Phase Transition Time {#3762}

**SIG group:** sig-storage \
**Stage:** Graduating to **Alpha** \
**Feature Gate:** `PersistentVolumeLastPhaseTransitionTime` **Default:** `false`

The persistent volume status now contains a `lastPhaseTransitionTime` field with the timestamp for the last phase transition of the volume.

For example, when a volume is left unclaimed and changes phase to `Released`.

This field, besides providing information to admins, will be very valuable for performance tests. It will also be useful when investigating issues like the data loss some user have experienced when changing the retain policy from `Delete` to `Retain`.

### [#2268](https://github.com/kubernetes/enhancements/issues/2268) Non-graceful Node Shutdown {#2268}

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `NodeOutOfServiceVolumeDetach` **Default:** `true`

This enhancement allows a Kubernetes cluster to prepare for a shutdown, terminate the pods gracefully, properly releasing all the resources.

You can do this by adding the `out-of-service` taint on the node:

```bash
kubectl taint nodes <node-name> node.kubernetes.io/out-of-service=nodeshutdown:NoExecute
```

In Kubernetes 1.28, some minor improvements were made to this feature:

- The `force_delete_pods_total` and `force_delete_pod_errors_total` metrics now account for all forceful pod deletions. Also, the metric will include the reason why the pod is deleted (terminated, orphaned, terminating due to `out-of-service`, or terminating and unscheduled).
- The `attachdetach_controller_forced_detaches` also contains a reason to know if the detach is caused by `out-of-service` or a timeout.

Stay tuned to the [Kubernetes.io blog](https://kubernetes.io/blog/), a new post will be published around release time with more info.


### [#3333](https://github.com/kubernetes/enhancements/issues/3333) Retroactive Default StorageClass Assignement {#3333}

**SIG group:** sig-storage \
**Stage:** Graduating to **Stable** \
**Feature Gate:** `RetroactiveDefaultStorageClass` **Default:** `true`

Currently if you create a PersistentVolumeClaim without specifying a `storageClassName` for the new PVC. If you don't have a default class defined, your PVC will remain undefined, and you can go back and change this value later on.

This enhancement changes this behaviour so that when you create a default `StorageClass`, it will be applied to all PVCs without a defined class.

## Other Changes and Deprecations {#deprecations}

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
- [k8s.io/code-generator](https://github.com/kubernetes/code-generator): `generate_groups.sh` and `generate_internal_groups.sh` ➜ `kube_codegen.sh`.
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



## Conclusion

Thanks for catching up on the latest Kubernetes version with us!

If you are interested in Kubernetes you may consider:

- [Reading the Kubernetes blog](https://kubernetes.io/blog/).
- [Tapping into the Kubernetes Slack](https://kubernetes.slack.com/).
- [Contributing to the Kubernetes project](https://www.kubernetes.dev/docs/guide/).
