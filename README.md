# demo-log-monitor

Demo Logging và Monitoring deployment trên K8S

## Thực hành 1: Deploy EFK stack

- Deploy ứng dụng counter để sinh log trong file `deployment.yaml`

```bash
kubectl -n <ns> apply -f deployment.yaml
```

- Kiểm tra log của pod:

```bash
kubectl -n <ns> get pod
kubectl -n <ns> logs -f counter-xxx 
```

- Elasticsearch đã được cài đặt trên VM tại địa chỉ: `<ip>:9200`

- Chỉnh sửa file `fluentbit-values.yaml` và sửa lại config backend với thông tin elasticsearch ở trên. Default config tham khảo tại [https://github.com/helm/charts/blob/master/stable/fluent-bit/values.yaml](https://github.com/helm/charts/blob/master/stable/fluent-bit/values.yaml)

- Deploy fluentbit helm chart:

```bash
helm upgrade --install -n <ns> -f fluentbit-values.yaml fluentbit stable/fluent-bit
```

- Kiểm tra fluent bit đã run hay chưa:

```bash
kubectl -n <ns> logs -f fluentbit-xxx
```

- Sửa file `kibana-values.yaml` và sửa lại giá trị nodeport tương ứng tránh trùng nhau

- Deploy kibana helm chart:

```bash
helm repo add elastic https://helm.elastic.co
helm repo update

helm upgrade --install -n <ns> -f kibana-values.yaml kibana elastic/kibana
```

- Truy cập `http://<host-ip>:<nodeport>` để access kibana

- Mở menu `Stack Management > Index Management` để check xem index đã xuất hiện chưa

- Mở menu `Stack Management > Index Patterns` và thêm 1 index pattern với prefix chính là prefix trong config của fluentbit:

```yaml
logstash_prefix: k8s_log
```

~> Tạo pattern là `k8s_log-*`

- Mở menu `Discovery` và bắt đầu search log

## Thực hành 2: Deploy prometheus-operator

- Cài đặt prometheus-operator:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install -n <ns> prom-operator prometheus-community/kube-prometheus-stack
```

- Verify bằng cách get pod đang chạy và service prometheus, grafana 
- Debug prometheus:

```bash
kubectl -n <ns> port-forward svc/prom-operator-kube-prometh-prometheus 9090:9090
```

- Forward grafana để test nhanh:

```bash
kubectl -n <ns> port-forward svc/prom-operator-grafana 8080:80
```

- Đăng nhập grafana: `admin/prop-operator`