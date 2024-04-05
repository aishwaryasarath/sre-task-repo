# SRE Fundamentals with Google - Project - [Uplimit course](https://uplimit.com/course/sre-fundamentals-with-google)

In this project, the task is to deploy UpCommerce's website as a service on Kubernetes. 
- Define alert rules based on key performance indicators (KPIs) like CPU utilisation, memory usage, and request rates, allowing you to detect and respond to potential issues before they affect the user experience. 
- Create and deploy these rules using Prometheus. Prometheus is an open-source monitoring and alerting system designed for containerized environments, such as Kubernetes. It collects and stores metrics from various sources, including applications, infrastructure components, and custom exporters, allowing you to monitor the performance and health of your systems in real-time. Prometheus uses a powerful query language called PromQL to analyze and visualize the collected metrics, enabling you to gain valuable insights into the behavior of your applications and infrastructure.
- By the end of this project, we gain a comprehensive view of the UpCommerce website, enabling you to identify trends, patterns, and potential issues that may affect the user experience or the overall stability of the system. We'll also have gained hands-on experience in managing a production-grade e-commerce platform and the skills necessary to maintain its reliability and performance.

## Prerequisite

Set up docker, helm, minikube
Define the alert rules and triggers that should fire off the alerts in prometheus.yml


## Create a single-node, Kubernetes cluster using Minikube & create a namespace 
```
minikube start
kubectl create namespace sre
```

## Helm repo for prometheus
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus  -f prometheus.yml  --namespace sre   
```
Add and update helm repo for prometheus; installs the Prometheus chart from the prometheus-community repository into the Kubernetes cluster

## Helm repo for grafana
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana  --namespace sre  --set adminPassword="admin"
```
Add, update helm repo for grafana and install installs the Grafana chart from the grafana repository into the Kubernetes cluster
