```sh
helm upgrade gitlab gitlab/gitlab \
  --install \
  --namespace gitlab \
  --create-namespace \
  --version 6.10.3 \
  --set global.hosts.domain=localdomain \
  --set global.hosts.gitlab.name=gitlab.localdomain \
  --set global.hosts.minio.name=gitlab-minio.localdomain \
  --set global.hosts.registry.name=gitlab-registry.localdomain \
  --set global.ingress.configureCertmanager=false \
  --set global.ingress.annotations."cert-manager\.io/cluster-issuer"="rke2-tools-ca-issuer" \
  --set global.kas.enabled=false \
  --set gitlab.webservice.ingress.tls.secretName=gitlab.local-tls \
  --set minio.ingress.tls.secretName=gitlab-minio.local-tls \
  --set registry.ingress.tls.secretName=gitlab-registry.local-tls \
  --set certmanager.install=false \
  --set gitlab-runner.install=false
```