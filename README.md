# Docker Setup for Prometheus, Grafana, Node Exporter, Blackbox Exporter, and cAdvisor

This project sets up a monitoring and visualization stack using Docker, Prometheus, and Grafana. It uses several exporters to collect metrics from the system and Docker containers, which are then visualized in Grafana.

## Docker Compose File (`docker-compose.yml`)

This setup consists of multiple services defined in a Docker Compose file. Here's the breakdown of the services and their purposes:

### Services

1. **Prometheus**

   Prometheus is a monitoring and alerting toolkit. It scrapes and stores metrics in a time-series database.

   - **Image**: `prom/prometheus:latest`
   - **Container Name**: `prometheus`
   - **Ports**:
     - `9090:9090` – Exposes the Prometheus web UI to access metrics and configure scraping.
   - **Volumes**:
     - `./prometheus.yml:/etc/prometheus/prometheus.yml` – Mounts your custom Prometheus configuration file to the container.
   - **Command**:
     - `--config.file=/etc/prometheus/prometheus.yml` – Specifies the configuration file for Prometheus.

2. **Grafana**

   Grafana is an open-source platform for monitoring and observability, providing powerful dashboards and visualizations for data collected by Prometheus.

   - **Image**: `grafana/grafana:latest`
   - **Container Name**: `grafana`
   - **Ports**:
     - `3000:3000` – Exposes the Grafana web UI, typically accessible at `http://localhost:3000`.
   - **Volumes**:
     - `grafana-data:/var/lib/grafana` – Persists Grafana data (such as dashboards and settings) in a Docker volume.
   - **Environment**:
     - `GF_SECURITY_ADMIN_USER=admin` – Sets the Grafana admin username.
     - `GF_SECURITY_ADMIN_PASSWORD=admin` – Sets the Grafana admin password.

3. **Node Exporter**

   The Node Exporter exposes hardware and OS metrics (like CPU, memory, and disk usage) to Prometheus from the host system.

   - **Image**: `prom/node-exporter:latest`
   - **Container Name**: `node_exporter`
   - **Ports**:
     - `9100:9100` – Exposes Node Exporter's metrics at `http://localhost:9100/metrics`.
   - **Volumes**:
     - `/proc:/host/proc:ro` – Mounts the host `/proc` directory for CPU, memory, and other system stats.
     - `/sys:/host/sys:ro` – Mounts the host `/sys` directory for kernel parameters and hardware info.
     - `/:/rootfs:ro` – Mounts the root file system to provide additional system metrics.

4. **Blackbox Exporter**

   Blackbox Exporter allows Prometheus to perform endpoint monitoring, like checking the availability of websites or services via HTTP, HTTPS, TCP, ICMP, etc.

   - **Image**: `prom/blackbox-exporter:latest`
   - **Container Name**: `blackbox_exporter`
   - **Ports**:
     - `9115:9115` – Exposes the Blackbox Exporter's metrics at `http://localhost:9115/metrics`.

5. **cAdvisor**

   cAdvisor (Container Advisor) provides container-specific metrics, such as CPU, memory, and disk usage, for each running container.

   - **Image**: `gcr.io/cadvisor/cadvisor:latest`
   - **Container Name**: `cadvisor`
   - **Ports**:
     - `8080:8080` – Exposes cAdvisor's web UI and metrics at `http://localhost:8080`.
   - **Volumes**:
     - `/var/run/docker.sock:/var/run/docker.sock:ro` – Provides access to the Docker socket to gather container data.
     - `/sys:/sys:ro` – Mounts system files to gather system-level metrics.
     - `/var/lib/docker/:/var/lib/docker:ro` – Mounts Docker's directory for container-level stats.

### Volumes

- **grafana-data** – Persists Grafana's data, including dashboards and settings, in Docker volumes. This ensures data is retained even if the container is recreated.

---

## Prometheus Configuration (`prometheus.yml`)

In your `prometheus.yml` file, you define which targets Prometheus should scrape for metrics. It includes configuration for scraping metrics from the Node Exporter, Blackbox Exporter, and cAdvisor. Here's an example configuration:

```yaml
global:
  scrape_interval: 15s  # Default scrape interval

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']

  - job_name: 'blackbox_exporter'
    static_configs:
      - targets: ['blackbox_exporter:9115']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']



# Setting up Grafana Dashboards
## Once your Prometheus, Node Exporter, Blackbox Exporter, and cAdvisor containers are up and running, you can add them to Grafana for visualization. Here’s how to do it:

1. Access Grafana
Open your browser and go to http://localhost:3000 (or the appropriate IP if not on localhost).
Login using the credentials:
Username: admin
Password: admin

2. Add Prometheus as a Data Source
In the Grafana dashboard, click on the gear icon (⚙️) on the left-hand menu and choose "Data Sources."
Click "Add data source" and select Prometheus.

Set the following:
Name: Prometheus (default)
URL: http://<server ip>:9090 (this should match the Prometheus service's URL in your Docker Compose setup).
Click "Save & Test" to verify the connection.

3. Import Dashboards
Grafana provides pre-configured dashboards that you can import for Node Exporter, cAdvisor, etc.

### Import Node Exporter Full Dashboard
- In Grafana, click the "+" icon on the left panel and choose "Import."
- Enter the dashboard ID: 1860 (for Node Exporter Full Dashboard).Click "Load." (https://grafana.com/grafana/dashboards/1860-node-exporter-full/)
- Select your Prometheus data source and click "Import."

### Import cAdvisor Dashboard
- In Grafana, click the "+" icon and choose "Import."
- Enter the dashboard ID: 1694 (for cAdvisor Dashboard).Click "Load." (https://grafana.com/grafana/dashboards/14282-cadvisor-exporter/)
- Select your Prometheus data source and click "Import."
- Import Blackbox Exporter Dashboard (Optional)

### To monitor external endpoints, you can also import a Blackbox Exporter dashboard:

- In Grafana, click the "+" icon and choose "Import."
- Enter the dashboard ID: 1516 (for Blackbox Exporter Dashboard).Click "Load." (https://grafana.com/grafana/dashboards/7587-prometheus-blackbox-exporter/)
- Select your Prometheus data source and click "Import."
```
