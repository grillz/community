# Dynamic Audit Policy
This document covers the audit policy for the Dynamic Auditing feature.

## Audit Class 
The AuditClass object is a set of rules that apply requests to classes. Each rule is its own object, and  
multiple rules may match a given request.

```yaml
apiVersion: auditregistration.k8s.io/v1alpha1
kind: AuditClass
metadata:
  name: my-audit-class
spec:
  users:
  - myuser
  userGroups:
  - mygroup
  verbs:
  - get
  resources:
  - group: apps.k8s.io
    resources:
    - deployments
  namespaces:
  - my-app
  nonResourceURLs:
  - "/metrics"
```

```go
// AuditClass is a set of rules that categorize requests
type AuditClass struct {
    // The users (by authenticated user name) this rule applies to.
	// An empty list implies every user.
	// +optional
	Users []string
	// The user groups this rule applies to. A user is considered matching
	// if it is a member of any of the UserGroups.
	// An empty list implies every user group.
	// +optional
	UserGroups []string

	// The verbs that match this rule.
	// An empty list implies every verb.
	// +optional
	Verbs []string

	// Rules can apply to API resources (such as "pods" or "secrets"),
	// non-resource URL paths (such as "/api"), or neither, but not both.
	// If neither is specified, the rule is treated as a default for all URLs.

	// Resources that this rule matches. An empty list implies all kinds in all API groups.
	// +optional
	Resources []GroupResources
	// Namespaces that this rule matches.
	// The empty string "" matches non-namespaced resources.
	// An empty list implies every namespace.
	// +optional
	Namespaces []string

	// NonResourceURLs is a set of URL paths that should be audited.
	// *s are allowed, but only as the full, final step in the path.
	// Examples:
	//  "/metrics" - Log requests for apiserver metrics
	//  "/healthz*" - Log all health checks
	// +optional
	NonResourceURLs []string
}

// GroupResources represents resource kinds in an API group.
type GroupResources struct {
	// Group is the name of the API group that contains the resources.
	// The empty string represents the core API group.
	// +optional
	Group string
	// Resources is a list of resources this rule applies to.
	//
	// For example:
	// 'pods' matches pods.
	// 'pods/log' matches the log subresource of pods.
	// '*' matches all resources and their subresources.
	// 'pods/*' matches all subresources of pods.
	// '*/scale' matches all scale subresources.
	//
	// If wildcard is present, the validation rule will ensure resources do not
	// overlap with each other.
	//
	// An empty list implies all resources and subresources in this API groups apply.
	// +optional
	Resources []string
	// ResourceNames is a list of resource instance names that the policy matches.
	// Using this field requires Resources to be specified.
	// An empty list implies that every instance of the resource is matched.
	// +optional
	ResourceNames []string
}
```

## Class Rule
A ClassRule is a per sink rule that says what to do with classes. This allows the sinks to share classes 
and keeps their definitions short.

```yaml
apiVersion: auditregistration.k8s.io/v1alpha1
kind: AuditSink
metadata:
  name: <name>
spec:
  policy:
    level: RequestResponse
    stages:
    - ResponseStarted
    classRules:
    - name: my-audit-class
      level: Metadata
      stages:
      - RequestRecieved
```

```go
// Policy defines the configuration of how audit events are logged
type Policy struct {
	// The Level that all requests are recorded at.
	// available options: None, Metadata, Request, RequestResponse
	// required
	Level Level

	// Stages is a list of stages for which events are created.
	// +optional
    Stages []Stage
    
    // ClassRules define how classes should be handled per sink
    // +optional
    ClassRules []ClassRule
}

// ClassRule defines how a class is handled per sink
type ClassRule struct {
    // Name of the AuditClass object
    Name string
	// The Level that all requests are recorded at.
	// available options: None, Metadata, Request, RequestResponse
	// required
	Level Level

	// Stages is a list of stages for which events are created.
	// +optional
    Stages []Stage
}
```