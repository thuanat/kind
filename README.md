
# 1. Chuẩn bị môi trường Lab
Trước khi bắt đầu, hãy đảm bảo máy bạn đã cài:

Docker

Kind

kubectl

Helm

## Khởi tạo Cluster Kind

Tạo một tệp kind-config.yaml để map port từ Cluster ra máy local:
...
Sau đó chạy lệnh: kind create cluster --config kind-config.yaml

##Bước 1: Thêm Repo Helm

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

##Bước 2: Cài đặt Loki (Logs)

helm upgrade --install loki grafana/loki \
  --set loki.auth_enabled=false \
  --set deploymentMode=SingleBinary \
  --set loki.commonConfig.replication_factor=1

  ##Bước 3: Cài đặt Tempo (Traces)

  helm upgrade --install tempo grafana/tempo \
  --set tempo.searchEnabled=true \
  --set tempo.reportingEnabled=true

  ##Bước 4: Cài đặt Mimir (Metrics)

  helm upgrade --install mimir grafana/mimir-distributed \
  --set mimir.base_config.auth_enabled=false

  ##Bước 5: Cài đặt Grafana (Visualization)

  helm upgrade --install grafana grafana/grafana \
  --set service.type=NodePort \
  --set service.nodePort=30000

  # Fix CoreDNS
  # Lệnh này ép CoreDNS sử dụng DNS Google thay vì bị loop bởi DNS của máy Host
kubectl -n kube-system patch configmap coredns --type merge -p '{"data":{"Corefile":".:53 {\n    errors\n    health\n    ready\n    kubernetes cluster.local in-addr.arpa ip6.arpa {\n       pods insecure\n       fallthrough in-addr.arpa ip6.arpa\n       ttl 30\n    }\n    prometheus :9153\n    forward . 8.8.8.8\n    cache 30\n    loop\n    reload\n    loadbalance\n}"}}'

# Khởi động lại CoreDNS để nhận cấu hình mới
kubectl rollout restart deployment coredns -n kube-system