#containers

## Working with Registries

- Show images in registry

```bash
❯ curl localhost:30050/v2/_catalog

{"repositories":["bitnami/kubectl","bitnami/rabbitmq-cluster-operator","bitnami/rmq-messaging-topology-operator","external-secrets/external-secrets","fluent/fluent-bit","ingress-nginx/controller","ingress-nginx/kube-webhook-certgen","jimmidyson/configmap-reload","kube-state-metrics/kube-state-metrics","kubebuilder/kube-rbac-proxy","percona/percona-postgresql-operator","percona/pmm-client","prom/pushgateway","prometheus/alertmanager","prometheus/node-exporter","prometheus/prometheus","prometheus-adapter/prometheus-adapter","prometheuscommunity/postgres-exporter","my-api-console","my-assets","my-compute","my-config-hub","my-config-operator","my-context","my-context-hub","my-context-sql-sync","my-custom-calculations-runtime","my-dash-hub","my-dashboarding","my-datasource","my-error-pages","my-filters","my-fingerprints","my-hps","my-keycloak","my-keycloak-config","my-migrate-hps-postgres","my-ml-enterprise-gateway","my-ml-enterprise-gateway-kernel-python-311","my-ml-enterprise-gateway-kernel-python-viz","my-ml-enterprise-gateway-kernelspecs","my-ml-hub","my-ml-jupyter-server","my-notifications","my-postgres","my-postgres-configuration","my-rabbitmq","my-timeseries-builder","my-translations","my-trend-hub2","my-work-organisation","my-zementis","velero/velero","velero/velero-plugin-for-aws","velero/velero-plugin-for-microsoft-azure"]}
```

- Show specific tags of an image

```bash
# curl localhost:30050/v2/$IMAGE/tags/list
❯ curl localhost:30050/v2/bitnami/kubectl/tags/list

{"name":"bitnami/kubectl","tags":["1.28"]}
```
