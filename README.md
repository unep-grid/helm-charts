# MapX meets k8s

Set of instructions to deploy [MapX](https://github.com/unep-grid/mapx) in a Kubernetes cluster.

## Prerequisites

Install:

- [kubctl](https://kubernetes.io/docs/tasks/tools/#kubectl) ([cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/))
- [helm](https://helm.sh/docs/intro/install/)

Optional but useful tools:

- [kubectx + kubens](https://github.com/ahmetb/kubectx#installation)
- [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)

## Deployment

1. Load _kubeconfig_ file to configure access to the cluster:

   ```sh
   export KUBECONFIG=<path to kubeconfig>
   ```

2. Fill `./helm-chart/mapx/values.yaml` with the values required by MapX to be deployed in the k8s cluster.

3. Fill `./utilities/kube_prometheus_stack/overrides.yaml` with the values required by `kube-prometheus-stack` to be deployed in the k8s cluster.

### Add Helm repositories

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo add longhorn https://charts.longhorn.io
helm repo add traefik https://traefik.github.io/charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add git.unepgrid.ch https://git.unepgrid.ch/api/packages/mapx/helm
helm repo update
```

### Cluster utilities helm charts

Helm provides a way to perform an install-or-upgrade as a single command. Use `helm upgrade` with the `--install` command. This will cause Helm to see if the release is already installed. If not, it will run an install. If it is, then the existing release will be upgraded.

```sh
helm upgrade \
  --install \
  <release_name> <chart_directory> \
  --namespace <namespace_name> \
  --create-namespace \
  --version <version> \
  --values <values file> \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

```sh
--atomic               if set, upgrade process rolls back changes made in case of failed upgrade. The --wait flag will be set automatically if --atomic is used
--cleanup-on-fail      allow deletion of new resources created in this upgrade when upgrade fails
--debug                enable verbose output
--timeout duration     time to wait for any individual Kubernetes operation (like Jobs for hooks) (default 5m0s)
--wait                 if set, will wait until all Pods, PVCs, Services, and minimum number of Pods of a Deployment, StatefulSet, or ReplicaSet are in a ready state before marking the release as successful. It will wait for as long as --timeout
```

⚠ Please note that the versions used in this document may not be the latest by the time you read it. If `--version` is not specified, the latest version is used automatically.

#### Cert-manager

```sh
helm upgrade \
  --install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.13.2 \
  --set installCRDs=true \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

To begin issuing certificates, an `Issuer`/`ClusterIssuer` resource needs to be set up.
ClusterIssuers for Let's Encrypt (staging and production) are available in `./utilities/cert_manager/`.
Both of these ClusterIssuers are configured to use the [HTTP01](https://cert-manager.io/docs/configuration/acme/http01/) challenge provider.

`spec.acme.email` must be filled before deploying these manifests in the cluster. The email is required by Let's Encrypt and used for certificate expiration and update notifications.

```sh
kubctl apply -f ./utilities/cert_manager/
```

#### Longhorn

[Longhorn](https://github.com/longhorn/longhorn) is the CSI driver recommended by [SixSq](https://sixsq.com/) to deploy MapX in a Kubernetes cluster.

Useful reading about the operation and deployment of Longhorn: <https://www.exoscale.com/syslog/longhorn-sks/>

```sh
helm upgrade \
  --install \
  longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --version 1.5.2 \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

List of alternative CSI drivers that could be used to manage storage: <https://kubernetes-csi.github.io/docs/drivers.html>

#### Traefik

```sh
helm upgrade \
  --install \
  traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --version 25.0.0 \
  --set ports.web.redirectTo=websecure \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

External Traefik IP to configure in DNS:

```sh
export TRAEFIK_EXTERNAL_IP=$(kubectl get services \
    traefik \
    --namespace traefik \
    --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

#### Kube-prometheus-stack

⚠ `./utilities/kube_prometheus_stack/overrides.yaml` uses values from `./helm-chart/mapx/values.yaml`. These two files must be completed before continuing with the deployment.

```sh
helm upgrade \
  --install \
  prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace prometheus-stack \
  --create-namespace \
  --version 52.1.0 \
  --values ./utilities/kube_prometheus_stack/overrides.yaml \
  --values ./helm-chart/mapx/values.yaml \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

The installation of this set of Grafana dashboards is recommended: <https://github.com/dotdc/grafana-dashboards-kubernetes/>

### MapX

⚠ Recommendation from [Sokube](https://www.sokube.io/en/home): databases should not be deployed in a k8s cluster especially for production.

Make sure you have access to a functional MapX database from the cluster before deploying MapX.

Deployment from the local Helm chart:

```sh
helm upgrade \
  --install \
  <release_name> helm-chart/mapx/ \
  --namespace <namespace_name> \
  --create-namespace \
  --values helm-chart/mapx/values.yaml \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

Deployment from the online [Helm repository](https://git.unepgrid.ch/mapx/-/packages/helm/mapx/):

```sh
helm upgrade \
  --install \
  <release_name> git.unepgrid.ch/mapx \
  --namespace <namespace_name> \
  --create-namespace \
  --values helm-chart/mapx/values.yaml \
  --atomic \
  --cleanup-on-fail \
  --timeout 3m \
  --debug=true
```

If a pre-populated instance of MapX is used, import the `userdata` folder into the dedicated volume (e.g., `NFS` for the UniGe instance).

## Additional information

The Helm package has been reviewed by [SixSq](https://sixsq.com/) in May and July 2023.
