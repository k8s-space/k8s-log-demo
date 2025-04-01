
# Kubernetes Setup for NGINX, Fluent Bit, and Elasticsearch

This repository provides Kubernetes configuration files to easily set up a deployment involving NGINX, Fluent Bit, and Elasticsearch. The setup is designed to collect NGINX logs, process them with Fluent Bit, and store the logs in an Elasticsearch database for easy querying and visualization.

## Prerequisites

- Kubernetes cluster
- kubectl configured to access the cluster
- curl command-line tool
- Basic knowledge of Kubernetes, Fluent Bit, NGINX, and Elasticsearch

## Setup

1. **Clone the repository:**

   ```bash
   git clone https://github.com/k8s-space/k8s-log-demo
   cd k8s-log-demo
   ```

2. **Deploy the elasticsearch files:**

   Apply the Kubernetes manifests for Elasticsearch.

   ```bash
   kubectl apply -f elasticsearch-config.yaml -k8s-log-demo
   kubectl apply -f elasticsearch-deployment.yaml -k8s-log-demo
   
   kubectl apply -f k8s/nginx-deployment.yaml
   kubectl apply -f k8s/fluent-bit-deployment.yaml
   ```

3. **Expose the elasticsearch deployment:**

   Creates the service needed for fluent bit to connect to elasticsearch.
   
   ```bash
   kubectl expose deployment -n k8s-log-demo elasticsearch
   ```

4. **Deploy the fluent bit files:**

   Apply the Kubernetes manifests Fluent Bit.
   
   ```bash
   kubectl apply -f fluentbit-config.yaml -k8s-log-demo
   kubectl apply -f fluentbit-deployment.yaml -k8s-log-demo
   ```

4. **Deploy the nginx application:**

   Apply the Kubernetes manifests NGINX.
   
   ```bash
   kubectl apply -f nginx-deployment.yaml -k8s-log-demo
   ```
   
5. **Verify the deployment:**

   Check the status of the pods to ensure they are running correctly:

   ```bash
   kubectl get pods
   ```
   
## Create Index and Mapping in Elasticsearch

   Index erstellen
    
   ```bash
   curl -X PUT "http://<elasticsearch-host>:9200/nginx-logs" -H 'Content-Type: application/json' -d'
   {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 1
   }
   }
   '
   ```

   Mapping erstellen
    
   ```bash
   curl -X PUT "http://<elasticsearch-host>:9200/nginx-logs/_mapping" -H 'Content-Type: application/json' -d'
   {
        "properties": {
            "@timestamp": {
            "type": "date"
            },
            "remote_addr": {
            "type": "ip"
            },
            "status": {
            "type": "integer"
            },
            "request": {
            "type": "text"
            },
            "body_bytes_sent": {
            "type": "integer"
            },
            "http_referer": {
            "type": "text"
            },
            "http_user_agent": {
            "type": "text"
            }
        }
   }
   '
   ```
    
#### Look up the log entries in elasticsearch
   ```bash
   curl -X GET "http://localhost:9200/nginx-logs/_search?pretty" -H 'Content-Type: application/json' -d '
   {
   "size": 10,
   "sort": [{ "timestamp": { "order": "desc" } }]
   }'
   ```

## After everything runs you can check your logging

### Generating Logs

To generate logs in the NGINX server, use the following `curl` command to send a request to the NGINX service:

   ```bash
   curl http://<nginx-service-ip>/your-endpoint
   ```

This will trigger a log entry in the NGINX logs, which Fluent Bit will collect and forward to Elasticsearch.


## Components

- **NGINX:** Acts as the web server and generates logs for HTTP requests.
- **Fluent Bit:** Collects the logs from NGINX and forwards them to Elasticsearch.
- **Elasticsearch:** Stores the logs in a searchable database for analysis.


## Cleanup

To clean up the deployed resources, run:

   ```bash
   kubectl delete service -n k8s-log-demo --all
   kubectl delete deployment -n k8s-log-demo --all
   kubectl delete configmaps -n k8s-log-demo --all
   ```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
