---
title: activemq
parent: Charts Reference
---

# activemq

![Version: 3.6.2](https://img.shields.io/badge/Version-3.6.2-informational?style=flat-square) ![AppVersion: 5.18.7](https://img.shields.io/badge/AppVersion-5.18.7-informational?style=flat-square)

A Helm chart providing a basic Apache ActiveMQ deployment required to evaluate ACS (not meant to be used in production).

This chart can deploy ActiveMQ in two modes:
- **ActiveMQ Classic**: The default deployment, using a standard Kubernetes Deployment. Configurations for this mode are at the top level of the `values.yaml` file (e.g., `replicaCount`, `image`, `persistence`).
- **ActiveMQ Artemis**: A newer, more advanced broker. When enabled via `artemis.enabled: true`, it deploys ActiveMQ Artemis as a StatefulSet, enabling clustering features. Artemis-specific configurations are under the `artemis` block in `values.yaml`.

If `artemis.enabled` is `true`, the ActiveMQ Classic deployment and its related top-level configurations (except for global settings like `enabled` and `global.alfrescoRegistryPullSecrets`) are ignored.

## ActiveMQ Artemis Mode

When `artemis.enabled: true` in your `values.yaml`, this chart deploys ActiveMQ Artemis. This mode utilizes a Kubernetes StatefulSet to facilitate clustering and stable network identifiers for broker communication.

Key features and configuration options for Artemis mode include:

*   **Enabling Artemis**: Set `artemis.enabled: true`.
*   **Clustering**:
    *   Artemis is deployed as a StatefulSet, which is fundamental for clustering.
    *   Clustering is typically enabled by default when Artemis mode is active (`artemis.cluster.enabled: true`).
    *   `artemis.cluster.replicaCount`: Defines the number of Artemis broker instances in the cluster (e.g., 3 for a typical cluster).
    *   `artemis.cluster.name`: A name for the Artemis cluster (e.g., `artemis-cluster`).
    *   **Cluster Credentials**: `artemis.cluster.clusterUser` and `artemis.cluster.clusterPassword` are used for inter-broker authentication. **It is crucial to change the default `clusterPassword` for production deployments.**
    *   Cluster discovery is configured using static connectors that leverage Kubernetes headless service DNS for pod-to-pod communication.
*   **Broker Image**:
    *   `artemis.image.repository`: Specifies the Docker image repository for Artemis (e.g., `apache/activemq-artemis`).
    *   `artemis.image.tag`: Specifies the image tag (e.g., `latest-alpine`). Ensure the chosen image is suitable for your needs.
*   **Admin User**:
    *   `artemis.adminUser.login` and `artemis.adminUser.password`: Credentials for the Artemis admin user, used for management purposes (e.g., accessing the web console). **It is crucial to change the default `password` for production deployments.**
    *   `artemis.adminUser.existingSecretName`: Alternatively, provide the name of an existing Kubernetes secret containing `ARTEMIS_USER` and `ARTEMIS_PASSWORD` keys for the admin credentials.
*   **Persistence**:
    *   `artemis.persistence.enabled`: Controls whether persistence is enabled for Artemis.
    *   If enabled, it uses `volumeClaimTemplates` within the StatefulSet to provide each Artemis pod with its own PersistentVolumeClaim.
    *   Configuration options include `artemis.persistence.size` and `artemis.persistence.storageClassName`.
*   **Resources**:
    *   `artemis.resources`: Allows specifying CPU and memory requests and limits for the Artemis pods (e.g., `artemis.resources.requests.cpu`, `artemis.resources.limits.memory`). Ensure these are adequate for your expected load.

**Important Considerations for Artemis Mode**:
*   **Change Default Credentials**: For any production or security-sensitive environment, **you MUST change** the default passwords for `artemis.cluster.clusterPassword` and `artemis.adminUser.password`, or use `artemis.adminUser.existingSecretName`.
*   **Sufficient Resources**: Artemis clusters require adequate CPU, memory, and storage. Monitor resource usage and adjust `artemis.resources` and `artemis.persistence.size` as needed.
*   **Headless Service**: A headless service (`<release-name>-artemis-headless`) is created to provide DNS records for each pod, enabling stable network identifiers for the StatefulSet and cluster discovery.
*   **Client Connections**: A separate service (`<release-name>-artemis`) is created for client connections to the Artemis cluster.

When `artemis.enabled` is `false` (the default), the chart deploys ActiveMQ Classic, and the configurations described below apply.

## Source Code

* <https://github.com/Alfresco/alfresco-helm-charts>

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://alfresco.github.io/alfresco-helm-charts/ | alfresco-common | 4.0.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| adminUser.existingSecretName | string | `nil` | **Classic Mode:** An existing kubernetes secret that contains BROKER_USERNAME and BROKER_PASSWORD keys to override the default user credentials. |
| adminUser.password | string | `"admin"` | **Classic Mode:** Password for the default user with administrative privileges. |
| adminUser.user | string | `"admin"` | **Classic Mode:** Username for the default user with administrative privileges. |
| artemis.adminUser.existingSecretName | string | `""` | **Artemis Mode:** Name of an existing Kubernetes secret (containing `ARTEMIS_USER`, `ARTEMIS_PASSWORD`) for Artemis admin credentials. |
| artemis.adminUser.login | string | `"artemis"` | **Artemis Mode:** Admin username for Artemis console. **Change for production.** |
| artemis.adminUser.password | string | `"CHANGE_ME_PLEASE_TOO"` | **Artemis Mode:** Admin password for Artemis console. **MUST be changed for production.** |
| artemis.cluster.clusterPassword | string | `"CHANGE_ME_PLEASE"` | **Artemis Mode:** Password for inter-broker communication. **MUST be changed for production.** |
| artemis.cluster.clusterUser | string | `"artemisClusterAdmin"` | **Artemis Mode:** Username for inter-broker communication. |
| artemis.cluster.enabled | bool | `true` | **Artemis Mode:** Enable Artemis cluster (effective if artemis.enabled is true). |
| artemis.cluster.name | string | `"artemis-cluster"` | **Artemis Mode:** Name for the Artemis cluster. |
| artemis.cluster.replicaCount | int | `3` | **Artemis Mode:** Number of Artemis replicas in the cluster. |
| artemis.enabled | bool | `false` | Enable ActiveMQ Artemis mode. If true, deploys Artemis as a StatefulSet and ignores Classic configurations. |
| artemis.image.pullPolicy | string | `"IfNotPresent"` | **Artemis Mode:** Image pull policy for Artemis. |
| artemis.image.repository | string | `"apache/activemq-artemis"` | **Artemis Mode:** Image repository for Artemis. |
| artemis.image.tag | string | `"latest-alpine"` | **Artemis Mode:** Image tag for Artemis. Ensure this image exists and is suitable. |
| artemis.persistence.data.mountPath | string | `"/var/lib/artemis-instance/data"` | **Artemis Mode:** Mount path for Artemis data within the container. |
| artemis.persistence.data.path | string | `"/var/lib/artemis-instance/data"` | **Artemis Mode:** Artemis data directory path (often managed by the image itself). |
| artemis.persistence.data.subPath | string | `""` | **Artemis Mode:** SubPath for Artemis data (usually not needed with StatefulSet volumeClaimTemplates). |
| artemis.persistence.enabled | bool | `true` | **Artemis Mode:** Enable persistence for Artemis using volumeClaimTemplates. |
| artemis.persistence.size | string | `"10Gi"` | **Artemis Mode:** Persistence volume size for each Artemis replica. |
| artemis.persistence.storageClassName | string | `""` | **Artemis Mode:** Storage class for Artemis persistence. If empty, uses default. |
| artemis.resources.limits.cpu | string | `"1Gi"` | **Artemis Mode:** CPU limit for Artemis pods. |
| artemis.resources.limits.memory | string | `"1Gi"` | **Artemis Mode:** Memory limit for Artemis pods. |
| artemis.resources.requests.cpu | string | `"500m"` | **Artemis Mode:** CPU request for Artemis pods. |
| artemis.resources.requests.memory | string | `"500Mi"` | **Artemis Mode:** Memory request for Artemis pods. |
| enabled | bool | `true` | If true, deploys the chart (either Classic or Artemis mode). |
| global.alfrescoRegistryPullSecrets | string | `"quay-registry-secret"` | Authenticate to image registry before pulling by providing an existing secret of type kubernetes.io/dockerconfigjson. Applies to both Classic and Artemis modes. |
| image.pullPolicy | string | `"IfNotPresent"` | **Classic Mode:** Image pull policy. |
| image.repository | string | `"alfresco/alfresco-activemq"` | **Classic Mode:** Image repository. |
| image.tag | string | `"5.18.7-jre17-rockylinux8"` | **Classic Mode:** Image tag. |
| livenessProbe.failureThreshold | int | `6` | **Classic Mode:** Liveness probe failure threshold. |
| livenessProbe.initialDelaySeconds | int | `60` | **Classic Mode:** Liveness probe initial delay. |
| livenessProbe.periodSeconds | int | `10` | **Classic Mode:** Liveness probe period. |
| livenessProbe.tcpSocket.port | string | `"openwire"` | **Classic Mode:** Liveness probe TCP socket port. |
| livenessProbe.timeoutSeconds | int | `1` | **Classic Mode:** Liveness probe timeout. |
| nodeSelector | object | `{}` | **Classic Mode:** Node selector for pod assignment. |
| persistence.accessModes | list | `["ReadWriteOnce"]` | **Classic Mode:** Access modes for persistent volume. [Access_Modes] (https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) |
| persistence.baseSize | string | `"20Gi"` | **Classic Mode:** Base size for persistence. |
| persistence.data.mountPath | string | `"/opt/activemq/data"` | **Classic Mode:** Mount path for data. |
| persistence.data.subPath | string | `"alfresco-infrastructure/activemq-data"` | **Classic Mode:** SubPath for data. |
| persistence.enabled | bool | `true` | **Classic Mode:** Enable persistence. |
| persistence.existingClaim | string | `nil` | **Classic Mode:** Existing PVC to use. |
| persistence.storageClass | string | `nil` | **Classic Mode:** Storage class for persistence. |
| podSecurityContext.fsGroup | int | `1000` | Security context for the pod (fsGroup). Applies to Classic mode; Artemis has its own defaults or can be configured via common helpers if needed. |
| podSecurityContext.runAsGroup | int | `1000` | Security context for the pod (runAsGroup). Applies to Classic mode. |
| podSecurityContext.runAsUser | int | `33031` | Security context for the pod (runAsUser). Applies to Classic mode. |
| readinessProbe.failureThreshold | int | `6` | **Classic Mode:** Readiness probe failure threshold. |
| readinessProbe.initialDelaySeconds | int | `5` | **Classic Mode:** Readiness probe initial delay. |
| readinessProbe.periodSeconds | int | `10` | **Classic Mode:** Readiness probe period. |
| readinessProbe.tcpSocket.port | string | `"openwire"` | **Classic Mode:** Readiness probe TCP socket port. |
| readinessProbe.timeoutSeconds | int | `1` | **Classic Mode:** Readiness probe timeout. |
| replicaCount | int | `1` | **Classic Mode:** Number of ActiveMQ Classic replicas. |
| resources.limits.cpu | string | `"2"` | **Classic Mode:** CPU limit. |
| resources.limits.memory | string | `"2048Mi"` | **Classic Mode:** Memory limit. |
| resources.requests.cpu | string | `"0.25"` | **Classic Mode:** CPU request. |
| resources.requests.memory | string | `"512Mi"` | **Classic Mode:** Memory request. |
| service.name | string | `"activemq"` | **Classic Mode:** Name for the ActiveMQ Classic service. |
| services.broker.ports.external.amqp | int | `5672` | **Classic Mode:** External AMQP port. Artemis mode uses `artemis.service.ports.amqp.port` or similar if customized. |
| services.broker.ports.external.openwire | int | `61616` | **Classic Mode:** External OpenWire port. Artemis mode uses `artemis.service.ports.core.port` or similar. |
| services.broker.ports.external.stomp | int | `61613` | **Classic Mode:** External STOMP port. Artemis mode may expose STOMP differently if configured. |
| services.broker.ports.internal.amqp | int | `5672` | **Classic Mode:** Internal AMQP port. |
| services.broker.ports.internal.openwire | int | `61616` | **Classic Mode:** Internal OpenWire port. |
| services.broker.ports.internal.stomp | int | `61613` | **Classic Mode:** Internal STOMP port. |
| services.broker.type | string | `"ClusterIP"` | **Classic Mode:** Service type for the broker. Artemis mode uses `artemis.service.type`. |
| services.webConsole.ports.external.webConsole | int | `8161` | **Classic Mode:** External web console port. Artemis mode uses `artemis.service.ports.webConsole.port` or similar. |
| services.webConsole.ports.internal.webConsole | int | `8161` | **Classic Mode:** Internal web console port. |
| services.webConsole.type | string | `"NodePort"` | **Classic Mode:** Service type for the web console. Artemis mode uses `artemis.service.type`. |
