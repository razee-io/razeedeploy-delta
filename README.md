# Razeedeploy-delta

[![Build Status](https://travis-ci.com/razee-io/razeedeploy-delta.svg?branch=master)](https://travis-ci.com/razee-io/razeedeploy-delta)
[![Dependabot Status](https://api.dependabot.com/badges/status?host=github&repo=razee-io/razeedeploy-delta)](https://dependabot.com)

> **⚠️ DEPRECATION NOTICE**
>
> This project is deprecated and is no longer actively maintained.
> The razee.io components are no longer being developed or supported.
>
> ## Alternatives
>
> **For IBM Cloud users:**
>
> - [IBM Cloud Continuous Delivery](https://www.ibm.com/products/continuous-delivery)
>
> **For open source users:**
>
> - [Argo CD](https://argo-cd.readthedocs.io/en/stable/)
> - [Flux CD](https://fluxcd.io/)
> - [Tekton](https://tekton.dev/)

## Building from Source

If you need to build the container images yourself, follow these instructions:

### Prerequisites

- Docker or Podman installed
- Access to a container registry (Docker Hub, Quay.io, etc.)

### Build Instructions

1. Clone the repository:

   ```bash
   git clone <repository-url>
   cd razeedeploy-delta
   ```

2. Build the container image:

   ```bash
   docker build -t <your-registry>/razeedeploy-delta:<tag> .
   ```

3. Push the image to your registry:

   ```bash
   docker push <your-registry>/razeedeploy-delta:<tag>
   ```

4. Update your Kubernetes manifests to use your custom image.

## Running Install/Remove Job Manually

1. Download [Job](https://github.com/razee-io/razeedeploy-delta/releases/latest/download/job.yaml)
1. Replace `{{ NAMESPACE }}` with the namespace you want everything installed into/removed from.
1. Replace `{{ COMMAND }}` with either `install` or `remove`
1. Replace `{{ ARGS_ARRAY }}` with and array of the options you want to run. eg. `["--rr", "--wk", "-a"]`
1. Run `kubectl apply -f job.yaml`

### Install Job Options

[Code Reference](https://github.com/razee-io/razeedeploy-delta/blob/master/src/install.js#L35-L63)

```text
-h, --help
    : help menu
-d, --debug=''
    : loop to keep the pod running. Does not attempt install (Default 5 minutes)
-n, --namespace=''
    : namespace to populate razeedeploy resources into (Default 'razeedeploy')
-s, --file-source=''
    : url that razeedeploy-job should source razeedeploy resource files from (Default 'https://github.com/razee-io')
--fp, --file-path=''
    : the path directly after each component, e.g. \${fileSource}/WatchKeeper/\${filePath}. (Default 'releases/{{install_version}}/resource.yaml')
-r, --registry=''
    : image registry that razeedeploy-job should install razeedeploy images with (Default 'quay.io/razee/')
--wk, --watchkeeper='', --watch-keeper=''
    : install watchkeeper at a specific version (Default 'latest')
--cs, --clustersubscription=''
    : install clustersubscription at a specific version (Default 'latest')
--rd-url, --razeedash-url=''
    : url that watchkeeper should post data to (Default '\${--razeedash-api}/api/v2' if provided)
--rd-api, --razeedash-api=''
    : razee api baseUrl (Default '\${--razeedash-url}.origin' if provided)
--rd-org-key, --razeedash-org-key=''
    : org key used to authenticate with razee
--rd-cluster-id, --razeedash-cluster-id=''
    : cluster id to be used as the cluster id in RazeeDash (Default 'namespace.metadata.uid')
--rd-cluster-metadata64, --razeedash-cluster-metadata64=''
    : base64 encoded JSON object of cluster metadata entries {key: value, ...}. To be stored into watch-keeper-cluster-metadata ConfigMap and sent to RazeeDash
--rr, --remoteresource=''
    : install remoteresource at a specific version (Default 'latest')
--rrs3, --remoteresources3=''
    : install remoteresources3 at a specific version (Default 'latest')
--rrs3d, --remoteresources3decrypt=''
    : install remoteresources3decrypt at a specific version (Default 'latest')
--mtp, --mustachetemplate=''
    : install mustachetemplate at a specific version (Default 'latest')
--iw, --impersonationwebhook=''
    : install impersonation webhook at a specific version (Default 'latest'). When remote resource controller and/or mustache template controller are installed, this webhook will be installed even if this flag is not set
--iw-cert
    : base64 encoded JSON object of webhook certificate in PEM format. The corresponding keys for CA cert, server cert, and server key are: 'ca', 'server', 'key'. If CA cert is missing, the server cert will be used as CA cert. Each key holds the base64 encoded representation of the corresponding PEM. And the whole JSON file is encoded in base64.
--ffsld, --featureflagsetld=''
    : install featureflagsetld at a specific version (Default 'latest')
--er, --encryptedresource=''
    : install encryptedresource at a specific version (Default 'latest')
--ms, --managedset=''
    : install managedset at a specific version (Default 'latest')
-f, --force
    : overwrite prerequisite configuration already installed on the cluster (Default false)
-a, --autoupdate
    : will create a remoteresource that will pull and keep specified resources updated to latest (even if a version was specified). if no resources specified, will do all known resources
```

### Remove Job Options

[Code Reference](https://github.com/razee-io/razeedeploy-delta/blob/master/src/remove.js#L33-L49)

```text
-h, --help
    : help menu
-d, --debug=''
    : loop to keep the pod running. Does not attempt removal (Default 5 minutes)
-n, --namespace=''
    : namespace to remove razeedeploy resources from (Default 'razeedeploy')
--dn, --delete-namespace
    : include namespace as a resource to delete (Default false)
-s, --file-source=''
    : url that razeedeploy-job should source razeedeploy resource files from (Default 'https://github.com')
--fp, --file-path=''
    : the path directly after each component, e.g. ${fileSource}/WatchKeeper/${filePath}. (Default 'releases/{{install_version}}/resource.yaml')
-t, --timeout
    : time (minutes) before failing to delete CRD (Default 5)
-a, --attempts
    : number of attempts to verify CRD is deleted before failing (Default 5)
-f, --force
    : force delete the CRD and CR instances without allowing the controller to clean up children (Default false)
```

## Ensure Exist Resources

Some resources created by this job are considered `ensure exist`. That means
if they have been created/modified already, the install job wont replace whats
already there. If you would like to re-install RazeeDeploy on a cluster completely
from scratch, you must first delete these resources:

- PreReqs: (all installs)
  - ServiceAccount: `razeedeploy-sa`
  - ClusterRole: `razeedeploy-admin-cr`
  - ClusterRoleBinding: `razeedeploy-rb`
- WatchKeeper Config: (only when installing watchkeeper)
  - ServiceAccount: `watch-keeper-sa`
  - ClusterRole: `razee-cluster-reader`
  - ClusterRoleBinding: `watch-keeper-rb`
  - ConfigMap: `watch-keeper-config`
  - ConfigMap: `razee-identity`
  - Secret: `razee-identity`
  - ConfigMap: `watch-keeper-limit-poll`
  - ConfigMap: `watch-keeper-non-namespaced`
  - NetworkPolicy: `watch-keeper-deny-ingress`
- ClusterSubscription Config: (only when installing clustersubscription)
  - ConfigMap: `razee-identity`
  - Secret: `razee-identity`
- ImpersonationWebhook:
  - Secret: `impersonation`
  - MutatingWebhookConfiguration: `impersonation-webhook`

## Razeeupdate

The `update` command is used for the razeeupdate cronjob returned by the Operators System Subscription found in Razeedash-api. This will update Cluster Subscription, Remote Resource and Watch-Keeper if they already exist in the cluster
