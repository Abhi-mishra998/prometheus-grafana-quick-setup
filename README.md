# 🚀 Complete DevOps Setup Guide
## Docker, Kubernetes, Prometheus & Grafana Installation

**Author:** Abhishek Mishra  
**Last Updated:** November 2025

---

## 📑 Table of Contents
- [Docker Installation](#-docker-installation)
- [Docker Compose Setup](#-docker-compose-setup)
- [Kubernetes & Minikube](#-kubernetes--minikube)
- [Helm Package Manager](#-helm-package-manager)
- [Prometheus Monitoring](#-prometheus-monitoring)
- [Node Exporter Setup](#-node-exporter-setup)
- [Grafana Dashboard](#-grafana-dashboard)
- [Troubleshooting](#-troubleshooting)
- [Useful Commands Reference](#-useful-commands-reference)

---

## 🐳 Docker Installation

### Method 1: Quick Install (Ubuntu/Debian)
```bash
# Install Docker from default repository
sudo apt install docker.io -y

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Method 2: Official Docker Repository (Recommended)
```bash
# Update package index and install prerequisites
sudo apt update && sudo apt install sudo -y
sudo apt-get update
sudo apt-get install ca-certificates curl -y

# Create directory for Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's official GPG key (ensures package authenticity)
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to system sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and related components
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add current user to docker group (allows running Docker without sudo)
sudo usermod -aG docker $USER

# IMPORTANT: Logout and login again for group changes to take effect
# Or run: newgrp docker
```

### Verify Docker Installation
```bash
# Check Docker service status
sudo systemctl status docker

# Start Docker if not running
sudo systemctl start docker

# Test Docker with hello-world container
sudo docker run hello-world

# List all containers (running and stopped)
docker ps -a

# Check Docker version
docker --version

# View system-wide Docker information
docker info
```

### Essential Docker Commands
```bash
# IMAGE MANAGEMENT
docker images                    # List all downloaded images
docker pull <image>              # Download an image from Docker Hub
docker rmi <image>               # Remove an image
docker rmi $(docker images -q)   # Remove all images

# CONTAINER MANAGEMENT
docker ps                        # List running containers
docker ps -a                     # List all containers
docker run -d <image>            # Run container in detached mode
docker run -it <image> /bin/bash # Run container interactively
docker stop <container-id>       # Stop a running container
docker start <container-id>      # Start a stopped container
docker restart <container-id>    # Restart a container
docker rm <container-id>         # Remove a container
docker rm $(docker ps -aq)       # Remove all stopped containers

# CONTAINER LOGS & DEBUGGING
docker logs <container-id>       # View container logs
docker logs -f <container-id>    # Follow container logs in real-time
docker exec -it <container-id> /bin/bash  # Access container shell
docker inspect <container-id>    # View detailed container information
docker stats                     # View resource usage of all containers

# CLEANUP COMMANDS
docker system prune              # Remove unused data
docker system prune -a           # Remove all unused images
docker volume prune              # Remove unused volumes
docker network prune             # Remove unused networks
```

---

## 📦 Docker Compose Setup

Docker Compose allows you to define and run multi-container applications using a YAML file.

### Installation Method 1: Plugin (Recommended)
```bash
# Update package list
sudo apt update

# Install Docker Compose plugin
sudo apt install docker-compose-plugin -y

# Verify installation
docker compose version
```

### Installation Method 2: Manual Binary Download
```bash
# Download latest Docker Compose binary
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### Test Docker Compose
```bash
# Create a test directory
mkdir test-compose && cd test-compose

# Create a sample docker-compose.yml file
cat > docker-compose.yml <<EOF
services:
  hello:
    image: hello-world
EOF

# Run the compose file
docker compose up

# Stop and remove containers (including volumes)
docker compose down -v
```

### Essential Docker Compose Commands
```bash
docker compose up                # Start all services
docker compose up -d             # Start services in detached mode
docker compose down              # Stop and remove containers
docker compose down -v           # Stop, remove containers and volumes
docker compose ps                # List running services
docker compose logs              # View output from all services
docker compose logs -f           # Follow logs in real-time
docker compose exec <service> bash  # Execute command in running service
docker compose restart           # Restart all services
docker compose pull              # Pull latest images for all services
docker compose build             # Build or rebuild services
```

---

## ☸️ Kubernetes & Minikube

Kubernetes is a container orchestration platform. Minikube runs a single-node cluster locally for testing.

### Install Minikube
```bash
# Download Minikube binary
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64

# Install Minikube to system path
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# Start Minikube cluster (creates a local Kubernetes cluster)
minikube start

# Check cluster status
minikube status

# Access Kubernetes dashboard
minikube dashboard
```

### Install kubectl (Kubernetes CLI)
```bash
# Download the latest stable kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install kubectl to system path with proper permissions
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client

# Expected output shows client version and kustomize version
# Client Version: v1.31.0
# Kustomize Version: v5.4.1
```

### Essential kubectl Commands
```bash
# CLUSTER INFORMATION
kubectl cluster-info             # Display cluster info
kubectl get nodes                # List all nodes in cluster
kubectl get nodes -o wide        # Detailed node information

# NAMESPACE MANAGEMENT
kubectl get namespaces           # List all namespaces
kubectl create namespace <name>  # Create new namespace
kubectl delete namespace <name>  # Delete namespace
kubectl config set-context --current --namespace=<name>  # Switch namespace
kubectl config view --minify | grep namespace:  # View current namespace

# POD MANAGEMENT
kubectl get pods                 # List pods in current namespace
kubectl get pods -A              # List pods in all namespaces
kubectl get pods -o wide         # Show pods with node information
kubectl describe pod <pod-name>  # Detailed pod information
kubectl logs <pod-name>          # View pod logs
kubectl logs -f <pod-name>       # Follow pod logs in real-time
kubectl exec -it <pod-name> -- /bin/bash  # Access pod shell
kubectl delete pod <pod-name>    # Delete a pod

# DEPLOYMENT MANAGEMENT
kubectl get deployments          # List all deployments
kubectl create deployment <name> --image=<image>  # Create deployment
kubectl scale deployment <name> --replicas=3      # Scale deployment
kubectl rollout status deployment/<name>          # Check rollout status
kubectl rollout history deployment/<name>         # View rollout history
kubectl delete deployment <name>                  # Delete deployment

# SERVICE MANAGEMENT
kubectl get services             # List all services
kubectl get svc                  # Shorthand for services
kubectl expose deployment <name> --port=80 --type=NodePort  # Expose deployment
kubectl describe service <name>  # Service details
kubectl delete service <name>    # Delete service

# DEBUGGING & TROUBLESHOOTING
kubectl get events               # View cluster events
kubectl get events --sort-by='.lastTimestamp'  # Sort events by time
kubectl top nodes                # View node resource usage
kubectl top pods                 # View pod resource usage
kubectl port-forward <pod-name> 8080:80  # Forward local port to pod

# CONFIGURATION
kubectl apply -f <file.yaml>     # Apply configuration from file
kubectl delete -f <file.yaml>    # Delete resources from file
kubectl get all                  # List all resources in namespace
kubectl get all -A               # List all resources in all namespaces
```

---

## ⎈ Helm Package Manager

Helm is the package manager for Kubernetes, making it easy to deploy complex applications.

### Install Helm
```bash
# Download and install Helm using official script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify installation
helm version
```

### Setup Monitoring with Helm

#### Create Monitoring Namespace
```bash
# Create dedicated namespace for monitoring tools
kubectl create namespace monitoring

# Set monitoring as default namespace for current context
kubectl config set-context --current --namespace=monitoring

# Verify current namespace
kubectl config view --minify | grep namespace:
```

#### Install Prometheus
```bash
# Add Prometheus community Helm chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update Helm repositories
helm repo update

# Install Prometheus (collects and stores metrics)
helm install prometheus prometheus-community/prometheus -n monitoring

# Check installation status
helm status prometheus -n monitoring
```

#### Install Grafana
```bash
# Add Grafana Helm chart repository
helm repo add grafana https://grafana.github.io/helm-charts

# Update repositories
helm repo update

# Install Grafana (visualization dashboard)
helm install my-grafana grafana/grafana --namespace monitoring --create-namespace

# Verify pods are running
kubectl get pods -n monitoring

# Get Grafana admin password
kubectl get secret --namespace monitoring my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Access Grafana Dashboard
```bash
# Method 1: Port Forward (for local access)
kubectl port-forward --address 0.0.0.0 -n monitoring svc/my-grafana 3000:80

# Method 2: NodePort (for external access)
kubectl patch svc my-grafana -n monitoring -p '{"spec": {"type": "NodePort"}}'
kubectl get svc my-grafana -n monitoring  # Note the NodePort

# Access Grafana at: http://<node-ip>:<nodeport>
# Or with port-forward: http://localhost:3000
# Default credentials: admin / <password-from-secret>
```

### Essential Helm Commands
```bash
# REPOSITORY MANAGEMENT
helm repo list                   # List added repositories
helm repo add <name> <url>       # Add a repository
helm repo update                 # Update all repositories
helm repo remove <name>          # Remove a repository
helm search repo <keyword>       # Search for charts

# INSTALLATION & MANAGEMENT
helm install <release> <chart>   # Install a chart
helm list                        # List all releases
helm list -A                     # List releases in all namespaces
helm status <release>            # Show release status
helm upgrade <release> <chart>   # Upgrade a release
helm rollback <release> <revision>  # Rollback to previous version
helm uninstall <release>         # Uninstall a release

# TROUBLESHOOTING
helm get values <release>        # Show values used in release
helm get manifest <release>      # Show generated manifests
helm history <release>           # Show release history
```

### Cleanup (Optional)
```bash
# Uninstall Prometheus
helm uninstall prometheus -n monitoring

# Uninstall Grafana
helm uninstall my-grafana -n monitoring

# Delete monitoring namespace
kubectl delete namespace monitoring
```

---

## 📊 Prometheus Monitoring

Prometheus is an open-source monitoring system that collects metrics from configured targets.

### Install Prometheus (Binary Method)
```bash
# Navigate to installation directory
cd /opt

# Create Prometheus user (no login, no home directory)
sudo useradd --no-create-home --shell /bin/false prometheus

# Create required directories
sudo mkdir /etc/prometheus /var/lib/prometheus

# Download Prometheus latest release
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz

# Extract the archive
sudo tar xvf prometheus-2.54.1.linux-amd64.tar.gz
cd prometheus-2.54.1.linux-amd64

# Move binaries to system path
sudo mv prometheus promtool /usr/local/bin/

# Move console files
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

### Configure Prometheus
```bash
# Create Prometheus configuration file
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
  scrape_interval: 15s          # How often to scrape targets
  evaluation_interval: 15s      # How often to evaluate rules

# Alerting configuration (optional)
alerting:
  alertmanagers:
    - static_configs:
        - targets: []

# Scrape configurations (what to monitor)
scrape_configs:
  # Monitor Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Monitor Node Exporter (system metrics)
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
EOF
```

### Set Permissions
```bash
# Set ownership for Prometheus user
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

### Create Systemd Service
```bash
# Create service file for automatic startup
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus Monitoring System
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \\
  --config.file=/etc/prometheus/prometheus.yml \\
  --storage.tsdb.path=/var/lib/prometheus/ \\
  --web.listen-address=:9090 \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF
```

### Start Prometheus
```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Start Prometheus service
sudo systemctl start prometheus

# Enable Prometheus to start on boot
sudo systemctl enable prometheus

# Check service status
sudo systemctl status prometheus

# View logs if needed
sudo journalctl -u prometheus -f
```

### Access Prometheus Web UI
```
Open browser: http://<your-server-ip>:9090

Key sections:
- Status → Targets: View all monitored targets
- Graph: Query and visualize metrics
- Alerts: View active alerts
- Configuration: View current config
```

---

## 📡 Node Exporter Setup

Node Exporter exposes system metrics (CPU, memory, disk, network) for Prometheus to scrape.

### Install Node Exporter
```bash
# Navigate to installation directory
cd /opt

# Download Node Exporter
sudo curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

# Extract archive
sudo tar xvf node_exporter-1.8.2.linux-amd64.tar.gz

# Move binary to system path
sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/

# Clean up
sudo rm -rf node_exporter-1.8.2.linux-amd64*

# Install gzip if not present (for compressed metrics)
sudo apt install gzip -y
```

### Create Node Exporter Service
```bash
# Create systemd service file
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Documentation=https://github.com/prometheus/node_exporter
After=network.target

[Service]
User=nobody
Group=nogroup
Type=simple
ExecStart=/usr/local/bin/node_exporter

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=default.target
EOF
```

### Start Node Exporter
```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Start Node Exporter
sudo systemctl start node_exporter

# Enable on boot
sudo systemctl enable node_exporter

# Check status
sudo systemctl status node_exporter

# Test metrics endpoint
curl http://localhost:9100/metrics
```

### Verify in Prometheus
```
1. Open Prometheus: http://<server-ip>:9090
2. Go to Status → Targets
3. You should see 'node_exporter' with state 'UP'
4. Try query: node_cpu_seconds_total
```

---

## 📈 Grafana Dashboard

Grafana provides beautiful dashboards for visualizing Prometheus metrics.

### Configure Grafana Data Source
```
1. Access Grafana: http://<server-ip>:3000
2. Login with admin credentials
3. Go to: Connections → Data Sources → Add data source
4. Select: Prometheus
5. URL: http://localhost:9090
6. Click: Save & Test
```

### Import Node Exporter Dashboard
```
1. In Grafana, click: + → Import
2. Dashboard ID: 1860 (Official Node Exporter Full)
3. Select your Prometheus data source
4. Click: Import

Alternative Dashboards:
- Dashboard ID 405: Node Exporter Server Metrics
- Dashboard ID 11074: Node Exporter for Prometheus
```

### Create Custom Dashboard
```
1. Click: + → Dashboard
2. Add Panel
3. Select visualization type
4. Write PromQL query, examples:
   - CPU Usage: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   - Memory Usage: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
   - Disk Usage: (node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
5. Customize and save
```

---

## 🔧 Troubleshooting

### Check Pod Status
```bash
# View all pods with detailed status
kubectl get pods -n monitoring -o wide

# Describe pod for events and issues
kubectl describe pod <pod-name> -n monitoring

# View pod logs
kubectl logs <pod-name> -n monitoring

# Follow logs in real-time
kubectl logs -f <pod-name> -n monitoring

# Check previous container logs (if pod crashed)
kubectl logs <pod-name> -n monitoring --previous
```

### Check Services
```bash
# List all services in monitoring namespace
kubectl get svc -n monitoring

# Describe service for endpoint details
kubectl describe svc <service-name> -n monitoring

# Check service endpoints
kubectl get endpoints -n monitoring
```

### Helm Troubleshooting
```bash
# List all Helm releases
helm list -n monitoring

# Check release status
helm status prometheus -n monitoring
helm status my-grafana -n monitoring

# View release values
helm get values prometheus -n monitoring

# View generated manifests
helm get manifest prometheus -n monitoring

# Rollback if needed
helm rollback <release-name> <revision> -n monitoring
```

### Docker Issues
```bash
# Check Docker daemon status
sudo systemctl status docker

# Restart Docker
sudo systemctl restart docker

# View Docker logs
sudo journalctl -u docker -f

# Check disk space (Docker needs space)
df -h

# Clean up unused Docker resources
docker system prune -a
```

### Prometheus Issues
```bash
# Check Prometheus status
sudo systemctl status prometheus

# View Prometheus logs
sudo journalctl -u prometheus -f

# Test Prometheus config
/usr/local/bin/promtool check config /etc/prometheus/prometheus.yml

# Restart Prometheus
sudo systemctl restart prometheus
```

### Node Exporter Issues
```bash
# Check service status
sudo systemctl status node_exporter

# Test metrics endpoint
curl http://localhost:9100/metrics

# Restart service
sudo systemctl restart node_exporter

# View logs
sudo journalctl -u node_exporter -f
```

---

## 📚 Useful Commands Reference

### System Information
```bash
# Check OS version
cat /etc/os-release
lsb_release -a

# Check system resources
free -h                 # Memory usage
df -h                   # Disk usage
top                     # Process monitor
htop                    # Better process monitor (install: sudo apt install htop)
uptime                  # System uptime and load

# Network information
ip addr show            # IP addresses
netstat -tulpn          # Open ports
ss -tulpn              # Socket statistics (modern netstat)
```

### Port Management
```bash
# Check if port is in use
sudo lsof -i :<port>
sudo netstat -tulpn | grep :<port>

# Kill process on port
sudo kill -9 $(sudo lsof -t -i:<port>)

# Allow port through firewall (if UFW enabled)
sudo ufw allow <port>
```

### Service Management
```bash
# List all services
systemctl list-units --type=service

# Enable service on boot
sudo systemctl enable <service>

# Disable service
sudo systemctl disable <service>

# Restart service
sudo systemctl restart <service>

# Reload service configuration
sudo systemctl reload <service>

# View service logs
sudo journalctl -u <service> -f
```

### File Operations
```bash
# Find files
find / -name "<filename>"

# Search in files
grep -r "search-term" /path/

# Check file permissions
ls -la

# Change permissions
chmod 755 <file>
chmod +x <file>

# Change ownership
chown user:group <file>
```

### Network Testing
```bash
# Test connectivity
ping <host>

# Test port connectivity
telnet <host> <port>
nc -zv <host> <port>

# DNS lookup
nslookup <domain>
dig <domain>

# Download files
wget <url>
curl -O <url>
```

---

## 🎯 Quick Setup Summary

### Minimum Setup for Monitoring Stack
```bash
# 1. Install Docker
sudo apt install docker.io -y

# 2. Install Minikube & kubectl
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start

# 3. Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 4. Install Prometheus & Grafana
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -n monitoring
helm install my-grafana grafana/grafana -n monitoring

# 5. Access Grafana
kubectl port-forward --address 0.0.0.0 -n monitoring svc/my-grafana 3000:80

# Get password
kubectl get secret --namespace monitoring my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

---

## 📝 Notes

- Always use `sudo` for system-level operations
- Logout and login after adding user to docker group
- Keep your system updated: `sudo apt update && sudo apt upgrade`
- Monitor disk space regularly when running containers
- Backup Prometheus data directory: `/var/lib/prometheus/`
- Default Grafana credentials: admin / (get from secret)
- Prometheus stores metrics for 15 days by default

---

## 🤝 Contributing

Feel free to contribute by:
- Reporting issues
- Suggesting improvements
- Adding new sections
- Fixing errors

---

## 📧 Contact

**Author:** Abhishek Mishra

For questions or suggestions, please create an issue or reach out!

---

**Happy DevOps Learning! 🚀**
