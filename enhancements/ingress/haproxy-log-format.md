---
title: logging-api-extension
authors:
  - "@rohara"
reviewers:
  - "@Miciah"
  - "@knobunc"
approvers:
  - "@Miciah"
  - "@knowbunc"
creattion-date: 2025-05-01
last-updated: 2025-06-17
status:
see-also: logging-api
replaces:
superseded-by:
---

# Ingress Logging API extension

## Summary

This enhancement extends the IngressController API to allow the user
to configure different log formats for HAProxy. Currently, users may
only specify the 'HttpLogFormat', which is only used for HTTP traffic
with no SSL termination or passthrough. This enhancement would
introduce both 'HttpsLogFormat' and 'TcpLogFormat' such that users
can define the log formats.

## Motivation

Users desire the ability to log extra SSL information when
applicable. This is useful when debugging SSL related issues. As such,
the user may want to define the log format similarly to how the API
allows for HTTP log format.

For completeness, a user may also want to define the log format for
TCP logging. This is useful SSL passthrough.

### User Stories

This enhancement will allow users to customize log formats for
HttpsLogFormat and TcpLogFormat, similarly to how HttpLogFormat can
currently be customized. Additionally, using HttpsLogFormat will allow
users to log useful SSL-related information that may assist in
troubleshooting.

### Goals

1. Provide an option to customize HTTPS log format.
2. Provide an option to customize TCP log format.

### Non-goals

Since all access logs are collected at a single endpoint, it is beyond
the scope of this proposal to allow different log formats to be
collected in distinct logs. All logs will be collected into a single
log file which may contain different log formats.

## Proposal

The `AccessLogging` type allows specifying a destination using the `Destination`
field, which has type `LoggingDestination`, and the log format using the
`HttpLogFormat` field, which has type `string`:

```go
// AccessLogging describes how client requests should be logged.
type AccessLogging struct {
        // destination is where access logs go.
        //
        // +kubebuilder:validation:Required
        // +required
        Destination LoggingDestination `json:"destination"`

        // httpLogFormat specifies the format of the log message for an HTTP
        // request.
        //
        // If this field is empty, log messages use the implementation's default
        // HTTP log format.  For HAProxy's default HTTP log format, see the
        // HAProxy documentation:
        // http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#8.2.3
        //
        // +optional
        HttpLogFormat string `json:"httpLogFormat,omitempty"`

        // httpsLogFormat specifies the format of the log messages for
	// an HTTPS request
	//
	// If this field is empty, log messages use the implementation's default
        // HTTPS log format.  For HAProxy's default HTTP log format, see the
	// HAProxy documentation:
        // http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#8.2.3
        //
        // +optional
        HttpsLogFormat string `json:"httpLogFormat,omitempty"`

        // tcpLogFormat specifies the format of the log messages for a
        // TCP request.
        //
        // If this field is empty, log messages use the implementation's default
        // TCP log format.  For HAProxy's default HTTP log format, see the
        // HAProxy documentation:
        // http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#8.2.3
        //
        // +optional
        TcpLogFormat string `json:"httpLogFormat,omitempty"`
}
```

The user may specify `spec.logging.access.httpLogFormat` to customize
the HTTP log format. The user may also specify
`spec.logging.access.httpsLogFormat` to customize the HTTPS log
format. The user may also specify `spec.logging.access.tcpLogFormat`
to customize TCP log format. For example, the following is the
definition for an IngressController that customizes each of the log
formats (HTTP, HTTPS, and TCP):

```yaml
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: default
  namespace: openshift-ingress-operator
spec:
  replicas: 2
  endpointPublishingStrategy:
    type: NodePortService
  logging:
    access:
      httpLogFormat:
      httpsLogFormat:
      tcpLogFormat:
```

Each of the log format fields contain a free-form format string, so
node of these are validated.

### Workflow Description

### API Extensions

### Topology Considerations

#### Hypershift / Hosted Control Planes

#### Standalone Clusters

#### Single-node Deployments or MicroShift

### Implementation Details/Notes/Constraints

HAProxy can forward access logs to a syslog endpoint, and openshift-router
recognizes environment variables to enable this feature and specify
the log formats. Currently the only log format is HttpLogFormat
(`ROUTER_SYSLOG_FORMAT`). Additional environment variables for
HttpsLogFormat (`ROUTER_HTTPS_LOG_FORMAT`) and TcpLogFormat
(`ROUTER_TCP_LOG_FORMAT`) will be recognized by the ingress operator
to configure custom log formats for HTTPS and TCP logs,
respespectively.

Note that the current environment variable for HttpLogFormat
(`ROUTER_SYSLOG_FORMAT`) is non-specific and should be renamed to
(`ROUTER_HTTP_LOG_FORMAT`). The old environment variable
(`ROUTER_SYSLOG_FORMAT`) should still be honored, but the new
environment variable (`ROUTER_HTTP_LOG_FORMAT`) will take precedence.

### Risks and Mitigations

The meaning of the specified log formats depends on the underlying
implementation.  If we were to change away from HAProxy, the semantics of the
format would likely change.  We can mitigate this risk by documenting
it.

### Drawbacks

N/A

## Alternatives (Not Implemented)

N/A

## Open Questions [optional]

N/A

## Test Plan

There are no planned e2e or integration tests for this
enhancement. This is largely due to the fact that log format is a
free-form string.

## Graduation Criteria

N/A

### Dev Preview -> Tech Preview

### Tech Preview -> GA

### Removing a deprecated feature

## Upgrade / Downgrade Strategy

## Version Skew Strategy

N/A

## Operational Aspects of API Extensions

## Support Procedures

If any format string contains an invalid variable, HAProxy will exit
immediately without providing a descriptive log message.

## Infrastructure Needed [optional]
