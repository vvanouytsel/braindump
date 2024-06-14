#containers

## Working with Registries

- Show images in registry

```bash
❯ curl localhost:30050/v2/_catalog

{"repositories":["bitnami/kubectl","bitnami/rabbitmq-cluster-operator","bitnami/rmq-messaging-topology-operator","external-secrets/external-secrets","fluent/fluent-bit","ingress-nginx/controller","ingress-nginx/kube-webhook-certgen","jimmidyson/configmap-reload","kube-state-metrics/kube-state-metrics","kubebuilder/kube-rbac-proxy","percona/percona-postgresql-operator","percona/pmm-client","prom/pushgateway","prometheus/alertmanager","prometheus/node-exporter","prometheus/prometheus","prometheus-adapter/prometheus-adapter","prometheuscommunity/postgres-exporter","tm-api-console","tm-assets","tm-compute","tm-config-hub","tm-config-operator","tm-context","tm-context-hub","tm-context-sql-sync","tm-custom-calculations-runtime","tm-dash-hub","tm-dashboarding","tm-datasource","tm-error-pages","tm-filters","tm-fingerprints","tm-hps","tm-keycloak","tm-keycloak-config","tm-migrate-hps-postgres","tm-ml-enterprise-gateway","tm-ml-enterprise-gateway-kernel-python-311","tm-ml-enterprise-gateway-kernel-python-viz","tm-ml-enterprise-gateway-kernelspecs","tm-ml-hub","tm-ml-jupyter-server","tm-notifications","tm-postgres","tm-postgres-configuration","tm-rabbitmq","tm-timeseries-builder","tm-translations","tm-trend-hub2","tm-work-organisation","tm-zementis","velero/velero","velero/velero-plugin-for-aws","velero/velero-plugin-for-microsoft-azure"]}
```

- Show specific tags of an image

```bash
# curl localhost:30050/v2/$IMAGE/tags/list
❯ curl localhost:30050/v2/bitnami/kubectl/tags/list

{"name":"bitnami/kubectl","tags":["1.28"]}
```