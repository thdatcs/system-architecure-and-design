# system-architecture-and-design
Design a large-scale system

![system-architecture](https://user-images.githubusercontent.com/6086297/72776741-cbedf100-3c45-11ea-8890-2b6cc1927722.png)

### Components
- **gateway-service:**
  - Filter requests
    - Authenticate & authorize requests
    - Validate requests
  - Route requests to the cluster that can execute them most efficiently 
- **core-service:** do business logic
- **repository-service:**
  - Manage access to data
  - Partition data

### Tools
- **Traefik:** Proxy
- **Consul:** Service discovery
- **Loki + Grafana:** Logging
- **Prometheus + Grafana:** Monitoring
- **Jaeger:** Tracing
- **Redis:** Cache
- **Kafka:** Queue
- **ProxySQL:** MySQL proxy - split read & write operation to database
- **MySQL:** Database
