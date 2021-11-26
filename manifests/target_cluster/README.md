# Manage another Cluster

Steps adapted from [here](https://inlets.dev/blog/2021/06/02/argocd-private-clusters.html) and [here](https://argo-cd.readthedocs.io/en/stable/getting_started/#5-register-a-cluster-to-deploy-apps-to-optional)

To manage a remote cluster, ArgoCD needs to be able to authenticate. For this purpose, create a `Namespace`, `ServiceAccount`, permissive `Role` and `ClusterRoleBinding` on the TARGET cluster.

Find the right secret:

```shell
oc get sa -n nova-gitops-manager argocd-manager -o jsonpath='{.secrets}{"\n"}'
```

get the token data from it
```shell
oc get -n nova-gitops-manager secret/argocd-manager-token-vxj2p -o jsonpath='{.data.token}' | base64 --decode
oc get -n nova-gitops-manager secret/argocd-manager-token-vxj2p -o jsonpath='{.data.ca\.crt}{"\n"}'
```

Add the token to the source cluster:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cluster-api.xj.cars.lab
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: cluster-api.xj.cars.lab
  server: https://api.xj.cars.lab:6443
  config: |
    {
      "bearerToken": "<TOKEN>",
      "tlsClientConfig": {
        "serverName": "api.xj.cars.lab",
        "caData": "<CERTIFICATE>"
      }
    }
```

See the [ArgoCD Docs](https://argo-cd.readthedocs.io/en/release-1.8/operator-manual/declarative-setup/#clusters) for how to use other configurations.
