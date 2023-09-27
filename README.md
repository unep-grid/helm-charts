# MapX meets k8s

Set of instructions to deploy [MapX](https://github.com/unep-grid/mapx) in a Kubernetes cluster.

⚠ Please note that the versions used in this document may not be the latest by the time you read it. If `--version` is not specified, the latest version is used automatically.

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

#### Cert-manager

```sh
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.13.0 \
  --set installCRDs=true \
  --wait
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
helm install \
  longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --version 1.5.1 \
  --wait
```

List of alternative CSI drivers that could be used to manage storage: <https://kubernetes-csi.github.io/docs/drivers.html>

#### Traefik

```sh
helm install \
  traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --version 24.0.0 \
  --set ports.web.redirectTo=websecure \
  --wait
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
helm install \
  -f ./utilities/kube_prometheus_stack/overrides.yaml \
  -f ./helm-chart/mapx/values.yaml \
  prometheus-stack  prometheus-community/kube-prometheus-stack \
  --namespace prometheus-stack \
  --create-namespace \
  --version 51.0.3 \
  --wait
```

The installation of this set of Grafana dashboards is recommended: <https://github.com/dotdc/grafana-dashboards-kubernetes/>

### MapX

⚠ Recommendation from [Sokube](https://www.sokube.io/en/home): databases should not be deployed in a k8s cluster especially for production.

Make sure you have access to a functional MapX database from the cluster before deploying MapX.

Deployment from the local Helm chart:

```sh
helm install \
  dev helm-chart/mapx/ \
  --namespace mapx-dev \
  --create-namespace \
  -f helm-chart/mapx/values.yaml
```

Deployment from the online [Helm repository](https://git.unepgrid.ch/mapx/-/packages/helm/mapx/):

```sh
helm install \
  dev git.unepgrid.ch/mapx \
  --namespace mapx-dev \
  --create-namespace \
  -f helm-chart/mapx/values.yaml
```

If a pre-populated instance of MapX is used, import the `userdata` folder into the dedicated volume (e.g., `NFS` for the UniGe instance).

## Additional information

The Helm package has been reviewed by [SixSq](https://sixsq.com/) in May and July 2023.
