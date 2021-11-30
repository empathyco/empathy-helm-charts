# Elasticsearch-data Helm Chart

This Helm chart is Empathy's own Helm chart (based on the official [Elasticsearch Helm chart](https://github.com/elastic/helm-charts/tree/master/elasticsearch)) to deploy data and ingest Elasticsearch 6.6.2 nodes with a custom Dockerfile habilitating unlimited memory lock (`ulimit -l unlimited`). Additionally, [justwatchcom Elasticsearh Exporter](https://github.com/justwatchcom/elasticsearch_exporter) is deployed as a sidecar container to expose Prometheus metrics in port `9114`.

**N.B.** Please note this Helm chart is specifically tailored to deploy Elasticsearch nodes with the data and ingest roles. For a fully functional Elasticsearch deployment you will have to also deploy Elasticsearch master nodes which can be achieved with Empathy's [Elasticsearch-master Helm chart](../elasticsearch-master).

## Requirements

- Kubernetes >= 1.14
- [Helm](https://helm.sh/) 3
- Minimum cluster requirements include the following to run this chart with default settings.
	- Three Kubernetes nodes to respect the `topologyKey: "kubernetes.io/hostname"` antiaffinity.
	- 3GB of RAM for the JVM heap.
	- 1 vCPU available for resource limits.
	- 200Gi of capacity for each persistent volume.

## Installing

To install the chart with the release name `my-release` and default values, run:

```bash
$ helm repo add empathy-public https://empathyco.github.io/helm-charts
$ helm install --name my-release empathy-public/elasticsearch-data
```

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Usage notes

- The chart deploys a StatefulSet and by default will do an automated rolling update of your cluster. It does this by waiting for the cluster health to become green after each instance is updated. If you prefer to update manually you can set OnDelete updateStrategy.
- It is important to verify that the JVM heap size in `elastic_config.ES_JAVA_OPTS` and to set the CPU/Memory resources to something suitable for your cluster.
- To simplify chart and maintenance each set of node groups is deployed as a separate Helm release, this chart referring to nodes with the data and ingest roles, while the Empathy's [Elasticsearch-master Helm chart](../elasticsearch-master) handles master nodes. This is basically the idea expressed in the official Elasticsearch Helm chart [multi example](https://github.com/elastic/helm-charts/tree/master/elasticsearch/examples/multi/). Without doing this it isn't possible to resize persistent volumes in a StatefulSet. By setting it up this way it makes it possible to add more nodes with a new storage size then drain the old ones. It also solves the problem of allowing the user to determine which node groups to update first when doing upgrades or changes.
- Although based on the official [Elasticsearch Helm Chart](https://github.com/elastic/helm-charts/tree/master/elasticsearch) and the multi example mentioned in the previous point, this chart is much more opinionated that the official Helm chart. It is only thought to deploy nodes with the data and index roles, not masters, with the [justwatchcom Elasticsearh Exporter](https://github.com/justwatchcom/elasticsearch_exporter) and a custom docker image of Elastic 6.6.2 with an unlimited memory lock. Please make sure this chart fits your needs before using it.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| busybox.image | string | `"busybox:1.31"` | Image for busybox initContainers (sysctlInitContainer in official Elasticsearch Helm chart) |
| elastic_config | object | `{"ES_JAVA_OPTS":"-Xms2048m -Xmx2048m","bootstrap.memory_lock":"true","logger.org.elasticsearch.discovery.gce":"TRACE","network.bind_host":"0.0.0.0","node.attr.type":"search","node.data":"true","node.ingest":"true","node.master":"false","node.ml":"false","transport.tcp.compress":"true"}` | Elasticsearch configuration added in a configMap and passed to the Elasticsearch pods as Env. Vars. |
| elastic_config."bootstrap.memory_lock" | string | `"true"` | Elasticsearch enable memory lock to avoid swapping |
| elastic_config."logger.org.elasticsearch.discovery.gce" | string | `"TRACE"` | Elasticsearch GCE discovery log level |
| elastic_config."network.bind_host" | string | `"0.0.0.0"` | Elasticsearch network.bind_host, network address(es) to which the node should bind in order to listen for incoming connections. |
| elastic_config."node.attr.type" | string | `"search"` | Elasticsearch node attribute type |
| elastic_config."node.data" | string | `"true"` | Elasticsearch data node role |
| elastic_config."node.ingest" | string | `"true"` | Elasticsearch ingest node role |
| elastic_config."node.master" | string | `"false"` | Elasticsearch master node role |
| elastic_config."node.ml" | string | `"false"` | Elasticsearch ml node role |
| elastic_config."transport.tcp.compress" | string | `"true"` | Elasticsearch enable compression between nodes |
| elastic_config.ES_JAVA_OPTS | string | `"-Xms2048m -Xmx2048m"` | Elasticsearch JVM options |
| enabled | bool | `true` | Enabling or disabling master nodes |
| fullnameOverride | string | `""` | Overrides the clusterName and nodeGroup when used in the naming of resources. This should only be used when using a single nodeGroup, otherwise you will have name conflicts |
| image.pullPolicy | string | `"IfNotPresent"` | The Kubernetes imagePullPolicy value |
| image.repository | string | `"eu.gcr.io/carrefour-fr-750c11a47fecaeff/empathy-search/elasticsearch-gcp"` | Docker repository for Elasticsearch image |
| image.tag | string | `"6.6.2-memlock1"` | Overrides the image tag whose default is the chart appVersion. |
| imagePullSecrets | list | `[]` | Configuration for imagePullSecrets so that you can use a private registry for your image |
| ingress.annotations | object | `{}` | Annotations for Kubernetes Ingress |
| ingress.className | string | `""` | IngressClass name for ingress exposition |
| ingress.enabled | bool | `false` | Enable Kubernetes Ingress to expose Elasticsearch pods |
| ingress.hosts | list | `[]` | Host and path for Kubernetes Ingress. See values.yaml for an example |
| ingress.tls | list | `[]` | TLS secret for exposing Elasticsearch with https. See values.yaml for an example |
| isMaster | bool | `true` | This property indicates its the master |
| nameOverride | string | `""` | Overrides the clusterName when used in the naming of resources |
| nodeSelector | object | `{}` | Configurable nodeSelector so that you can target specific nodes for your Elasticsearch cluster |
| podAnnotations | object | `{}` | Configurable annotations applied to all Elasticsearch pods |
| podSecurityContext | object | `{}` | Allows you to set the securityContext for the pod |
| podSecurityPolicy.create | bool | `false` | Create a podSecurityPolicy with minimal permissions to run this Helm chart. Be sure to also set rbac.create to true, otherwise Role and RoleBinding won't be created. |
| podSecurityPolicy.name | string | `""` | The name of the podSecurityPolicy to use. If not set and create is true, a name is generated using the fullname template |
| podSecurityPolicy.spec | object | `{}` | Spec to apply to the podSecurityPolicy. See values.yaml for an example |
| prometheus.annotations | object | `{"app":"prometheus-operator","release":"prometheus"}` | Annotations to include in the ServiceMonitor |
| prometheus.dashboard | object | `{"enabled":true,"namespace":"monitoring"}` | Deploy a Grafana Dashboard |
| prometheus.enabled | bool | `true` | Deploy a ServiceMonitor for Prometheus scrapping |
| prometheus.exporter.image | string | `"justwatch/elasticsearch_exporter:1.1.0"` | Exporter image to deploy as a sidecar container |
| rbac.create | bool | `false` | Whether RBAC rules should be created (Role and Rolebinding) |
| replicaCount | int | `3` | Kubernetes replica count for the StatefulSet (i.e. how many pods) |
| resources.limits.cpu | string | `"1000m"` | CPU limits for the StatefulSet |
| resources.limits.memory | string | `"3Gi"` | Memory limits for the StatefulSet |
| resources.requests.cpu | string | `"100m"` | CPU requests for the StatefulSet |
| resources.requests.memory | string | `"3Gi"` | Memory requests for the StatefulSet |
| securityContext | object | `{"capabilities":{"add":["IPC_LOCK","SYS_RESOURCE"]}}` | Allows you to set the securityContext for the container |
| service.port | int | `9200` | Kubernetes service port, used by Ingress to expose Elasticsearch pods |
| service.type | string | `"ClusterIP"` | Kubernetes service type |
| serviceAccount.annotations | object | `{}` | Annotations to add to the service account |
| serviceAccount.create | bool | `true` | Specifies whether a service account should be created |
| serviceAccount.name | string | `""` | The name of the service account to use. If not set and create is true, a name is generated using the fullname template |
| tolerations | list | `[]` | Configurable tolerations |
| volume.annotations | object | `{}` | Annotations for statefulSet volumes |
| volume.storage | string | `"200Gi"` | Storage resources for statefulSet volumes |
| volume.storage_class | string | `"standard"` | Storage class for statefulSet volumes |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.5.0](https://github.com/norwoodj/helm-docs/releases/v1.5.0)