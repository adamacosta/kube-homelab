# Install `cert-manager`

```sh
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.0 \
  --set installCRDs=true
```

# Create a self-signed CA to issue certificates

```sh
kubectl apply -f -<<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: rke2-homelab-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: rke2-homelab-ca
  secretName: root-secret
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: rke2-homelab-ca-issuer
spec:
  ca:
    secretName: root-secret
EOF
```
