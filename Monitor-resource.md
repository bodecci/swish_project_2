## Monitor Resource Usage

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
Setting up Node Autoscaling
The cluster's node autoscaler adds or removes nodes to match the resource requirements of the workloads.
here is a sample cmd to create a multiple node groups using eksctl:

`eksctl create nodegroup --cluster=my-cluster --name=dev-node-group \`
  `--nodes=2 --nodes-min=1 --nodes-max=5 --node-labels=environment=dev --taints=environment=dev:NoSchedule`

`eksctl create nodegroup --cluster=my-cluster --name=uat-node-group \`
  `--nodes=2 --nodes-min=1 --nodes-max=5 --node-labels=environment=uat --taints=environment=uat:NoSchedule`


**6)**
A good way to automatically scale resources and update configurations in Kubernetes is to implement GitOps for automation. A good tool is ArgoCD, which helps to update, scale configurations in Helm chart and/or Kubernetes manifests, based on alerts.