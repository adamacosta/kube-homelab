# Install `logging` stack

Add the Elastic Helm repo:

```sh
helm repo add elastic https://helm.elastic.co
helm repo update
```

Install `eck-operator`:

```sh
helm upgrade \
  elastic-operator elastic/eck-operator \
  --install \
  --namespace elastic-system \
  --create-namespace \
  --version 2.6.1
```

Install `Elasticsearch` and `Kibana`:

```sh
kubectl create ns logging
```

```sh
kubectl apply -f -<<EOF
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch
  namespace: logging
  annotations:
    eck.k8s.elastic.co/license: basic
spec:
  version: 8.7.0
  nodeSets:
    - config:
        node.store.allow_mmap: false
      count: 3
      name: default
      podTemplate:
        spec:
          containers:
          - name: elasticsearch
            resources:
              limits:
                memory: 1Gi
              requests:
                cpu: 1
          securityContext:
            runAsUser: 1000
            fsGroup: 1000
---
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
  namespace: logging
  annotations:
    eck.k8s.elastic.co/license: basic
spec:
  version: 8.7.0
  config:
    server.publicBaseUrl: https://kibana.localdomain
  count: 1
  elasticsearchRef:
    name: elasticsearch
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  podTemplate:
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
EOF
```

Create an `Ingress` for `Kibana`:

```sh
kubectl apply -f -<<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: rke2-homelab-ca-issuer
  name: kibana
  namespace: logging
spec:
  rules:
  - host: kibana.localdomain
    http:
      paths:
      - backend:
          service:
            name: kibana-kb-http
            port:
              number: 5601
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - kibana.localdomain
    secretName: kibana.local-tls
EOF
```

Create values file for `fluent-bit`:

```sh
cat <<"EOF" > fluent-bit-values.yaml
env:
  - name: FLUENT_ELASTICSEARCH_PASSWORD
    valueFrom:
      secretKeyRef:
        name: elasticsearch-es-elastic-user
        key: elastic

extraVolumes:
  - secret:
      secretName: elasticsearch-es-http-certs-public
    name: elasticsearch-certs

extraVolumeMounts:
  - mountPath: /etc/elasticsearch/certs/
    name: elasticsearch-certs

config:
  outputs: |
    [OUTPUT]
        Name                     es
        Match                    kube.*
        Host                     elasticsearch-es-http
        HTTP_User                elastic
        HTTP_Passwd              ${FLUENT_ELASTICSEARCH_PASSWORD}
        Logstash_Format          On
        Suppress_Type_Name       On
        Retry_Limit              False
        Replace_Dots             On
        tls                      On
        tls.verify               On
        tls.ca_file              /etc/elasticsearch/certs/ca.crt
        storage.total_limit_size 10G
    [OUTPUT]
        Name                     es
        Match                    host.*
        Host                     elasticsearch-es-http
        HTTP_User                elastic
        HTTP_Passwd              ${FLUENT_ELASTICSEARCH_PASSWORD}
        Logstash_Format          On
        Suppress_Type_Name       On
        Logstash_Prefix          node
        Retry_Limit              False
        tls                      On
        tls.verify               On
        tls.ca_file              /etc/elasticsearch/certs/ca.crt
        storage.total_limit_size 10G

tolerations:
  - effect: NoExecute
    key: CriticalAddonsOnly
    operator: Exists
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
EOF
```

Add the `fluent-bit` Helm repo:

```sh
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
```

Install `fluentbit`:

```sh
helm upgrade \
  fluent-bit fluent/fluent-bit \
  --install \
  --namespace logging \
  --version 0.28.0 \
  --values fluent-bit-values.yaml
```

Grab the default user password for `Kibana`:

```sh
kubectl get secret -n logging elasticsearch-es-elastic-user -otemplate='{{.data.elastic | base64decode}}'
```

Log into [Kibana](https://kibana.localdomain) with the above password (username is "elastic") and view the log indices. Set up data view(s) according to [data views](https://www.elastic.co/guide/en/kibana/8.5/data-views.html).
