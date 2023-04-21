# Deploy MapX to a local k3d cluster

Local deployment based on [k3d](https://k3d.io/), a lightweight wrapper to run [k3s](https://k3s.io/) in docker. k3d makes it very easy to create single- and multi-node k3s clusters in docker, e.g. for local development on Kubernetes.

## Requirements

Install:

- [docker](https://docs.docker.com/get-docker/)
- [k3d](https://k3d.io/)
- [kubctl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [helm](https://helm.sh/docs/intro/install/)

Optional but useful tools:

- [kubectx + kubens](https://github.com/ahmetb/kubectx#installation)
- [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)
- [kompose](https://kompose.io/installation/)

### How to deloy MapX?

1. Create a local cluster:

   ```sh
   k3d cluster create --config k3d-config.yaml
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
   ```

2. Identify the IP address of the local cluster's load balancer:

   ```sh
   kubectl get svc -A
   ```

3. Add MapX local subdomains in `/etc/hosts`:

   ```txt
   <IP> app.mapx.local
   <IP> api.mapx.local
   <IP> search.mapx.local
   <IP> geoserver.mapx.local
   <IP> traefik.mapx.local
   ```

4. Deploy [Traefik](https://traefik.io/) dashboard:

   ```sh
   kubectl apply -f traefik-dashboard.yaml
   ```

5. Fill `../helm-chart/mapx/values.yaml` with the values needed for MapX to work properly

6. Deploy MapX:

   ```sh
   helm install dev ./helm-chart/mapx/ -f ./helm-chart/mapx/values.yaml
   ```
