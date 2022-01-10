本地新建一个prometheus.yml文件

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    # 监控本地及端口
    - targets: ['localhost:9090']


docker pull prom/prometheus


docker run -p 9090:9090  --restart=always  --name  prometheus_emp  -v /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml   prom/prometheus