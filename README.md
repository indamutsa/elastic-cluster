# 🚀 Elasticsearch Cluster Setup 🌍

Elasticsearch is a distributed, RESTful search and analytics engine capable of addressing a growing number of use cases. Its scalability and versatile nature make it a choice for applications that need to search large volumes of data. Running Elasticsearch as a cluster ensures high availability, fault tolerance, and scalability.

## Architecture 🏛️

Elasticsearch cluster components include:

- **Master Nodes**: Responsible for cluster-wide management and configuration tasks.
- **Data Nodes**: Store indexed data and handle CRUD, search, and aggregations.
- **Client Nodes**: Route search and index requests but don’t store any data.
- **Ingest Nodes**: Pre-process documents before indexing.

Refer to the detailed architecture diagram for a visual representation:

![Elasticsearch Cluster Architecture](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/deploy_1@2x.png)

---

## Repository Structure 🌲

```
❯ tree
.
├── docker-compose.yml                 # Docker Compose configuration
├── elasticsearch                      # Elasticsearch Docker configurations
│   ├── Dockerfile                     # Docker build instructions for Elasticsearch
│   └── elasticsearch.yml              # Elasticsearch config file
├── filebeat                           # Filebeat configurations
│   ├── Dockerfile                     # Docker build instructions for Filebeat
│   └── filebeat.yml                   # Filebeat config file
├── k8                                 # Kubernetes configurations
│   ├── 1-es-master.yml                # Elasticsearch master node setup
│   ├── 2-es-worker1.yml               # Elasticsearch worker 1 setup
│   ├── 3-es-worker2.yml               # Elasticsearch worker 2 setup
│   ├── 4-logstash.yml                 # Logstash setup
│   ├── 5-kibana.yml                   # Kibana setup
│   ├── 6-filebeat-daemonset.yml       # Filebeat daemonset setup
│   └── spin-cluster.sh                # Shell script to deploy the cluster manifests
├── kibana                             # Kibana configurations
│   ├── Dockerfile                     # Docker build instructions for Kibana
│   └── kibana.yml                     # Kibana config file
├── logstash                           # Logstash configurations
│   ├── Dockerfile                     # Docker build instructions for Logstash
│   ├── logstash.yml                   # Logstash config file
│   └── pipeline
│       └── logstash.conf              # Logstash pipeline configuration
└── spinner.sh                         # Helper script for quick deployments
```

---

## Docker Compose Setup 🐳

### 1. Starting the Cluster 🚀

We can spin up all the environment components using Docker Compose and folder structure. To initiate the Elasticsearch cluster via Docker Compose, execute:

```bash
╰─❯ chmod +x spinner.sh
╰─❯ ./spinner.sh
```

To initiate the Elasticsearch cluster via Docker Compose, execute:

```bash
╰─❯ docker-compose up --build --force-recreate -d
```

This action will construct the Docker images, instantiate the containers, and ensure they're running without hitches.

### 2. Connecting to the Cluster 🔗

You can interact with your Elasticsearch cluster using various tools. One popular method is via the Elasticsearch RESTful API:

You can check if the cluster is up and running by navigating to:

```bash
❯ curl -u elastic:changeme -XGET 'http://localhost:9200/_cluster/health?pretty=true'
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 9,
  "active_shards" : 18,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

Head to Kibana at `http://localhost:5601/` to interact with the cluster.
The default credentials are `username: elastic, password: changeme`.

Upon connection, you should see the Elasticsearch cluster's status and details.

### 3. Cleaning Up Resources 🧹

To dismantle the cluster or start anew, use:

```bash
╰─❯ docker-compose down -v --remove-orphans && docker system prune -a --volumes
```

This command halts and eradicates containers, their volumes, and cleans up idle Docker assets.

---

## Kubernetes Setup with `kind` 🌐

Deploying an Elasticsearch cluster on Kubernetes offers scalability, high availability, and flexibility. This guide leverages `kind` for local Kubernetes cluster deployment.

### Prerequisites 📚

- Ensure you have `kind` and `kubectl` installed.
- A tool like Postman or a browser for interacting with the Elasticsearch RESTful API.

### Deployment Steps 🚀

#### 1. Navigate to the Kubernetes Configuration Directory

Before running any commands, ensure you are in the `k8` directory where the Kubernetes manifests are located.

```bash
cd k8
```

#### 2. Run the Initialization Script

This script will generate the necessary Kubernetes manifests for the Elasticsearch cluster.

```bash
./spin-cluster.sh
```

#### 3. Set Up the `kind` Cluster

If you already have a cluster named `es-cluster`, it's best to delete it and recreate:

```bash
kind delete cluster --name es-cluster
kind create cluster --name es-cluster
```

#### 4. Deploy the Elasticsearch Cluster

Apply the generated Kubernetes manifests to your `kind` cluster:

```bash
kubectl apply -f .
```

#### 5. Monitor the Deployment

To observe the status of the pods as they are being created:

```bash
watch -n 3 kubectl get po
```

Bear in mind, the Elasticsearch cluster components might take some time to initialize and become ready.

#### 6. Connecting to the Cluster

Once all the pods are up and running, establish a connection to the Elasticsearch service:

```bash
kubectl port-forward svc/es-master 9200
```

Now, you can interact with your Elasticsearch cluster using the RESTful API:

```
http://localhost:9200/
```

You can check if the cluster is up and running by navigating to:

```bash
❯ curl -u elastic:changeme -XGET 'http://localhost:9200/_cluster/health?pretty=true'
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 9,
  "active_shards" : 18,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

Head to Kibana at `http://localhost:5601/` to interact with the cluster.
The default credentials are `username: elastic, password: changeme`.

#### Optional: Inspecting the Cluster

For a deeper inspection of pods, services, or deployments, you can use the `describe` or `logs` commands:

```bash
# For Pod details
kubectl describe po [POD_NAME]

# For Service details
kubectl describe svc [SERVICE_NAME]

# For Deployment details
kubectl describe deploy [DEPLOYMENT_NAME]

# To view logs of a specific pod
kubectl logs [POD_NAME]
```

---

## Clean-Up 🧹

When done with the cluster, don't forget to delete the `kind` cluster to free up resources:

```bash
kubectl delete all,secrets,configmaps,pv,pvc --all --all-namespaces
kind delete cluster --name es-cluster
```

---

**Congratulations!** 🎉 You've successfully deployed an Elasticsearch cluster on Kubernetes using `kind`. Dive deeper, refine your deployments, and unlock the power of scalable search.
