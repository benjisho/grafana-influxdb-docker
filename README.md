# grafana-influxdb-docker

# Grafana + InfluxDB Monitoring Stack

## Table of Contents
1. [Introduction](#introduction)
2. [Features](#features)
3. [Getting Started](#getting-started)
   - [Prerequisites](#prerequisites)
   - [Installation](#installation)
   - [InfluxDB Secret Files Setup](#influxdb-secret-files-setup)
4. [Generate Self-Signed SSL Certificate](#generate-ssl-self-signed-certificate-into-certs-directory)
5. [Updated Grafana Version](#updated-grafana-version)
6. [InfluxDB Configuration](#influxdb-configuration)
7. [Usage](#usage)
8. [Monitoring Container Logs](#monitoring-container-logs)
9. [Contributing](#contributing)
10. [License](#license)

## Introduction

This project provides a complete monitoring stack with Grafana and InfluxDB in a containerized environment. It combines Grafana, an open-source platform for monitoring and observability, with InfluxDB, a time-series database purpose-built to collect, store, process, and visualize metrics and events. This stack is designed to be easy to deploy and manage, making it ideal for environments that benefit from containerization.

The monitoring stack offers a scalable way to collect time-series data in InfluxDB and visualize it through beautiful Grafana dashboards, providing comprehensive insights into your system metrics, application performance, and business data.

## Features

- Complete monitoring stack with Grafana and InfluxDB
- Easy to deploy in a containerized environment
- InfluxDB v2 with secure credential management using Docker secrets
- Grafana with pre-configured health checks
- Nginx reverse proxy with SSL/TLS support
- Configurable settings for different monitoring needs
- Integration with various data sources
- Time-series data collection and visualization
- Scalable architecture suitable for production environments

## Getting Started

### Prerequisites

- Docker and Docker Compose installed on your system
- Basic understanding of containerization, Grafana, and InfluxDB
- Understanding of time-series databases and monitoring concepts

---

### Installation

1. Clone the repository:

```bash
git clone https://github.com/benjisho/grafana-influxdb-docker.git
```

2. Navigate to the cloned directory:

```bash
cd grafana-influxdb-docker
```

---

### InfluxDB Secret Files Setup

This setup follows the official InfluxDB Docker Compose guide using secrets for secure credential management. Before starting the containers, you need to create secret files for InfluxDB authentication.

1. Create the secrets directory:

```bash
mkdir -p secrets
```

2. Create the InfluxDB admin username file:

```bash
echo "admin" > secrets/influxdb2-admin-username
```

3. Create the InfluxDB admin password file (replace with your desired password):

```bash
echo "MySecurePassword123!" > secrets/influxdb2-admin-password
```

4. Create the InfluxDB admin token file (replace with your desired token):

```bash
echo "MySecureAdminToken123==" > secrets/influxdb2-admin-token
```

> **Security Note:** These files contain sensitive credentials. They are included in `.gitignore` to prevent accidental commits to version control. Ensure they are properly protected and consider using environment-specific passwords and tokens for production deployments.

---

## Generate SSL self-signed certificate into `certs/` directory

> **Note:** This section is relevant if you haven't generated a well-known SSL certificate from a certified authority.
> These will guide you on creating a self-signed certificate, useful for testing or development purposes.

1. Run the following command to generate a 2048-bit RSA private key, which is used to decrypt traffic:

```bash
openssl genrsa -out certs/server.key 2048
```

2. Run the following command to generate a certificate, using the private key from the previous step.

```bash
openssl req -new -key certs/server.key -out certs/server.csr
```

3. Run the following command to self-sign the certificate with the private key, for a period of validity of 365 days:

```bash
openssl x509 -req -days 365 -in certs/server.csr -signkey certs/server.key -out certs/server.crt
```

---

## Updated Grafana Version

The Grafana container now uses version `12.0.1-ubuntu`. This version is based on Ubuntu and provides enhanced compatibility and stability. Ensure that your environment is compatible with this version before deployment.

### Key Notes:
- The Ubuntu-based image may have different dependencies compared to the Alpine-based images. Ensure that any custom plugins or configurations are compatible.
- If you encounter issues, refer to the [Grafana Docker documentation](https://grafana.com/docs/grafana/latest/installation/docker/) for troubleshooting.

### Running the Setup
To start all services (Grafana, InfluxDB, and Nginx), use the following command:

```bash
docker-compose up -d
```

This will pull the specified versions and start all services with proper networking and dependencies.

---

## InfluxDB Configuration

This project uses InfluxDB v2 configured according to the official Docker Compose setup guide. The configuration includes:

### Database Setup
- **Image**: `influxdb:2` (latest InfluxDB v2)
- **Port**: `8086` (standard InfluxDB port)
- **Organization**: `docs` (default organization)
- **Bucket**: `home` (default bucket for data storage)

### Security Features
- **Docker Secrets**: Credentials are managed using Docker Compose secrets to prevent exposure in container inspection
- **Secret Files**: Username, password, and admin token are stored in separate files
- **Automatic Setup**: InfluxDB initializes automatically on first run with the provided credentials

### Data Persistence
- **Data Volume**: `influxdb2-data` mounted to `/var/lib/influxdb2` for database files
- **Config Volume**: `influxdb2-config` mounted to `/etc/influxdb2` for configuration files

### Environment Variables
The InfluxDB container uses the following environment variables for initialization:
- `DOCKER_INFLUXDB_INIT_MODE: setup` - Enables automatic setup mode
- `DOCKER_INFLUXDB_INIT_USERNAME_FILE` - Path to username secret file
- `DOCKER_INFLUXDB_INIT_PASSWORD_FILE` - Path to password secret file
- `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN_FILE` - Path to admin token secret file

### Accessing InfluxDB
- **Web UI**: http://localhost:8086
- **API**: http://localhost:8086/api/v2/
- **CLI**: Available through `docker exec` commands

### InfluxDB CLI Usage
To run InfluxDB CLI commands inside the container:

```bash
# Access the InfluxDB CLI
docker exec -it <container_name> influx

# Example: List buckets
docker exec -it <container_name> influx bucket list

# Example: Create a new bucket
docker exec -it <container_name> influx bucket create -n my-bucket -o docs
```

---

## Usage

To use the complete monitoring stack:

### Starting the Stack

1. Ensure you have created the InfluxDB secret files (see [InfluxDB Secret Files Setup](#influxdb-secret-files-setup))

2. Start all containers:

```bash
docker-compose up -d
```

### Accessing the Services

#### Grafana
- **URL**: https://localhost (via Nginx) or http://localhost:3000 (direct)
- **Default Login**: `admin` / `admin` (change on first login)
- **Purpose**: Data visualization and dashboard creation

#### InfluxDB
- **URL**: http://localhost:8086
- **Login**: Use the credentials from your secret files
  - Username: Contents of `secrets/influxdb2-admin-username`
  - Password: Contents of `secrets/influxdb2-admin-password`
- **Purpose**: Time-series data storage and management

### Connecting Grafana to InfluxDB

1. Access Grafana and log in
2. Navigate to `Configuration > Data Sources`
3. Click `Add data source` and select `InfluxDB`
4. Configure the data source:
   - **URL**: `http://influxdb2:8086`
   - **Organization**: `docs`
   - **Token**: Use the token from `secrets/influxdb2-admin-token`
   - **Default Bucket**: `home`
5. Click `Save & Test` to verify the connection

### Sample Data Ingestion

To test your setup, you can write sample data to InfluxDB:

```bash
# Using InfluxDB CLI inside the container
docker exec -it $(docker-compose ps -q influxdb2) influx write \
  --bucket home \
  --org docs \
  --precision s \
  'temperature,location=room1 value=23.5'
```

### Creating Dashboards

1. In Grafana, go to `Dashboards > New Dashboard`
2. Add a new panel
3. Select your InfluxDB data source
4. Write a Flux query to visualize your data:
   ```flux
   from(bucket: "home")
     |> range(start: -1h)
     |> filter(fn: (r) => r._measurement == "temperature")
   ```
5. Configure visualization options and save

For more detailed usage instructions, refer to:
- [Grafana Documentation](https://grafana.com/docs/)
- [InfluxDB Documentation](https://docs.influxdata.com/)

---

## Monitoring Container Logs

This section guides you on how to use `docker-compose logs -f` to follow the log output of all containers (Grafana, InfluxDB, and Nginx) in real-time.

### Command Explanation
- `docker-compose logs -f`: This command fetches the logs of the containers.
- The `-f` flag means "follow", allowing you to view the log output in real-time.

### Steps to Follow Logs
1. Navigate to where your `docker-compose.yml` file is located.
2. Execute the following command to follow the logs for all services:

   ```bash
   docker-compose logs -f
   ```
   
   This will display a continuous stream of logs of all containers in the `docker-compose.yml`.

3. If you want to follow the logs of a specific container from the `docker-compose.yml`, specify the service name.
   
   For Grafana logs:
   ```bash
   docker-compose logs -f grafana
   ```

   For InfluxDB logs:
   ```bash
   docker-compose logs -f influxdb2
   ```

   For Nginx logs:
   ```bash
   docker-compose logs -f nginx
   ```

### Tips for Log Outputs
- **Grafana Logs:** Look for indications of successful startup, connection to data sources (especially InfluxDB), and any error messages related to querying data.
- **InfluxDB Logs:** Monitor for successful initialization, database setup completion, authentication events, and query performance metrics.
- **Nginx Logs:** These logs will include information about HTTP requests, responses, and any errors related to network connections or SSL/TLS certificates.

### Common Log Patterns to Watch For

#### InfluxDB Initialization Success:
```
ts=2024-XX-XX level=info msg="Starting InfluxDB" 
ts=2024-XX-XX level=info msg="Setup completed successfully"
```

#### Grafana Data Source Connection:
```
logger=datasources t=2024-XX-XX level=info msg="Initializing datasources"
```

> **Note:** Keep in mind that the volume of logs can be high depending on the level of activity. It's advisable to filter or search through the logs for specific keywords related to the issues you're investigating.

---

## Contributing

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.