# Memcached

> [Memcached](https://memcached.org/) is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.

## Azure-ready Charts with Containers from marketplace.azurecr.io

This Helm Chart has been configured to pull the Container Images from the Azure Marketplace Public Repository.
The following command allows you to download and install all the charts from this repository.
```bash
$ helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
```
## TL;DR

```console
$ helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
$ helm install my-release bitnami-azure/memcached
```

## Introduction

This chart bootstraps a [Memcached](https://github.com/bitnami/bitnami-docker-memcached) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

Bitnami charts can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters. This Helm chart has been tested on top of [Bitnami Kubernetes Production Runtime](https://kubeprod.io/) (BKPR). Deploy BKPR to get automated TLS certificates, logging and monitoring for your applications.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.0-beta3+

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
$ helm install my-release bitnami-azure/memcached
```

These commands deploy Memcached on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

The following tables lists the configurable parameters of the Memcached chart and their default values.

| Parameter                                | Description                                                                                            | Default                                                      |
|------------------------------------------|--------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| `global.imageRegistry`                   | Global Docker image registry                                                                           | `nil`                                                        |
| `global.imagePullSecrets`                | Global Docker registry secret names as an array                                                        | `[]` (does not add image pull secrets to deployed pods)      |
| `image.registry`                         | Memcached image registry                                                                               | `docker.io`                                                  |
| `image.repository`                       | Memcached Image name                                                                                   | `bitnami/memcached`                                          |
| `image.tag`                              | Memcached Image tag                                                                                    | `{TAG_NAME}`                                                 |
| `image.pullPolicy`                       | Memcached image pull policy                                                                            | `IfNotPresent`                                               |
| `image.pullSecrets`                      | Specify docker-registry secret names as an array                                                       | `[]` (does not add image pull secrets to deployed pods)      |
| `nameOverride`                           | String to partially override memcached.fullname template with a string (will prepend the release name) | `nil`                                                        |
| `fullnameOverride`                       | String to fully override memcached.fullname template with a string                                     | `nil`                                                        |
| `clusterDomain`                          | Kubernetes cluster domain                                                                              | `cluster.local`                                              |
| `architecture`                           | Memcahed architecture. Allowed values: standalone or high-availability                                 | `standalone`                                                 |
| `replicaCount`                            | Number of containers                                                                                   | `1`                                                          |
| `extraEnv`                               | Additional env vars to pass                                                                            | `{}`                                                         |
| `arguments`                              | Arguments to pass                                                                                      | `["/run.sh"]`                                                |
| `memcachedUsername`                      | Memcached admin user                                                                                   | `nil`                                                        |
| `memcachedPassword`                      | Memcached admin password                                                                               | `nil`                                                        |
| `service.type`                           | Kubernetes service type for Memcached                                                                  | `ClusterIP`                                                  |
| `service.port`                           | Memcached service port                                                                                 | `11211`                                                      |
| `service.clusterIP`                      | Specific cluster IP when service type is cluster IP. Use `None` for headless service                   | `nil`                                                        |
| `service.nodePort`                       | Kubernetes Service nodePort                                                                            | `nil`                                                        |
| `service.loadBalancerIP`                 | `loadBalancerIP` if service type is `LoadBalancer`                                                     | `nil`                                                        |
| `service.annotations`                    | Additional annotations for Memcached service                                                           | `{}`                                                         |
| `resources.requests`                     | CPU/Memory resource requests                                                                           | `{memory: "256Mi", cpu: "250m"}`                             |
| `resources.limits`                       | CPU/Memory resource limits                                                                             | `{}`                                                         |
| `persistence.enabled`                    | Enable persistence using PVC (Requires architecture: "high-availability")                              | `true`                                                       |
| `persistence.storageClass`               | PVC Storage Class for Memcached volume                                                                   | `nil` (uses alpha storage class annotation)                  |
| `persistence.accessMode`                 | PVC Access Mode for Memcached volume                                                                     | `ReadWriteOnce`                                              |
| `persistence.size`                       | PVC Storage Request for Memcached volume                                                                 | `8Gi`                                                        |
| `securityContext.enabled`                | Enable security context                                                                                | `true`                                                       |
| `securityContext.fsGroup`                | Group ID for the container                                                                             | `1001`                                                       |
| `securityContext.runAsUser`              | User ID for the container                                                                              | `1001`                                                       |
| `securityContext.readOnlyRootFilesystem` | Enable read-only filesystem                                                                            | `false`                                                      |
| `podAnnotations`                         | Pod annotations                                                                                        | `{}`                                                         |
| `affinity`                               | Map of node/pod affinities                                                                             | `{}` (The value is evaluated as a template)                  |
| `nodeSelector`                           | Node labels for pod assignment                                                                         | `{}` (The value is evaluated as a template)                  |
| `tolerations`                            | Tolerations for pod assignment                                                                         | `[]` (The value is evaluated as a template)                  |
| `metrics.enabled`                        | Start a side-car prometheus exporter                                                                   | `false`                                                      |
| `metrics.image.registry`                 | Memcached exporter image registry                                                                      | `docker.io`                                                  |
| `metrics.image.repository`               | Memcached exporter image name                                                                          | `bitnami/memcached-exporter`                                 |
| `metrics.image.tag`                      | Memcached exporter image tag                                                                           | `{TAG_NAME}`                                                 |
| `metrics.image.pullPolicy`               | Image pull policy                                                                                      | `IfNotPresent`                                               |
| `metrics.image.pullSecrets`              | Specify docker-registry secret names as an array                                                       | `[]` (does not add image pull secrets to deployed pods)      |
| `metrics.podAnnotations`                 | Additional annotations for Metrics exporter                                                            | `{prometheus.io/scrape: "true", prometheus.io/port: "9150"}` |
| `metrics.resources`                      | Exporter resource requests/limit                                                                       | `{}`                                                         |
| `metrics.service.type`                   | Kubernetes service type for Prometheus metrics                                                         | `ClusterIP`                                                  |
| `metrics.service.port`                   | Prometheus metrics service port                                                                        | `9150`                                                       |
| `metrics.service.annotations`            | Prometheus exporter svc annotations                                                                    | `{prometheus.io/scrape: "true", prometheus.io/port: "9150"}` |

The above parameters map to the env variables defined in [bitnami/memcached](http://github.com/bitnami/bitnami-docker-memcached). For more information please refer to the [bitnami/memcached](http://github.com/bitnami/bitnami-docker-memcached) image documentation.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install my-release --set memcachedUsername=user,memcachedPassword=password bitnami-azure/memcached
```

The above command sets the Memcached admin account username and password to `user` and `password` respectively.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
$ helm install my-release -f values.yaml bitnami-azure/memcached
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Configuration and installation details

### [Rolling VS Immutable tags](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

### Production configuration

This chart includes a `values-production.yaml` file where you can find some parameters oriented to production configuration in comparison to the regular `values.yaml`. You can use this file instead of the default one.

- Start a side-car prometheus exporter:

```diff
- metrics.enabled: false
+ metrics.enabled: true
```

- Enable read-only filesystem:

```diff
- securityContext.readOnlyRootFilesystem=false
+ securityContext.readOnlyRootFilesystem=true
```

## Persistence

When using `architecture: "high-availability"` the [Bitnami Memcached](https://github.com/bitnami/bitnami-docker-memcached) image stores the cache-state at the `/cache-state` path of the container if enabled.

Persistent Volume Claims are used to keep the data across deployments. This is known to work in GCE, AWS, and minikube.
See the [Parameters](#parameters) section to configure the PVC or to disable persistence.

## Notable changes

### 4.0.0

Backwards compatibility is not guaranteed unless you modify the labels used on the chart's deployments.
Use the workaround below to upgrade from versions previous to 4.0.0. The following example assumes that the release name is memcached:

```console
$ kubectl delete deployment  memcached --cascade=false
$ helm upgrade memcached bitnami-azure/memcached
```

### 3.0.0

This release uses the new bash based `bitnami/memcached` container which uses bash scripts for the start up logic of the container and is smaller in size.

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami’s Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

## Upgrading

### To 5.0.0

[On November 13, 2020, Helm v2 support was formally finished](https://github.com/helm/charts#status-of-the-project), this major version is the result of the required changes applied to the Helm Chart to be able to incorporate the different features added in Helm v3 and to be consistent with the Helm project itself regarding the Helm v2 EOL.

**What changes were introduced in this major version?**

- Previous versions of this Helm Chart use `apiVersion: v1` (installable by both Helm 2 and 3), this Helm Chart was updated to `apiVersion: v2` (installable by Helm 3 only). [Here](https://helm.sh/docs/topics/charts/#the-apiversion-field) you can find more information about the `apiVersion` field.
- The different fields present in the *Chart.yaml* file has been ordered alphabetically in a homogeneous way for all the Bitnami Helm Charts

**Considerations when upgrading to this version**

- If you want to upgrade to this version from a previous one installed with Helm v3, you shouldn't face any issues
- If you want to upgrade to this version using Helm v2, this scenario is not supported as this version doesn't support Helm v2 anymore
- If you installed the previous version with Helm v2 and wants to upgrade to this version with Helm v3, please refer to the [official Helm documentation](https://helm.sh/docs/topics/v2_v3_migration/#migration-use-cases) about migrating from Helm v2 to v3

**Useful links**

- https://docs.bitnami.com/tutorials/resolve-helm2-helm3-post-migration-issues/
- https://helm.sh/docs/topics/v2_v3_migration/
- https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/

### To 1.0.0

Backwards compatibility is not guaranteed unless you modify the labels used on the chart's deployments.
Use the workaround below to upgrade from versions previous to 1.0.0. The following example assumes that the release name is memcached:

```console
$ kubectl patch deployment memcached --type=json -p='[{"op": "remove", "path": "/spec/selector/matchLabels/chart"}]'
```