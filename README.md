# MapX meets k8s

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

2. Fill `./helm-chart/mapx/values.yaml` with the values needed for `kube-prometheus-stack` and MapX to work properly.

### Add Helm repositories

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo add longhorn https://charts.longhorn.io
helm repo add traefik https://traefik.github.io/charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Cluster utilities helm charts

#### Cert-manager

```sh
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.3 \
  --set installCRDs=true \
  --wait
```

In order to begin issuing certificates, an `Issuer`/`ClusterIssuer` resource needs to be set up.
ClusterIssuers for Let's Encrypt (staging and production) are available in `./utilities/cert_manager/`.
Both of these ClusterIssuers are configured to use the [HTTP01](https://cert-manager.io/docs/configuration/acme/http01/) challenge provider.
`spec.acme.email` must be filled before deploying these manifests in the cluster. The email is required by Let's Encrypt and used for certificate expiration and update notifications.

```sh
kubctl apply -f ./utilities/cert_manager/
```

#### Longhorn

```sh
helm install \
  longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --version 1.5.1 \
  --wait
```

#### Traefik

```sh
helm install \
  traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --version 23.2.0 \
  --wait
```

#### Kube-prometheus-stack

Please configure `./utilities/kube_prometheus_stack/prometheus-stack-overrides.yml` as you wished. This will deploy a fully working persisted Prometheus and Grafana service. Generate certificate and make it reachable through a Traefik ingress. SMTP configuration should be passed also for now I'm reusing MapX Docker SMTP account. Grafana dashboards are loaded from the chart itself with a lot of usefull dashboards for Kubernetes. We can create/customize them as we do for MapX in prod.

```sh
helm install \
  -f ./utilities/kube_prometheus_stack/overrides.yml \
  -f ./helm-chart/mapx/values.yaml \
  prometheus-stack  prometheus-community/kube-prometheus-stack \
  --namespace prometheus-stack \
  --create-namespace \
  --version 48.2.2 
```

### MapX

⚠ Following [Sokube](https://www.sokube.io/en/home) recommendations, MapX database is not deployed in the k8s cluster. Make sure a working MapX database is accessible from the cluster.

```sh
helm install \
  dev  helm-chart/mapx/ \
  --namespace mapx-dev \
  --create-namespace \
  -f helm-chart/mapx/values.yaml
```

If a pre-populated instance of MapX is used, import the `userdata` folder into the dedicated volume (e.g. `NFS` for the UniGe instance).

## Additional information

The Helm package has been reviewed by [SixSq](https://sixsq.com/) in May 2023.
