# MapX meets k8s

## Requirements

Install:

- [kubctl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [helm](https://helm.sh/docs/intro/install/)

Optional but useful tools:

- [kubectx + kubens](https://github.com/ahmetb/kubectx#installation)
- [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)
- [kompose](https://kompose.io/installation/)

## Cluster requirements

- [cert-manager](https://cert-manager.io/)
- [Traefik k8s CRD](https://doc.traefik.io/traefik/reference/dynamic-configuration/kubernetes-crd/)

## How to deloy MapX?

⚠ Following [Sokube](https://www.sokube.io/en/home) recommendations, MapX database is not deployed in the k8s cluster. Make sure a working MapX database is accessible from the cluster.

1. Load _kubeconfig_ file to configure access to the cluster:

   ```sh
   export KUBECONFIG=<path to kubeconfig>
   ```

2. Fill `helm-chart/mapx/values.yaml` with the values needed for MapX to work properly.

3. Deploy MapX:

   ```sh
   helm install <namespace> helm-chart/mapx/ -f helm-chart/mapx/values.yaml
   ```

## Additional information

The Helm package has been reviewed by [SixSq](https://sixsq.com/) in May 2023.
