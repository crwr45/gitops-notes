# Manage another Cluster

Steps taken from [here](https://inlets.dev/blog/2021/06/02/argocd-private-clusters.html)


To manage a remote cluster, ArgoCD needs to be able to authenticate. For this purpose, create a `Namespace`, `ServiceAccount`, permissive `Role` and `ClusterRoleBinding` on the TARGET cluster.

Find the right secret:

```shell
oc get sa -n nova-gitops-manager nova-manager -o jsonpath='{.secrets}{"\n"}'
```

get the token data from it
```shell
oc get -n nova-gitops-manager secret/nova-manager-token-vxj2p -o jsonpath='{.data.token}' | base64 --decode
```

Add the token to the source cluster:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: xj-cars-lab
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: xj.cars.lab
  server: https://api.xj.cars.lab:6443
  config: |
    {
      "bearerToken": "<TOKEN>",
      "tlsClientConfig": {
        "serverName": "kubernetes.default.svc"
      }
    }

```

