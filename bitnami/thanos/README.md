# Thanos

[Thanos](https://thanos.io/) is a highly available metrics system that can be added on top of existing Prometheus deployments, providing a global query view across all Prometheus installations.

## Azure-ready Charts with Containers from marketplace.azurecr.io

This Helm Chart has been configured to pull the Container Images from the Azure Marketplace Public Repository.
The following command allows you to download and install all the charts from this repository.
```bash
$ helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
```
## TL;DR

```bash
helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
helm install my-release bitnami-azure/thanos
```

## Introduction

This chart bootstraps a [Thanos](https://github.com/bitnami/bitnami-docker-thanos) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

Bitnami charts can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.0-beta3+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
helm repo add bitnami-azure https://marketplace.azurecr.io/helm/v1/repo
helm install my-release bitnami-azure/thanos
```

These commands deploy Thanos on the Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` chart:

```bash
helm uninstall my-release
```

## Architecture

This charts allows you install several Thanos components, so you deploy an architecture as the one below:

```
                       ┌──────────────┐                  ┌──────────────┐      ┌──────────────┐
                       │ Thanos       │───────────┬────▶ │ Thanos Store │      │ Thanos       │
                       │ Query        │           │      │ Gateway      │      │ Compactor    │
                       └──────────────┘           │      └──────────────┘      └──────────────┘
                   push                           │             │                     │
┌──────────────┐   alerts   ┌──────────────┐      │             │ storages            │ Downsample &
│ Alertmanager │ ◀──────────│ Thanos       │ ◀────┤             │ query metrics       │ compact blocks
│ (*)          │            │ Ruler        │      │             │                     │
└──────────────┘            └──────────────┘      │             ▼                     │
      ▲                            │              │      ┌──────────────┐             │
      │ push alerts                └──────────────│────▶ │ MinIO (*)    │ ◀───────────┘
      │                                           │      │              │
┌ ── ── ── ── ── ── ── ── ── ──┐                  │      └──────────────┘
│┌────────────┐  ┌────────────┐│                  │             ▲
││ Prometheus │─▶│ Thanos     ││ ◀────────────────┘             │
││ (*)        │◀─│ Sidecar (*)││    query                       │ inspect
│└────────────┘  └────────────┘│    metrics                     │ blocks
└ ── ── ── ── ── ── ── ── ── ──┘                                │
                                                         ┌──────────────┐
                                                         │ Thanos       │
                                                         │ Bucket Web   │
                                                         └──────────────┘
```

> Note: Components marked with (*) are provided by subchart(s) (such as the [Bitnami Minio chart](https://github.com/bitnami/charts/tree/master/bitnami/minio)) or external charts (such as the [Bitnami kube-prometheus chart](https://github.com/bitnami/charts/tree/master/bitnami/kube-prometheus)).

Check the section [Integrate Thanos with Prometheus and Alertmanager](#integrate-thanos-with-prometheus-and-alertmanager) for detailed instructions to deploy this architecture.

## Parameters

The following tables lists the configurable parameters of the Thanos chart and their default values per section/component:

### Global parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `global.imageRegistry`                               | Global Docker image registry                                                                           | `nil`                                                   |
| `global.imagePullSecrets`                            | Global Docker registry secret names as an array                                                        | `[]` (does not add image pull secrets to deployed pods) |
| `global.storageClass`                                | Global storage class for dynamic provisioning                                                          | `nil`                                                   |

### Common parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `image.registry`                                     | Thanos image registry                                                                                  | `docker.io`                                             |
| `image.repository`                                   | Thanos image name                                                                                      | `bitnami/thanos`                                        |
| `image.tag`                                          | Thanos image tag                                                                                       | `{TAG_NAME}`                                            |
| `image.pullPolicy`                                   | Thanos image pull policy                                                                               | `IfNotPresent`                                          |
| `image.pullSecrets`                                  | Specify docker-registry secret names as an array                                                       | `[]` (does not add image pull secrets to deployed pods) |
| `nameOverride`                                       | String to partially override thanos.fullname                                                           | `nil`                                                   |
| `fullnameOverride`                                   | String to fully override thanos.fullname                                                               | `nil`                                                   |
| `clusterDomain`                                      | Default Kubernetes cluster domain                                                                      | `cluster.local`                                         |
| `objstoreConfig`                                     | [Objstore configuration](https://thanos.io/storage.md/#configuration)                                  | `nil`                                                   |
| `indexCacheConfig`                                   | [Index cache configuration](https://thanos.io/components/store.md/#memcached-index-cache)              | `nil`                                                   |
| `bucketCacheConfig`                                  | [Bucket cache configuration](https://thanos.io/components/store.md/#caching-bucket)                    | `nil`                                                   |
| `existingObjstoreSecret`                             | Name of existing secret with Objstore configuration                                                    | `nil`                                                   |
| `existingObjstoreSecretItems`                        | List of Secret Keys and Paths                                                                          | `[]`                                                    |
| `existingServiceAccount`                             | Name for the existing service account to be shared between the components                              | `nil`                                                   |

### Thanos Querier parameters

| Parameter                                       | Description                                                                                            | Default                                                 |
|-------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `querier.enabled`                               | Enable/disable Thanos Querier component                                                                | `true`                                                  |
| `querier.logLevel`                              | Thanos Querier log level                                                                               | `info`                                                  |
| `querier.replicaLabel`                          | Replica indicator(s) along which data is deduplicated                                                  | `[replica]`                                             |
| `querier.dnsDiscovery.enabled`                  | Enable store APIs discovery via DNS                                                                    | `true`                                                  |
| `querier.dnsDiscovery.sidecarsService`          | Sidecars service name to discover them using DNS discovery                                             | `nil` (evaluated as a template)                         |
| `querier.dnsDiscovery.sidecarsNamespace`        | Sidecars namespace to discover them using DNS discovery                                                | `nil` (evaluated as a template)                         |
| `querier.stores`                                | Store APIs to connect with Thanos Querier                                                              | `[]`                                                    |
| `querier.sdConfig`                              | Service Discovery configuration                                                                        | `nil`                                                   |
| `querier.existingSDConfigmap`                   | Name of existing ConfigMap with Ruler configuration                                                    | `nil`                                                   |
| `querier.extraFlags`                            | Extra Flags to passed to Thanos Querier                                                                | `[]`                                                    |
| `querier.replicaCount`                          | Number of Thanos Querier replicas to deploy                                                            | `1`                                                     |
| `querier.strategyType`                          | Deployment Strategy Type                                                                               | `RollingUpdate`                                         |
| `querier.affinity`                              | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `querier.nodeSelector`                          | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `querier.tolerations`                           | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `querier.priorityClassName`                     | Controller priorityClassName                                                                           | `nil`                                                   |
| `querier.securityContext.enabled`               | Enable security context for Thanos Querier pods                                                        | `true`                                                  |
| `querier.securityContext.fsGroup`               | Group ID for the Thanos Querier filesystem                                                             | `1001`                                                  |
| `querier.securityContext.runAsUser`             | User ID for the Thanos Querier container                                                               | `1001`                                                  |
| `querier.resources.limits`                      | The resources limits for the Thanos Querier container                                                  | `{}`                                                    |
| `querier.resources.requests`                    | The requested resources for the Thanos Querier container                                               | `{}`                                                    |
| `querier.podAnnotations`                        | Annotations for Thanos Querier pods                                                                    | `{}`                                                    |
| `querier.livenessProbe`                         | Liveness probe configuration for Thanos Querier                                                        | `Check values.yaml file`                                |
| `querier.readinessProbe`                        | Readiness probe configuration for Thanos Querier                                                       | `Check values.yaml file`                                |
| `querier.grpcTLS.server.secure`                 | Enable TLS for GRPC server                                                                             | `false`                                                 |
| `querier.grpcTLS.server.cert`                   | TLS Certificate for gRPC server                                                                        | `nil`                                                   |
| `querier.grpcTLS.server.key`                    | TLS Key for gRPC server                                                                                | `nil`                                                   |
| `querier.grpcTLS.server.ca`                     | TLS client CA for gRPC server used for client verification purposes on the server                      | `nil`                                                   |
| `querier.grpcTLS.client.secure`                 | Use TLS when talking to the gRPC server                                                                | `false`                                                 |
| `querier.grpcTLS.client.cert`                   | TLS Certificates to use to identify this client to the server                                          | `nil`                                                   |
| `querier.grpcTLS.client.key`                    | TLS Key for the client's certificate                                                                   | `nil`                                                   |
| `querier.grpcTLS.client.ca`                     | TLS CA Certificates to use to verify gRPC servers                                                      | `nil`                                                   |
| `querier.grpcTLS.client.servername`             | Server name to verify the hostname on the returned gRPC certificates.                                  | `nil`                                                   |
| `querier.service.type`                          | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `querier.service.clusterIP`                     | Thanos Querier service clusterIP IP                                                                    | `None`                                                  |
| `querier.service.http.port`                     | Service HTTP port                                                                                      | `9090`                                                  |
| `querier.service.http.nodePort`                 | Service HTTP node port                                                                                 | `nil`                                                   |
| `querier.service.grpc.port`                     | Service GRPC port                                                                                      | `10901`                                                 |
| `querier.service.grpc.nodePort`                 | Service GRPC node port                                                                                 | `nil`                                                   |
| `querier.service.loadBalancerIP`                | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `querier.service.loadBalancerSourceRanges`      | Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `querier.service.annotations`                   | Annotations for Thanos Querier service                                                                 | `{}`                                                    |
| `querier.service.labelSelectorsOverride`        | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `querier.serviceAccount.annotations`            | Annotations for Thanos Querier Service Account                                                         | `{}`                                                    |
| `querier.rbac.create`                           | Create RBAC                                                                                            | `false`                                                 |
| `querier.pspEnabled`                            | Create PodSecurityPolicy                                                                               | `false`                                                 |
| `querier.autoscaling.enabled`                   | Enable autoscaling for Thanos Querier                                                                  | `false`                                                 |
| `querier.autoscaling.minReplicas`               | Minimum number of Thanos Querier replicas                                                              | `nil`                                                   |
| `querier.autoscaling.maxReplicas`               | Maximum number of Thanos Querier replicas                                                              | `nil`                                                   |
| `querier.autoscaling.targetCPU`                 | Target CPU utilization percentage                                                                      | `nil`                                                   |
| `querier.autoscaling.targetMemory`              | Target Memory utilization percentage                                                                   | `nil`                                                   |
| `querier.pdb.create`                            | Enable/disable a Pod Disruption Budget creation                                                        | `false`                                                 |
| `querier.pdb.minAvailable`                      | Minimum number/percentage of pods that should remain scheduled                                         | `1`                                                     |
| `querier.pdb.maxUnavailable`                    | Maximum number/percentage of pods that may be made unavailable                                         | `nil`                                                   |
| `querier.ingress.enabled`                       | Enable ingress controller resource                                                                     | `false`                                                 |
| `querier.ingress.certManager`                   | Add annotations for cert-manager                                                                       | `false`                                                 |
| `querier.ingress.hostname`                      | Default host for the ingress resource                                                                  | `thanos.local`                                          |
| `querier.ingress.annotations`                   | Ingress annotations                                                                                    | `[]`                                                    |
| `querier.ingress.tls`                           | Create ingress TLS section                                                                             | `false`                                                    |
| `querier.ingress.extraHosts[0].name`            | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `querier.ingress.extraHosts[0].path`            | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `querier.ingress.extraTls[0].hosts[0]`          | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `querier.ingress.extraTls[0].secretName`        | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `querier.ingress.secrets[0].name`               | TLS Secret Name                                                                                        | `nil`                                                   |
| `querier.ingress.secrets[0].certificate`        | TLS Secret Certificate                                                                                 | `nil`                                                   |
| `querier.ingress.secrets[0].key`                | TLS Secret Key                                                                                         | `nil`                                                   |
| `querier.ingress.grpc.enabled`                  | Enable ingress controller resource (GRPC)                                                              | `false`                                                 |
| `querier.ingress.grpc.certManager`              | Add annotations for cert-manager (GRPC)                                                                | `false`                                                 |
| `querier.ingress.grpc.hostname`                 | Default host for the ingress resource (GRPC)                                                           | `thanos.local`                                          |
| `querier.ingress.grpc.annotations`              | Ingress annotations (GRPC)                                                                             | `[]`                                                    |
| `querier.ingress.grpc.extraHosts[0].name`       | Additional hostnames to be covered (GRPC)                                                              | `nil`                                                   |
| `querier.ingress.grpc.extraHosts[0].path`       | Additional hostnames to be covered (GRPC)                                                              | `nil`                                                   |
| `querier.ingress.grpc.extraTls[0].hosts[0]`     | TLS configuration for additional hostnames to be covered (GRPC)                                        | `nil`                                                   |
| `querier.ingress.grpc.extraTls[0].secretName`   | TLS configuration for additional hostnames to be covered (GRPC)                                        | `nil`                                                   |
| `querier.ingress.grpc.secrets[0].name`          | TLS Secret Name (GRPC)                                                                                 | `nil`                                                   |
| `querier.ingress.grpc.secrets[0].certificate`   | TLS Secret Certificate (GRPC)                                                                          | `nil`                                                   |
| `querier.ingress.grpc.secrets[0].key`           | TLS Secret Key (GRPC)                                                                                  | `nil`                                                   |

### Thanos Query Frontend parameters

| Parameter                                       | Description                                                                                            | Default                                                 |
|-------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `queryFrontend.enabled`                         | Enable/disable Thanos Query Frontend component                                                         | `true`                                                  |
| `queryFrontend.logLevel`                        | Thanos Query Frontend log level                                                                        | `info`                                                  |
| `queryFrontend.extraFlags`                      | Extra Flags to passed to Thanos Query Frontend                                                         | `[]`                                                    |
| `queryFrontend.config`                          | Thanos Query Frontend cache configuration                                                              | `nil`                                                   |
| `queryFrontend.existingConfigmap`               | Name of existing ConfigMap with Thanos Query Frontend cache configuration                              | `nil`                                                   |
| `queryFrontend.replicaCount`                    | Number of Thanos Query Frontend replicas to deploy                                                     | `1`                                                     |
| `queryFrontend.strategyType`                    | Deployment Strategy Type                                                                               | `RollingUpdate`                                         |
| `queryFrontend.affinity`                        | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `queryFrontend.nodeSelector`                    | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `queryFrontend.tolerations`                     | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `queryFrontend.priorityClassName`               | Controller priorityClassName                                                                           | `nil`                                                   |
| `queryFrontend.securityContext.enabled`         | Enable security context for Thanos Query Frontend pods                                                 | `true`                                                  |
| `queryFrontend.securityContext.fsGroup`         | Group ID for the Thanos Query Frontend filesystem                                                      | `1001`                                                  |
| `queryFrontend.securityContext.runAsUser`       | User ID for the Thanos queryFrontend container                                                         | `1001`                                                  |
| `queryFrontend.resources.limits`                | The resources limits for the Thanos Query Frontend container                                           | `{}`                                                    |
| `queryFrontend.resources.requests`              | The requested resources for the Thanos Query Frontend container                                        | `{}`                                                    |
| `queryFrontend.podAnnotations`                  | Annotations for Thanos Query Frontend pods                                                             | `{}`                                                    |
| `queryFrontend.livenessProbe`                   | Liveness probe configuration for Thanos Query Frontend                                                 | `Check values.yaml file`                                |
| `queryFrontend.readinessProbe`                  | Readiness probe configuration for Thanos Query Frontend                                                | `Check values.yaml file`                                |
| `queryFrontend.service.type`                    | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `queryFrontend.service.clusterIP`               | Thanos Query Frontend  service clusterIP IP                                                            | `None`                                                  |
| `queryFrontend.service.http.port`               | Service HTTP port                                                                                      | `9090`                                                  |
| `queryFrontend.service.http.nodePort`           | Service HTTP node port                                                                                 | `nil`                                                   |
| `queryFrontend.service.loadBalancerIP`          | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `queryFrontend.service.loadBalancerSourceRanges`| Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `queryFrontend.service.annotations`             | Annotations for Thanos Query Frontend service                                                          | `{}`                                                    |
| `queryFrontend.service.labelSelectorsOverride`  | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `queryFrontend.serviceAccount.annotations`      | Annotations for Thanos Query Frontend Service Account                                                  | `{}`                                                    |
| `queryFrontend.rbac.create`                     | Create RBAC                                                                                            | `false`                                                 |
| `queryFrontend.pspEnabled`                      | Create PodSecurityPolicy                                                                               | `false`                                                 |
| `queryFrontend.autoscaling.enabled`             | Enable autoscaling for Thanos Query Frontend                                                           | `false`                                                 |
| `queryFrontend.autoscaling.minReplicas`         | Minimum number of Thanos Query Frontend replicas                                                       | `nil`                                                   |
| `queryFrontend.autoscaling.maxReplicas`         | Maximum number of Thanos Query Frontend replicas                                                       | `nil`                                                   |
| `queryFrontend.autoscaling.targetCPU`           | Target CPU utilization percentage                                                                      | `nil`                                                   |
| `queryFrontend.autoscaling.targetMemory`        | Target Memory utilization percentage                                                                   | `nil`                                                   |
| `queryFrontend.pdb.create`                      | Enable/disable a Pod Disruption Budget creation                                                        | `false`                                                 |
| `queryFrontend.pdb.minAvailable`                | Minimum number/percentage of pods that should remain scheduled                                         | `1`                                                     |
| `queryFrontend.pdb.maxUnavailable`              | Maximum number/percentage of pods that may be made unavailable                                         | `nil`                                                   |
| `queryFrontend.ingress.enabled`                 | Enable ingress controller resource                                                                     | `false`                                                 |
| `queryFrontend.ingress.certManager`             | Add annotations for cert-manager                                                                       | `false`                                                 |
| `queryFrontend.ingress.hostname`                | Default host for the ingress resource                                                                  | `thanos.local`                                          |
| `queryFrontend.ingress.annotations`             | Ingress annotations                                                                                    | `[]`                                                    |
| `queryFrontend.ingress.extraHosts[0].name`      | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `queryFrontend.ingress.extraHosts[0].path`      | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `queryFrontend.ingress.extraTls[0].hosts[0]`    | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `queryFrontend.ingress.extraTls[0].secretName`  | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `queryFrontend.ingress.secrets[0].name`         | TLS Secret Name                                                                                        | `nil`                                                   |
| `queryFrontend.ingress.secrets[0].certificate`  | TLS Secret Certificate                                                                                 | `nil`                                                   |
| `queryFrontend.ingress.secrets[0].key`          | TLS Secret Key                                                                                         | `nil`                                                   |

### Thanos Bucket Web parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `bucketweb.enabled`                                  | Enable/disable Thanos Bucket Web component                                                             | `false`                                                 |
| `bucketweb.logLevel`                                 | Thanos Bucket Web log level                                                                            | `info`                                                  |
| `bucketweb.refresh`                                  | Refresh interval to download metadata from remote storage                                              | `30m`                                                   |
| `bucketweb.timeout`                                  | Timeout to download metadata from remote storage                                                       | `5m`                                                    |
| `bucketweb.extraFlags`                               | Extra Flags to passed to Thanos Bucket Web                                                             | `[]`                                                    |
| `bucketweb.replicaCount`                             | Number of Thanos Bucket Web replicas to deploy                                                         | `1`                                                     |
| `bucketweb.strategyType`                             | Deployment Strategy Type                                                                               | `RollingUpdate`                                         |
| `bucketweb.affinity`                                 | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `bucketweb.nodeSelector`                             | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `bucketweb.tolerations`                              | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `bucketweb.priorityClassName`                        | Controller priorityClassName                                                                           | `nil`                                                   |
| `bucketweb.securityContext.enabled`                  | Enable security context for Thanos Bucket Web pods                                                     | `true`                                                  |
| `bucketweb.securityContext.fsGroup`                  | Group ID for the Thanos Bucket Web filesystem                                                          | `1001`                                                  |
| `bucketweb.securityContext.runAsUser`                | User ID for the Thanos Bucket Web container                                                            | `1001`                                                  |
| `bucketweb.resources.limits`                         | The resources limits for the Thanos Bucket Web container                                               | `{}`                                                    |
| `bucketweb.resources.requests`                       | The requested resources for the Thanos Bucket Web container                                            | `{}`                                                    |
| `bucketweb.podAnnotations`                           | Annotations for Thanos Bucket Web pods                                                                 | `{}`                                                    |
| `bucketweb.livenessProbe`                            | Liveness probe configuration for Thanos Compactor                                                      | `Check values.yaml file`                                |
| `bucketweb.readinessProbe`                           | Readiness probe configuration for Thanos Compactor                                                     | `Check values.yaml file`                                |
| `bucketweb.service.type`                             | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `bucketweb.service.clusterIP`                        | Thanos Bucket Web service clusterIP IP                                                                 | `None`                                                  |
| `bucketweb.service.http.port`                        | Service HTTP port                                                                                      | `8080`                                                  |
| `bucketweb.service.http.nodePort`                    | Service HTTP node port                                                                                 | `nil`                                                   |
| `bucketweb.service.loadBalancerIP`                   | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `bucketweb.service.loadBalancerSourceRanges`         | Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `bucketweb.service.annotations`                      | Annotations for Thanos Bucket Web service                                                              | `{}`                                                    |
| `bucketweb.service.labelSelectorsOverride`           | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `bucketweb.serviceAccount.annotations`               | Annotations for Thanos Bucket Web Service Account                                                      | `{}`                                                    |
| `bucketweb.serviceAccount.existingServiceAccount`    | Name for an existing Thanos Bucket Web Service Account                                                 | `nil`                                                   |
| `bucketweb.pdb.create`                               | Enable/disable a Pod Disruption Budget creation                                                        | `false`                                                 |
| `bucketweb.pdb.minAvailable`                         | Minimum number/percentage of pods that should remain scheduled                                         | `1`                                                     |
| `bucketweb.pdb.maxUnavailable`                       | Maximum number/percentage of pods that may be made unavailable                                         | `nil`                                                   |
| `bucketweb.ingress.enabled`                          | Enable ingress controller resource                                                                     | `false`                                                 |
| `bucketweb.ingress.certManager`                      | Add annotations for cert-manager                                                                       | `false`                                                 |
| `bucketweb.ingress.hostname`                         | Default host for the ingress resource                                                                  | `thanos-bucketweb.local`                                |
| `bucketweb.ingress.annotations`                      | Ingress annotations                                                                                    | `[]`                                                    |
| `bucketweb.ingress.tls`                             | Create ingress TLS section                                                                               | `false`                                                    |
| `bucketweb.ingress.extraHosts[0].name`               | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `bucketweb.ingress.extraHosts[0].path`               | Additional hostnames to be covered                                                                     | `nil`                                                   |
| `bucketweb.ingress.extraTls[0].hosts[0]`             | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `bucketweb.ingress.extraTls[0].secretName`           | TLS configuration for additional hostnames to be covered                                               | `nil`                                                   |
| `bucketweb.ingress.secrets[0].name`                  | TLS Secret Name                                                                                        | `nil`                                                   |
| `bucketweb.ingress.secrets[0].certificate`           | TLS Secret Certificate                                                                                 | `nil`                                                   |
| `bucketweb.ingress.secrets[0].key`                   | TLS Secret Key                                                                                         | `nil`                                                   |

### Thanos Compactor parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `compactor.enabled`                                  | Enable/disable Thanos Compactor component                                                              | `false`                                                 |
| `compactor.logLevel`                                 | Thanos Compactor log level                                                                             | `info`                                                  |
| `compactor.retentionResolutionRaw`                   | Resolution and Retention flag                                                                          | `30d`                                                   |
| `compactor.retentionResolution5m`                    | Resolution and Retention flag                                                                          | `30d`                                                   |
| `compactor.retentionResolution1h`                    | Resolution and Retention flag                                                                          | `10y`                                                   |
| `compactor.consistencyDelay`                         | Minimum age of fresh blocks before they are being processed                                            | `30m`                                                   |
| `compactor.extraFlags`                               | Extra Flags to passed to Thanos Compactor                                                              | `[]`                                                    |
| `compactor.strategyType`                             | Deployment Strategy Type                                                                               | `RollingUpdate`                                         |
| `compactor.affinity`                                 | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `compactor.nodeSelector`                             | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `compactor.tolerations`                              | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `compactor.priorityClassName`                        | Controller priorityClassName                                                                           | `nil`                                                   |
| `compactor.securityContext.enabled`                  | Enable security context for Thanos Compactor pods                                                      | `true`                                                  |
| `compactor.securityContext.fsGroup`                  | Group ID for the Thanos Compactor filesystem                                                           | `1001`                                                  |
| `compactor.securityContext.runAsUser`                | User ID for the Thanos Compactor container                                                             | `1001`                                                  |
| `compactor.resources.limits`                         | The resources limits for the Thanos Compactor container                                                | `{}`                                                    |
| `compactor.resources.requests`                       | The requested resources for the Thanos Compactor container                                             | `{}`                                                    |
| `compactor.podAnnotations`                           | Annotations for Thanos Compactor pods                                                                  | `{}`                                                    |
| `compactor.livenessProbe`                            | Liveness probe configuration for Thanos Compactor                                                      | `Check values.yaml file`                                |
| `compactor.readinessProbe`                           | Readiness probe configuration for Thanos Compactor                                                     | `Check values.yaml file`                                |
| `compactor.service.type`                             | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `compactor.service.clusterIP`                        | Thanos Compactor service clusterIP IP                                                                  | `None`                                                  |
| `compactor.service.http.port`                        | Service HTTP port                                                                                      | `9090`                                                  |
| `compactor.service.http.nodePort`                    | Service HTTP node port                                                                                 | `nil`                                                   |
| `compactor.service.loadBalancerIP`                   | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `compactor.service.loadBalancerSourceRanges`         | Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `compactor.service.annotations`                      | Annotations for Thanos Compactor service                                                               | `{}`                                                    |
| `compactor.service.labelSelectorsOverride`          | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `compactor.serviceAccount.annotations`               | Annotations for Thanos Compactor Service Account                                                       | `{}`                                                    |
| `compactor.serviceAccount.existingServiceAccount`    | Name for an existing Thanos Compactor Service Account                                                  | `nil`                                                   |
| `compactor.persistence.enabled`                      | Enable data persistence                                                                                | `true`                                                  |
| `compactor.persistence.existingClaim`                | Use a existing PVC which must be created manually before bound                                         | `nil`                                                   |
| `compactor.persistence.storageClass`                 | Specify the `storageClass` used to provision the volume                                                | `nil`                                                   |
| `compactor.persistence.accessModes`                  | Access modes of data volume                                                                            | `["ReadWriteOnce"]`                                     |
| `compactor.persistence.size`                         | Size of data volume                                                                                    | `8Gi`                                                   |

### Thanos Store Gateway parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `storegateway.enabled`                               | Enable/disable Thanos Store Gateway component                                                          | `false`                                                 |
| `storegateway.logLevel`                              | Thanos Store Gateway log level                                                                         | `info`                                                  |
| `storegateway.extraFlags`                            | Extra Flags to passed to Thanos Store Gateway                                                          | `[]`                                                    |
| `storegateway.config`                                | Thanos Store Gateway cache configuration                                                               | `nil`                                                   |
| `storegateway.existingConfigmap`                     | Name of existing ConfigMap with Thanos Store Gateway cache configuration                               | `nil`                                                   |
| `storegateway.updateStrategyType`                    | Statefulset Update Strategy Type                                                                       | `RollingUpdate`                                         |
| `storegateway.podManagementPolicy`                   | Statefulset Pod Management Policy Type                                                                 | `OrderedReady`                                          |
| `storegateway.replicaCount`                          | Number of Thanos Store Gateway replicas to deploy                                                      | `1`                                                     |
| `storegateway.affinity`                              | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `storegateway.nodeSelector`                          | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `storegateway.tolerations`                           | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `storegateway.priorityClassName`                     | Controller priorityClassName                                                                           | `nil`                                                   |
| `storegateway.securityContext.enabled`               | Enable security context for Thanos Store Gateway pods                                                  | `true`                                                  |
| `storegateway.securityContext.fsGroup`               | Group ID for the Thanos Store Gateway filesystem                                                       | `1001`                                                  |
| `storegateway.securityContext.runAsUser`             | User ID for the Thanos Store Gateway container                                                         | `1001`                                                  |
| `storegateway.resources.limits`                      | The resources limits for the Thanos Store Gateway container                                            | `{}`                                                    |
| `storegateway.resources.requests`                    | The requested resources for the Thanos Store Gateway container                                         | `{}`                                                    |
| `storegateway.podAnnotations`                        | Annotations for Thanos Store Gateway pods                                                              | `{}`                                                    |
| `storegateway.livenessProbe`                         | Liveness probe configuration for Thanos Store Gateway                                                  | `Check values.yaml file`                                |
| `storegateway.readinessProbe`                        | Readiness probe configuration for Thanos Store Gateway                                                 | `Check values.yaml file`                                |
| `storegateway.service.type`                          | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `storegateway.service.clusterIP`                     | Thanos Store Gateway service clusterIP IP                                                              | `None`                                                  |
| `storegateway.service.http.port`                     | Service HTTP port                                                                                      | `9090`                                                  |
| `storegateway.service.http.nodePort`                 | Service HTTP node port                                                                                 | `nil`                                                   |
| `storegateway.service.grpc.port`                     | Service GRPC port                                                                                      | `10901`                                                 |
| `storegateway.service.grpc.nodePort`                 | Service GRPC node port                                                                                 | `nil`                                                   |
| `storegateway.service.loadBalancerIP`                | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `storegateway.service.loadBalancerSourceRanges`      | Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `storegateway.service.annotations`                   | Annotations for Thanos Store Gateway service                                                           | `{}`                                                    |
| `storegateway.service.labelSelectorsOverride`        | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `storegateway.service.additionalHeadless`            | Additional Headless service                                                                            | `false`                                                 |
| `storegateway.serviceAccount.annotations`            | Annotations for Thanos Store Gateway Service Account                                                   | `{}`                                                    |
| `storegateway.serviceAccount.existingServiceAccount` | Name for an existing Thanos Store Gateway Service Account                                              | `nil`                                                   |
| `storegateway.persistence.enabled`                   | Enable data persistence                                                                                | `true`                                                  |
| `storegateway.persistence.existingClaim`             | Use a existing PVC which must be created manually before bound                                         | `nil`                                                   |
| `storegateway.persistence.storageClass`              | Specify the `storageClass` used to provision the volume                                                | `nil`                                                   |
| `storegateway.persistence.accessModes`               | Access modes of data volume                                                                            | `["ReadWriteOnce"]`                                     |
| `storegateway.persistence.size`                      | Size of data volume                                                                                    | `8Gi`                                                   |
| `storegateway.autoscaling.enabled`                   | Enable autoscaling for Thanos Store Gateway                                                            | `false`                                                 |
| `storegateway.autoscaling.minReplicas`               | Minimum number of Thanos Store Gateway replicas                                                        | `nil`                                                   |
| `storegategay.autoscaling.maxReplicas`               | Maximum number of Thanos Store Gateway replicas                                                        | `nil`                                                   |
| `storegateway.autoscaling.targetCPU`                 | Target CPU utilization percentage                                                                      | `nil`                                                   |
| `storegateway.autoscaling.targetMemory`              | Target Memory utilization percentage                                                                   | `nil`                                                   |
| `storegateway.pdb.create`                            | Enable/disable a Pod Disruption Budget creation                                                        | `false`                                                 |
| `storegateway.pdb.minAvailable`                      | Minimum number/percentage of pods that should remain scheduled                                         | `1`                                                     |
| `storegateway.pdb.maxUnavailable`                    | Maximum number/percentage of pods that may be made unavailable                                         | `nil`                                                   |

### Thanos Ruler parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `ruler.enabled`                                      | Enable/disable Thanos Ruler component                                                                  | `false`                                                 |
| `ruler.logLevel`                                     | Thanos Ruler log level                                                                                 | `info`                                                  |
| `ruler.dnsDiscovery.enabled`                         | Enable Querier APIs discovery via DNS                                                                  | `true`                                                  |
| `ruler.alertmanagers`                                | Alermanager URLs array                                                                                 | `[]`                                                    |
| `ruler.evalInterval`                                 | The default evaluation interval to use                                                                 | `1m`                                                    |
| `ruler.clusterName`                                  | Used to set the 'ruler_cluster' label                                                                  | `nil`                                                   |
| `ruler.extraFlags`                                   | Extra Flags to passed to Thanos Ruler                                                                  | `[]`                                                    |
| `ruler.config`                                       | Ruler configuration                                                                                    | `nil`                                                   |
| `ruler.existingConfigmap`                            | Name of existing ConfigMap with Ruler configuration                                                    | `nil`                                                   |
| `ruler.updateStrategyType`                           | Statefulset Update Strategy Type                                                                       | `RollingUpdate`                                         |
| `ruler.podManagementPolicy`                          | Statefulset Pod Management Policy Type                                                                 | `OrderedReady`                                          |
| `ruler.replicaCount`                                 | Number of Thanos Ruler replicas to deploy                                                              | `1`                                                     |
| `ruler.affinity`                                     | Affinity for pod assignment                                                                            | `{}` (evaluated as a template)                          |
| `ruler.nodeSelector`                                 | Node labels for pod assignment                                                                         | `{}` (evaluated as a template)                          |
| `ruler.tolerations`                                  | Tolerations for pod assignment                                                                         | `[]` (evaluated as a template)                          |
| `ruler.priorityClassName`                            | Controller priorityClassName                                                                           | `nil`                                                   |
| `ruler.securityContext.enabled`                      | Enable security context for Thanos Ruler pods                                                          | `true`                                                  |
| `ruler.securityContext.fsGroup`                      | Group ID for the Thanos Ruler filesystem                                                               | `1001`                                                  |
| `ruler.securityContext.runAsUser`                    | User ID for the Thanos Ruler container                                                                 | `1001`                                                  |
| `ruler.resources.limits`                             | The resources limits for the Thanos Ruler container                                                    | `{}`                                                    |
| `ruler.resources.requests`                           | The requested resources for the Thanos Ruler container                                                 | `{}`                                                    |
| `ruler.podAnnotations`                               | Annotations for Thanos Ruler pods                                                                      | `{}`                                                    |
| `ruler.livenessProbe`                                | Liveness probe configuration for Thanos Ruler                                                          | `Check values.yaml file`                                |
| `ruler.readinessProbe`                               | Readiness probe configuration for Thanos Ruler                                                         | `Check values.yaml file`                                |
| `ruler.service.type`                                 | Kubernetes service type                                                                                | `ClusterIP`                                             |
| `ruler.service.clusterIP`                            | Thanos Ruler service clusterIP IP                                                                      | `None`                                                  |
| `ruler.service.http.port`                            | Service HTTP port                                                                                      | `9090`                                                  |
| `ruler.service.http.nodePort`                        | Service HTTP node port                                                                                 | `nil`                                                   |
| `ruler.service.grpc.port`                            | Service GRPC port                                                                                      | `10901`                                                 |
| `ruler.service.grpc.nodePort`                        | Service GRPC node port                                                                                 | `nil`                                                   |
| `ruler.service.loadBalancerIP`                       | loadBalancerIP if service type is `LoadBalancer`                                                       | `nil`                                                   |
| `ruler.service.loadBalancerSourceRanges`             | Address that are allowed when service is LoadBalancer                                                  | `[]`                                                    |
| `ruler.service.annotations`                          | Annotations for Thanos Ruler service                                                                   | `{}`                                                    |
| `ruler.service.labelSelectorsOverride`        | Selector for Thanos querier service                                                                    | `{}`                                                    |
| `ruler.service.additionalHeadless`                   | Additional Headless service                                                                            | `false`                                                 |
| `ruler.serviceAccount.annotations`                   | Annotations for Thanos Ruler Service Account                                                           | `{}`                                                    |
| `ruler.serviceAccount.existingServiceAccount`        | Name for an existing Thanos Ruler Service Account                                                      | `nil`                                                   |
| `ruler.persistence.enabled`                          | Enable data persistence                                                                                | `true`                                                  |
| `ruler.persistence.existingClaim`                    | Use a existing PVC which must be created manually before bound                                         | `nil`                                                   |
| `ruler.persistence.storageClass`                     | Specify the `storageClass` used to provision the volume                                                | `nil`                                                   |
| `ruler.persistence.accessModes`                      | Access modes of data volume                                                                            | `["ReadWriteOnce"]`                                     |
| `ruler.persistence.size`                             | Size of data volume                                                                                    | `8Gi`                                                   |
| `ruler.pdb.create`                                   | Enable/disable a Pod Disruption Budget creation                                                        | `false`                                                 |
| `ruler.pdb.minAvailable`                             | Minimum number/percentage of pods that should remain scheduled                                         | `1`                                                     |
| `ruler.pdb.maxUnavailable`                           | Maximum number/percentage of pods that may be made unavailable                                         | `nil`                                                   |

### Metrics parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `metrics.enabled`                                    | Enable the export of Prometheus metrics                                                                | `false`                                                 |
| `metrics.serviceMonitor.enabled`                     | if `true`, creates a Prometheus Operator ServiceMonitor (also requires `metrics.enabled` to be `true`) | `false`                                                 |
| `metrics.serviceMonitor.namespace`                   | Namespace in which Prometheus is running                                                               | `nil`                                                   |
| `metrics.serviceMonitor.labels`                      | Additional labels for ServiceMonitor                                                                   | `{}`                                                    |
| `metrics.serviceMonitor.interval`                    | Interval at which metrics should be scraped.                                                           | `nil` (Prometheus Operator default value)               |
| `metrics.serviceMonitor.scrapeTimeout`               | Timeout after which the scrape is ended                                                                | `nil` (Prometheus Operator default value)               |

### Volume Permissions parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `volumePermissions.enabled`           | Enable init container that changes the owner and group of the persistent volume(s) mountpoint to `runAsUser:fsGroup` | `false`                                                 |
| `volumePermissions.image.registry`    | Init container volume-permissions image registry                                                                     | `docker.io`                                             |
| `volumePermissions.image.repository`  | Init container volume-permissions image name                                                                         | `bitnami/minideb`                                       |
| `volumePermissions.image.tag`         | Init container volume-permissions image tag                                                                          | `buster`                                                |
| `volumePermissions.image.pullPolicy`  | Init container volume-permissions image pull policy                                                                  | `Always`                                                |
| `volumePermissions.image.pullSecrets` | Specify docker-registry secret names as an array                                                                     | `[]` (does not add image pull secrets to deployed pods) |

### MinIO chart parameters

| Parameter                                            | Description                                                                                            | Default                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------------|
| `minio.enabled`                                      | Enable/disable Minio chart installation                                                                | `false`                                                 |
| `minio.accessKey.password`                           | MinIO Access Key                                                                                       | _random 10 character alphanumeric string_               |
| `minio.secretKey.password`                           | MinIO Secret Key                                                                                       | _random 40 character alphanumeric string_               |
| `minio.defaultBuckets`                               | Comma, semi-colon or space separated list of buckets to create                                         | `nil`                                                   |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
helm install my-release --set querier.replicaCount=2 bitnami-azure/thanos
```

The above command install Thanos chart with 2 Thanos Querier replicas.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
helm install my-release -f values.yaml bitnami-azure/thanos
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Configuration and installation details

### [Rolling VS Immutable tags](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

### Production configuration

This chart includes a `values-production.yaml` file where you can find some parameters oriented to production configuration in comparison to the regular `values.yaml`. You can use this file instead of the default one.

- Enable Thanos Compactor:

```diff
- compactor.enabled: false
+ compactor.enabled: true
```

- Enable Thanos Store Gateway:

```diff
- storegateway.enabled: false
+ storegateway.enabled: true
```

- Enable Thanos Ruler:

```diff
- ruler.enabled: false
+ ruler.enabled: true
```

- Enable Prometheus Metrics:

```diff
- metrics.enabled: false
+ metrics.enabled: true
- metrics.serviceMonitor: false
+ metrics.serviceMonitor: true
```

### Adding extra flags

In case you want to add extra flags to any Thanos component, you can use `XXX.extraFlags` parameter(s), where XXX is placeholder you need to replace with the actual component(s). For instance, to add extra flags to Thanos Store Gateway, use:

```yaml
storegateway:
  extraFlags:
    - --sync-block-duration=3m
    - --chunk-pool-size=2GB
```

### Using custom Objstore configuration

This helm chart supports using custom Objstore configuration.

You can specify the Objstore configuration using the `objstoreConfig` parameter.

In addition, you can also set an external Secret with the configuration file. This is done by setting the `existingObjstoreSecret` parameter. Note that this will override the previous option. If needed you can also provide a custom Secret Key with `existingObjstoreSecretItems`, please be aware that the Path of your Secret should be `objstore.yml`.

### Using custom Querier Service Discovery configuration

This helm chart supports using custom Service Discovery configuration for Querier.

You can specify the Service Discovery configuration using the `querier.sdConfig` parameter.

In addition, you can also set an external ConfigMap with the Service Discovery configuration file. This is done by setting the `querier.existingSDConfigmap` parameter. Note that this will override the previous option.

### Using custom Ruler configuration

This helm chart supports using custom Ruler configuration.

You can specify the Ruler configuration using the `ruler.config` parameter.

In addition, you can also set an external ConfigMap with the configuration file. This is done by setting the `ruler.existingConfigmap` parameter. Note that this will override the previous option.

### Integrate Thanos with Prometheus and Alertmanager

You can intregrate Thanos with Prometheus & Alertmanager using this chart and the [Bitnami kube-prometheus chart](https://github.com/bitnami/charts/tree/master/bitnami/kube-prometheus) following the steps below:

> Note: in this example we will use MinIO (subchart) as the Objstore. Every component will be deployed in the "monitoring" namespace.

- Create a **values.yaml** like the one below:

```yaml
objstoreConfig: |-
  type: s3
  config:
    bucket: thanos
    endpoint: {{ include "thanos.minio.fullname" . }}.monitoring.svc.cluster.local:9000
    access_key: minio
    secret_key: minio123
    insecure: true
querier:
  dnsDiscovery:
    sidecarsService: kube-prometheus-prometheus-thanos
    sidecarsNamespace: monitoring
bucketweb:
  enabled: true
compactor:
  enabled: true
storegateway:
  enabled: true
ruler:
  enabled: true
  alertmanagers:
    - http://kube-prometheus-alertmanager.monitoring.svc.cluster.local:9093
  config: |-
    groups:
      - name: "metamonitoring"
        rules:
          - alert: "PrometheusDown"
            expr: absent(up{prometheus="monitoring/kube-prometheus"})
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
minio:
  enabled: true
  accessKey:
    password: "minio"
  secretKey:
    password: "minio123"
  defaultBuckets: "thanos"
```

- Install Prometheus Operator and Thanos charts:

For Helm 3:

```bash
kubectl create namespace monitoring
helm install kube-prometheus \
    --set prometheus.thanos.create=true \
    --namespace monitoring \
    bitnami-azure/kube-prometheus
helm install thanos \
    --values values.yaml \
    --namespace monitoring \
    bitnami-azure/thanos
```

That's all! Now you have Thanos fully integrated with Prometheus and Alertmanager.

## Persistence

The data is persisted by default using PVC(s) on Thanos components. You can disable the persistence setting the `XXX.persistence.enabled` parameter(s) to `false`. A default `StorageClass` is needed in the Kubernetes cluster to dynamically provision the volumes. Specify another StorageClass in the `XXX.persistence.storageClass` parameter(s) or set `XXX.persistence.existingClaim` if you have already existing persistent volumes to use.

> Note: you need to substitue the XXX placeholders above with the actual component(s) you want to configure.

### Adjust permissions of persistent volume mountpoint

As the images run as non-root by default, it is necessary to adjust the ownership of the persistent volumes so that the containers can write data into it.

By default, the chart is configured to use Kubernetes Security Context to automatically change the ownership of the volumes. However, this feature does not work in all Kubernetes distributions.
As an alternative, this chart supports using an initContainer to change the ownership of the volumes before mounting it in the final destination.

You can enable this initContainer by setting `volumePermissions.enabled` to `true`.

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami’s Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

## Upgrading

### To 3.0.0

[On November 13, 2020, Helm v2 support was formally finished](https://github.com/helm/charts#status-of-the-project), this major version is the result of the required changes applied to the Helm Chart to be able to incorporate the different features added in Helm v3 and to be consistent with the Helm project itself regarding the Helm v2 EOL.

**What changes were introduced in this major version?**

- Previous versions of this Helm Chart use `apiVersion: v1` (installable by both Helm 2 and 3), this Helm Chart was updated to `apiVersion: v2` (installable by Helm 3 only). [Here](https://helm.sh/docs/topics/charts/#the-apiversion-field) you can find more information about the `apiVersion` field.
- Move dependency information from the *requirements.yaml* to the *Chart.yaml*
- After running `helm dependency update`, a *Chart.lock* file is generated containing the same structure used in the previous *requirements.lock*
- The different fields present in the *Chart.yaml* file has been ordered alphabetically in a homogeneous way for all the Bitnami Helm Charts

**Considerations when upgrading to this version**

- If you want to upgrade to this version from a previous one installed with Helm v3, you shouldn't face any issues
- If you want to upgrade to this version using Helm v2, this scenario is not supported as this version doesn't support Helm v2 anymore
- If you installed the previous version with Helm v2 and wants to upgrade to this version with Helm v3, please refer to the [official Helm documentation](https://helm.sh/docs/topics/v2_v3_migration/#migration-use-cases) about migrating from Helm v2 to v3

**Useful links**

- https://docs.bitnami.com/tutorials/resolve-helm2-helm3-post-migration-issues/
- https://helm.sh/docs/topics/v2_v3_migration/
- https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/

### To 2.4.0

The Ingress API object name for Querier changes from:

```yaml
{{ include "thanos.fullname" . }}
```

> **NOTE**: Which in most cases (depending on any set values in `fullnameOverride` or `nameOverride`) resolves to the used Helm release name (`.Release.Name`).

To:

```yaml
{{ include "thanos.fullname" . }}-querier
```

### To 2.0.0

The format of the chart's `extraFlags` option has been updated to be an array (instead of an object), to support passing multiple flags with the same name to Thanos.

Now you need to specify the flags in the following way in your values file (where component is one of `querier/bucketweb/compactor/storegateway/ruler`):

```yaml
component:
  ...
  extraFlags
    - --sync-block-duration=3m
    - --chunk-pool-size=2GB
```

To specify the values via CLI::

```console
--set 'component.extraFlags[0]=--sync-block-duration=3m' --set 'ruler.extraFlags[1]=--chunk-pool-size=2GB'
```

### To 1.0.0

If you are upgrading from a `<1.0.0` release you need to move your Querier Ingress information to the new values settings:
```
ingress.enabled -> querier.ingress.enabled
ingress.certManager -> querier.ingress.certManager
ingress.hostname -> querier.ingress.hostname
ingress.annotations -> querier.ingress.annotations
ingress.extraHosts[0].name -> querier.ingress.extraHosts[0].name
ingress.extraHosts[0].path -> querier.ingress.extraHosts[0].path
ingress.extraHosts[0].hosts[0] -> querier.ingress.extraHosts[0].hosts[0]
ingress.extraHosts[0].secretName -> querier.ingress.extraHosts[0].secretName
ingress.secrets[0].name -> querier.ingress.secrets[0].name
ingress.secrets[0].certificate -> querier.ingress.secrets[0].certificate
ingress.secrets[0].key -> querier.ingress.secrets[0].key