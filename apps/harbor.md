# Install `Harbor`

`Harbor` is also installed via `Helm`. First, add the repo:

```sh
helm repo add harbor https://helm.goharbor.io
helm repo update
```

Now install:

```sh
helm upgrade \
  harbor harbor/harbor \
  --install \
  --namespace harbor \
  --create-namespace \
  --version 1.11.0 \
  --set expose.tls.certSource=secret \
  --set expose.tls.secret.secretName="harbor.local-tls" \
  --set expose.ingress.hosts.core="harbor.localdomain" \
  --set expose.ingress.hosts.notary="notary.harbor.localdomain" \
  --set expose.ingress.annotations."cert-manager\.io/cluster-issuer"="rke2-homelab-ca-issuer" \
  --set persistence.persistentVolumeClaim.database.size=5Gi \
  --set persistence.persistentVolumeClaim.registry.size=50Gi
```

Grab the bootstrap admin password via:

```sh
kubectl get secret -n harbor harbor-core -otemplate='{{.data.HARBOR_ADMIN_PASSWORD | base64decode}}'
```

The username is `admin`.
