# Cluster Logging and OpenTelemetry

This is the protocol and semantic conventions documentation for Red Hat OpenShift Logging.

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


The following sections define the attributes that are forwarded.


### Common Attributes

All log streams include a minimal set of attributes.

Resource Attributes:
```
cluster.id
node.name
openshift.log.source
openshift.log.type
```

Log Attributes:
```
body
timeUnixNano
severityNumber
observedTimeUnixNano
```

### Kubernetes Container Logs

Resource Attributes:
```
k8s.cluster.uid
k8s.container.name
k8s.namespace.name
k8s.pod.name
```

Log Attributes:
```
k8s.container.id
k8s.node.name
k8s.pod.uid
```

The following attributes, along with their OTEL equivalents, are added to support minimal backwards compatibility with the [ViaQ](https://github.com/openshift/cluster-logging-operator/blob/release-6.0/docs/reference/datamodels/viaq/v1.adoc) data model. These attributes should be considered deprecated
and will be removed in a future update.

```
kubernetes.container_name -> k8s.container.name
kubernetes.namespace_name -> k8s.namespace.name
kubernetes.pod_name       -> k8s.pod.name
log_source                -> openshift.log.source
log_type                  -> openshift.log.type
```

### Kubernetes and OpenShift API, OVN Logs

Log Attributes:
```
http.response.status.code
http.request.method
url.full
```

### Cluster Node Journal Logs

Log Attributes:
```
system.cgroup
system.cmdline
syslog.facility
syslog.identifier
system.invocation.id
syslog.procid
system.slice
system.unit
```

### Cluster Node Auditd Logs


Log Attributes:
```
```

## References
* [Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/)
* [Logs Data Model](https://opentelemetry.io/docs/specs/otel/logs/data-model/)
* [General Logs Attributes](https://opentelemetry.io/docs/specs/semconv/general/logs/)
* [Cluster Logging OTEL Support](https://github.com/openshift/enhancements/pull/1684)