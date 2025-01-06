## Monitor Resource Usage and Scale Resources

to monitor resource usage and accuracy

**1)**
 Use Metrics Server: Install k8s metrics server for collecting resource metrics. Metrics server provides an aggragated resource usage data.

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

verify installation
kubectl top pods -n dev-environments
kubectl top nodes

**2)** 
Integrate Metrics server with Prometheus and Grafana: 
Prometheus collects and stores metrics about the cluster and workloads, including CPU and memory usage, while Grafana visualizes metrics collected by Prometheus and displays them on customizable dashboards.

Use the official Helm chart to install Prometheus and Grafana:
`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
`helm repo update`
`helm install prometheus prometheus-community/kube-prometheus-stack`

Create a dashboard in Grafana to monitor:
Resource Requests vs Actual Usage for CPU and Memory. Identify underutilized resources.


You can also create a kubernetes dashboard:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml`

`kubectl proxy`

access the dashboard locallyhttp://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

To notify when resources are idle or underutilized, one would need to create prometheus alerts:

- Define custom alerts in Prometheus to monitor resource usage thresholds:I setup a sample file in the prom folder.
Then you would need to configure the alert manager to send notifications to slack, pagerduty or any other service you have.

**3)** 
Downscale Resources
To downscale resources when idle or underutilized, one would need to adjust the resource requests and replicas based on usage. To do this, HPA would be a good choice.

Enabling HPA, requires Metrics Server, which was reccommended to be installed above.
I have setup a sample HPA yaml file in the prom dir

**4)** 
Vertical Pod Autoscaler (VPA)
VPA adjusts CPU an memory usage: install VPA `kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
`

**5)** 
Node Autoscalling: To automatically scale up or down a cluster, one needs to set up Node Autoscaling
The cluster's node autoscaler adds or removes nodes to match the resource requirements of the workloads.
here is a sample cmd to create a multiple node groups using eksctl:

`eksctl create nodegroup --cluster=my-cluster --name=dev-node-group \`
  `--nodes=2 --nodes-min=1 --nodes-max=5 --node-labels=environment=dev --taints=environment=dev:NoSchedule`

`eksctl create nodegroup --cluster=my-cluster --name=uat-node-group \`
  `--nodes=2 --nodes-min=1 --nodes-max=5 --node-labels=environment=uat --taints=environment=uat:NoSchedule`


**6)**
A good way to automatically scale resources and update configurations in Kubernetes is to implement GitOps for automation. A good tool is ArgoCD, which helps to update, scale configurations in Helm chart and/or Kubernetes manifests, based on alerts.


**7)**
To architect a system where processes running inside environments above require 100-250GB of data in memory, requires careful planning and system design to ensure data is efficiently brought to aplication code, avoid memory bottleneck and cost.
Main challenges will be:
    - Handling Large Data sets in memory
    - Accessing data efficiently
    - and Cost optimization.

Steps to architect for large datasets:
    1. Object Storage: I would recommend use of an object storage (AWS S3) as the primary storage for large datasets. S3 provides high availability and durability. And you can access specific parts of the data (range queries).
        - If the application requires real-time data sharing, one can think of a Distributed File System, that allows for concurrent access to shared data by multiple pods
    2. Data loading and memory management: Loading the entire dataset at once (100-250GB) would be memory demanding, rather take advantage of streaming: Fetch only the necessary parts of data at runtime.
        - Use Distributed in-memory storage: Redis or Memcache for datasets less than 100GB and tools like Apache Ignite ofr datasets 250GB+. Kafka a another tool to lok to implement for streaming.
    3. Since the applications would be deployed in Kubernetes clusters, one would need to set apropriate memory limits and requests for pods.
        - Ensure nodes in the cluster have enough memory to suport resource requests to pods for datasets that large. 
        - You can take advantage of node affinity and taints/tolerations to assign high-memory workloads to specific nodes
    4. Data Processing: Using data processing frameworks would help alleviate some of the bottlenecks that one might encounter with such large datasets. Using frameworks like Apache Spark to distribute workload across multiple nodes or pods.
        - Batch vs Real time processing: If the dataset is not time-sensitive, process the data in batches to reduce memory pressures. If real-time processing is what is required, data streaming framework like Kafka is a good option.
    5. Observability and Monitoring: One would want to implement monitoring and observability for memory usage for the nodes and pods. Setup alerts for memory over-utilization.
    6. I talked about HPA and VPA for when scaling cluster above, but it is also a good option to use when looking to adjust the memory alocation.
        - VPA: use VPA to adjust for memory allocation dynamically based on usage.
        - HPA: If the workloads is distributed across multiple pods, using HPA to scale out based on memory utilization.
    
    These are teh steps I woudl take to load large datasets into application code.


