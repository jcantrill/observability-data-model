# Cluster Logging and OpenTelemetry

This is the protocol and semantic conventions documentation for Red Hat OpenShift Logging's OTEL support 
starting with Logging v6.0 which is considered **Tech-Preview**.  This document should be
considered as a work in progress and is subject to change until OTEL support graduates to **General Acceptance**. 

## Forwarding and Ingestion Protocol

Red Hat OpenShift Logging provides a log collection and forwarding solution that is capable of writing logs to OpenTelemetry endpoints using OTLP. [OTLP](https://opentelemetry.io/docs/specs/otlp/) is the *protocol* for encoding, transporting, and delivering telemetry data.  This product additionally is capable of providing a Loki storage deployment that provides an OTLP endpont to ingest log streams.  This document defines the semantic conventions associated with the logs collected from the various sources of an OpenShift cluster.

**Note:** Logs are forwarded using OTLP/HTTP as defined by the OpenTelemetry Observability Framework.  It uses Protobuf payloads encoded in JSON format.


## Semantic Conventions

The log collector provided by this solution collects the following log streams:

* Container logs
* Cluster node journal logs
* Cluster node auditd logs 
* Kubernetes and OpenShift API server logs
* OpenShift Virtual Network (OVN) logs

These streams are forwarded using the sementatic conventions defined by [OpenTelemetry semantic attributes](https://github.com/open-telemetry/semantic-conventions/tree/main/docs). The semantic conventions in OpenTelemetry define a *Resource* as an immutable representation of the entity producing telemetry as *Attributes*. For example, a process producing telemetry that is running in a container has a container_name, a cluster_id, a pod_name, a namespace, and possibly a deployment or app_name. All of these *Attributes* are included in the *Resource* object.  This grouping and reducing of common attributes is a powerful tool when sending logs as telemetry data.


The following sections define the attributes that are generally forwarded.


### Common Fields & Attributes

All log streams include the following [logData](https://opentelemetry.io/docs/specs/otel/logs/data-model/#log-and-event-record-definition) fields:

```
body
observedTimeUnixNano
severityNumber           #N/A for Kubernetes, OpenShift, OVN audit logs
severityText             #N/A for Kubernetes, OpenShift, OVN audit logs
timeUnixNano
```


All log streams include the minimal set of the following [resource](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/resource/sdk.md) attributes:

```
openshift.cluster.uid
openshift.log.source
openshift.log.type
```

All log streams include the following log attributes:

```
openshift.labels.*  # spec identifies these as resource attributes...
```



### Kubernetes Container Logs

Some of the following attributes (i.e. *k8s.(cronjob|daemonset|deployment|job|replicaset|statefulset).name*) are 
[conditionally](https://opentelemetry.io/docs/specs/semconv/general/attribute-requirement-level/#conditionally-required) 
forwarded based upon the log stream.  For example, logs from a pod that was created as a result of defining a cronjob may only include the *k8s.cronjob.name* attribute.

Resource Attributes:
```
k8s.node.name
k8s.node.uid

k8s.namespace.name
k8s.container.name
k8s.container.id
k8s.pod.name
k8s.pod.uid

k8s.cronjob.name 
k8s.daemonset.name 
k8s.deployment.name 
k8s.job.name
k8s.replicaset.name 
k8s.statefulset.name 
```

Log Attributes:
```
k8s.pod.labels.*    # spec identifies these as resource attributes...
log.iostream
```

The following attributes, along with their OTEL equivalents, are added to support minimal backwards compatibility with the [ViaQ](https://github.com/openshift/cluster-logging-operator/blob/release-6.0/docs/reference/datamodels/viaq/v1.adoc) data model. These attributes should be considered deprecated
and will be removed one release after **General Acceptance** of Red Hat OpenShift Logging.


| ViaQ | OTEL |
|------|---|
|`kubernetes.container_name` |`k8s.container.name`|
|`kubernetes.namespace_name` | `k8s.namespace.name`|
|`kubernetes.pod_name`       | `k8s.pod.name`|
|`log_source`                | `openshift.log.source`|
|`log_type`                  | `openshift.log.type`|


### Kubernetes and OpenShift API, OVN Logs

Resource Attributes:
```
k8s.node.name
k8s.node.uid
```

Log Attributes:
```
k8s.event.level
k8s.event.stage
k8s.event.user_agent
k8s.event.request.uri
k8s.event.response.code
k8s.event.annotations.*
k8s.user.username
k8s.user.groups
```

### Cluster Node Journal Logs

Resource Attributes:
```
k8s.node.name
k8s.node.uid

process.executable.name
process.executable.path
process.command_line
process.pid
service.name
```

Log Attributes:
```
```

### Cluster Node Auditd Logs

Resource Attributes:
```
k8s.node.name
k8s.node.uid
```

Log Attributes:
```
```

## References
* [Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/)
* [Logs Data Model](https://opentelemetry.io/docs/specs/otel/logs/data-model/)
* [General Logs Attributes](https://opentelemetry.io/docs/specs/semconv/general/logs/)
* [Cluster Logging OTEL Support](https://github.com/openshift/enhancements/pull/1684)