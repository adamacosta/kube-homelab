# Install `Rancher`

```sh
export RANCHER_PASSWORD=$(cat /dev/urandom | LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1) &&
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.localdomain \
  --set bootstrapPassword=$RANCHER_PASSWORD \
  --version 2.7.1
```
