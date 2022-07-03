# Prometheus 服务器

## 启动

docker run --rm -p 9090:9090 -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
