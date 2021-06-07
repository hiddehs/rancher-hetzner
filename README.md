# rancher-hetzner
k8s knowledge for rancher set-ups

## prep
On your running Rancher installation:
1. Add Hetzner cluster functionality for node templates and provisioning; https://{rancher_host}/n/drivers/node > Add [Rancher 2 Hetzner Cloud UI Driver](https://mxschmitt.github.io/ui-driver-hetzner/)
3. Create HZ project + network
4. Create templates for diffrent server types (don't forget to select network, disable private network if k8s API must be availble publicly (GitLab managed cluster for example))

## cluster creation

1. Check [csi-driver](https://github.com/hetznercloud/csi-driver) & [cloud-controller-manager](https://github.com/hetznercloud/hcloud-cloud-controller-manager) for latest versions
2. Add nodes with hz template
3. Kubernetes Options > Cloud Provider > External
4. Edit YAML (replace --xxxxx with API token for k8s cluster + network-id with id from the Project's network page)

```yaml
rancher_kubernetes_engine_config:
  ...
  addons: |-
    ---
    apiVersion: v1
    stringData:
      token: --xxxxx
      network: "NETWORK-ID"
    kind: Secret
    metadata:
      name: hcloud
      namespace: kube-system
    ---
    apiVersion: v1
    stringData:
      token: --zzzxxx
    kind: Secret
    metadata:
      name: hcloud-csi
      namespace: kube-system
  addons_include:
    - https://raw.githubusercontent.com/hetznercloud/csi-driver/v1.5.1/deploy/kubernetes/hcloud-csi.yml
    - https://raw.githubusercontent.com/hetznercloud/hcloud-cloud-controller-manager/v1.8.1/deploy/ccm-networks.yaml
```

5. Add `cluster_cidr` to assign `InternalIP`'s from init (afterwards sometimes requires refreshing all nodes to prevent routing issues)
   ```yaml
   rancher_kubernetes_engine_config:
   ...
     services:
     ...
       kube-controller:
          cluster_cidr: 10.244.0.0/16
   ```

## when ingressing
After ingress is pending/creating, patch the svc
(replace nbg1 with cluster dc loc)

```bash
kubectl patch svc ingress-nginx-ingress-controller -n gitlab-managed-apps -p '{"metadata":{"annotations":{"load-balancer.hetzner.cloud/location":"nbg1"}}}'
```

