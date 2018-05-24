### Prometheus
- Toolkit for monitoring and alerting
- Pull based metrics gathering
- Simple text for metrics representation
- PromQL: powerful query language

Note:
Sayan: Prometheus was suffering from performance problems... Did the community take care of this in version2.0?
---

### Prometheus2 Features
- New Time Series Database (Can persist 1,000,000+ samples/core/sec to disk)
- Optimized Scraping
- Prometheus Rules in standard yaml, instead of proprietary DSL
+++
![Image](prometheus-intro/assets/Prom_Bench_1.png)
+++
![Image](prometheus-intro/assets/Prom_Bench_2.png)
+++
![Image](prometheus-intro/assets/Prom_Bench_3.png)
+++
![Image](prometheus-intro/assets/Prom_rule_format.png)

Note:
Sayan: That fine but what about autoscaling.
---
### Horizontal Pod Autoscaler
- v1 cpu based auto scaling only
- v2 autoscaling with custom Metrics (beta)
- Custom metrics adapter is required (https://github.com/DirectXMan12/k8s-prometheus-adapter)
+++
![Image](prometheus-intro/assets/Custom_metrics_1.png)
+++
![Image](prometheus-intro/assets/Custom_metrics.png)

CNCF Talk https://www.youtube.com/watch?v=1xm_ccAYO90&feature=youtu.be
---
