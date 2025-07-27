# Scaling Node-RED

This project provides a setup for scaling Node-RED instances horizontally using Docker, NGINX, and MongoDB. It allows you to run multiple Node-RED instances that share the same flows, which are stored in a MongoDB database. Changes made to the flows in one instance are automatically propagated to all other instances in real-time.

This setup is specifically designed for scaling flows that are triggered by the HTTP-IN node in Node-RED. For other types of triggers, you may need to modify the setup to achieve scalability.

## Architecture

The architecture consists of the following components:

- **NGINX:** Acts as a reverse proxy, forwarding requests to the appropriate services.
- **Node-RED Frontend:** A single Node-RED instance that is used for editing the flows.
- **Node-RED API:** Multiple Node-RED instances that run the flows. Docker's internal DNS resolver load balances requests across these instances.
- **MongoDB:** A replica set that stores the Node-RED flows.

## Components

### Custom Node-RED Image

The `customNodeRed` directory contains the files to build a custom Node-RED Docker image.

- **`Dockerfile`:** Defines the steps to build the image. It starts from the official Node-RED image and installs the `node-red-mongo-storage-plugin-with-sync` npm package.
- **`nodered-settings.js`:** The settings file for Node-RED. It configures Node-RED to use the `node-red-mongo-storage-plugin-with-sync` as the storage module and sets the `httpNodeRoot` to `/red-nodes`.
- **`package.json`:** A placeholder package.json file.

The `node-red-mongo-storage-plugin-with-sync` is a crucial part of this setup. It stores all Node-RED flows in a MongoDB collection and listens for changes in that collection. When a change is detected (e.g., a flow is updated), the plugin automatically reloads the flows, ensuring that all instances are running the latest version of the flows.

### NGINX

The `nginx` directory contains the NGINX configuration file.

- **`nginx.conf`:** The NGINX configuration file. It routes all requests to the `/red-nodes` path to the `nodered-api` service. Docker's internal DNS resolver will then load balance the requests across the multiple `nodered-api` containers using a round-robin strategy. All other requests are routed to the `nodered-frontend` service, which is a single Node-RED instance used for editing the flows.

### Docker Compose

The `docker-compose.yaml` file defines the services that make up the application.

- **`nginx`:** The NGINX reverse proxy.
- **`mongo`:** The MongoDB replica set.
- **`nodered-frontend`:** The Node-RED instance for editing flows.
- **`nodered-api`:** The Node-RED instances for running flows. The number of replicas can be scaled up or down as needed.

## How it works

1.  All Node-RED instances use the same MongoDB database to store their flows.
2.  The `nodered-frontend` instance is used to edit the flows. When the flows are updated, the changes are saved to the MongoDB database.
3.  The `node-red-mongo-storage-plugin-with-sync` in each Node-RED instance detects the changes in the MongoDB database and automatically reloads the flows.
4.  The `httpNodeRoot` key in the `nodered-settings.js` file is set to `/red-nodes`. This means that all flows that use the HTTP-IN node will have the `/red-nodes` prefix in their URL.
5.  NGINX forwards all requests with the `/red-nodes` prefix to the `nodered-api` service. Docker's DNS resolver then load balances these requests across all the `nodered-api` containers.
6.  All other requests are routed to the `nodered-frontend` instance, which is used for editing the flows.

## Getting Started

To get started with this project, you need to have Docker and Docker Compose installed.

1.  Clone this repository.
2.  Run the following command to start the application:

```bash
docker compose up -d
```

This will start all the services defined in the `docker-compose.yaml` file.

You can then access the Node-RED editor at `http://localhost:8080`. Any changes you make to the flows will be saved to the MongoDB database and propagated to all other Node-RED instances.

## Scaling

To scale the number of `nodered-api` instances, you can use the following command:

```bash
docker compose up --scale nodered-api=<number_of_instances> -d
```

For example, to scale up to 5 instances, you would run:

```bash
docker compose up --scale nodered-api=5 -d
```

Docker will automatically distribute the requests to the `/red-nodes` path across all the running instances.
