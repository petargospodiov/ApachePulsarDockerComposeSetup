# Apache Pulsar Docker Compose Setup

This repository provides a Docker Compose setup to run an Apache Pulsar cluster locally. The setup includes Zookeeper, Bookie, Broker, and Pulsar Manager services.

## Prerequisites

- Docker and Docker Compose installed on your system.
- Sufficient disk space and memory for running Pulsar services.

## Setup and Instructions

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd <repository-folder>
```

### Step 2: Start Services

Start the services in detached mode:

```bash
docker-compose up -d
```

### Step 3: Verify the Setup

- Confirm that all containers are running using the following command:

```bash
docker ps
```

### Step 4: Configure Pulsar Manager

1. Open a terminal in the `pulsar-manager` container:

   ```bash
   docker exec -it pulsar-manager /bin/bash
   ```

2. Retrieve the CSRF token required for secure communication with the Pulsar Manager API:

   ```bash
   CSRF_TOKEN=$(curl http://localhost:7750/pulsar-manager/csrf-token)
   ```

3. Create a superuser for Pulsar Manager:

   ```bash
   curl -H "X-XSRF-TOKEN: $CSRF_TOKEN" -H "Cookie: XSRF-TOKEN=$CSRF_TOKEN;" -H "Content-Type: application/json" -X PUT http://localhost:7750/pulsar-manager/users/superuser -d '{"name": "admin", "password": "apachepulsar", "description": "test", "email": "username@test.org"}'
   ```

4. Exit the container terminal:

   ```bash
   exit
   ```

5. Access the Pulsar Manager Admin UI at [http://localhost:9527/](http://localhost:9527/).

   - **Login credentials:**
     - **Username:** `admin`
     - **Password:** `apachepulsar`

6. Create a new environment in Pulsar Manager with the following details:

   - **Service URL:** `http://broker:8080`
   - **Bookie URL:** `http://localhost:8080/admin/v2/bookies`

   These URLs are used to configure the connection to the Pulsar broker and bookie services.

## Service Details

### Networks

- **`pulsar`**: A Docker network used to connect the services in the cluster.

### Volumes

- **`pulsar-setup`**: A volume for persistent storage.

### Services Overview

#### Zookeeper
- Manages cluster metadata.
- Configurations are applied dynamically.
- Healthcheck command ensures service readiness.

#### Pulsar Init
- Initializes cluster metadata.
- Ensures dependent services are healthy before execution.

#### Bookie
- Stores messages for the Pulsar broker.
- Persistent storage is mapped to `./data/bookkeeper`.
- Includes a workaround for startup issues related to cookies.

#### Broker
- Provides messaging services.
- Exposes ports `6650` and `8080` for client and management access respectively.

#### Pulsar Manager
- Web-based UI for managing and monitoring the Pulsar cluster.
- Exposes ports `7750` and `9527`.
- Requires manual setup of admin user and environment configuration.

## Notes

- Ensure that the local directories `./data/zookeeper` and `./data/bookkeeper` exist before starting the services.
- If you encounter issues, review the logs using:

```bash
docker logs <container-name>
```

## Additional Resources

- [Apache Pulsar Documentation](https://pulsar.apache.org/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## Troubleshooting

- Ensure the necessary ports (`6650`, `8080`, `7750`, `9527`) are not blocked or in use by other applications.
- Verify service dependencies are healthy before starting dependent services.
