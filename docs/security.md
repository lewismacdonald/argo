# Security

## Workflow Controller Security

This has three parts.

### Controller Permissions

The controller has permission (via Kubernetes RBAC + its config map) with either all namespaces (cluster-scope install) or a single [managed namespace](managed-namespace.md) (namespace-install), notably:

* List/get/update workflows, and cron-workflows.
* Create/get/delete pods, PVCs, and PDBs.
* List/get template, config maps, service accounts, and secrets.

See [workflow-controller-clusterrole.yaml](manifests/cluster-install/workflow-controller-rbac/workflow-controller-clusterrole.yaml) or [workflow-controller-role.yaml](manifests/namespace-install/workflow-controller-rbac/workflow-controller-role.yaml)

### User Permissions

Users minimally need permission to create/read workflows. The controller will then create workflow pods (config maps etc) on behalf of the users, even if the user does not have permission to do this themselves. The controller will only create workflow pods in the workflow's namespace.

A way to think of this is that, if the user has permission to create a workflow in a namespace, then it is OK to create pods or anything else for them in that namespace.

If the user only has permission to create workflows, then they will be typically unable to configure other necessary resources such as config maps, or view the outcome of their workflow. This is useful when the user is a service.  

!!! Warning
    If you allow users to create workflows in the controller's namespace (typically `argo`), it may be possible for users to modify the controller itself.  In a namespace-install the managed namespace should therefore not be the controller's namespace.

You can typically further restrict what a user can do to just being able to submit workflows from templates using [the workflow requriments feature](workflow-restrictions.md).

### Workflow Pod Permissions

Workflow pods run using either:

* The `default` service account.
* The service account declared in the workflow spec.

There is no restriction on which service account in a namespace may be used.

This service account typically needs the following permissions:

* Get/watch/patch pods.
* Get/watch pod logs.

See [workflow-role.yaml](manifests/quick-start/base/workflow-role.yaml).

Different service accounts should be used if a workflow pod needs to have elevated permissions, e.g. to create other resources.

The main container will have the service account token mounted , allowing the main container to patch pods (amongst other permissions). Set `automountServiceAccountToken` to false to prevent this. See [fields](fields.md).

By default, workflows pods run as `root`. To further secure workflow pods, set the [workflow pod security context](workflow-pod-security-context.md).

You should configure the controller with the correct [workflow executor](workflow-executors.md) for your trade off between security and scalabily.

These settings can be set by default using [workflow defaults](default-workflow-specs.md).

## Argo Server Security

Argo Server implements security in three layers.

Firstly, you should enable [transport layer security](tls.md) to ensure your data cannot be read in transit.

Secondly, you should enable an [authentication mode](argo-server.md#auth-mode) to ensure that you do not run workflows from unknown users.

Finally, you should configure the `argo-server` role and role binding with the correct permissions.

### Read-Only

You can achieve this by configuring the `argo-server` role ([example](https://github.com/argoproj/argo/blob/master/manifests/namespace-install/argo-server-rbac/argo-server-role.yaml) with only read access (i.e. only `get`/`list`/`watch` verbs).
