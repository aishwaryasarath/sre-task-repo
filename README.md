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

## Create the deployment & service
```
kubectl apply -f deployment.yml -n sre
kubectl apply -f service.yml -n sre
```

## Create Minikube service and forwards it to a port 
```
minikube service upcommerce-service -n sre
```

## Forward  Prometheus AlertManager to a port 
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace sre port-forward $POD_NAME 9093
```

## Forward  Prometheus server to a port 
```
export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace sre port-forward $POD_NAME 9090
```

## Test the deployment
```
minikube service upcommerce-service -n sre --url
```
If the result is 
http://192.168.49.2:30521
# export $PORT=30521 from the above command
kubectl port-forward service/upcommerce-service -n sre $PORT:5000
```
<img width="1103" alt="image" src="https://github.com/aishwaryasarath/sre-task-repo/assets/49971693/078f4f4e-bdec-4e0d-9052-df36cd4d31f7">


## Slack and gmail alert
1. Create a slack workspace and a channel
2. Create an app at https://api.slack.com/apps/new
3. Activate incoming web hooks
<img width="695" alt="image" src="https://github.com/aishwaryasarath/sre-task-repo/assets/49971693/f0d73bc3-9f55-4868-9849-9a9987840b52">
4. Add new webhook to workspace and copy the Webhook URL and save it
<img width="297" alt="image" src="https://github.com/aishwaryasarath/sre-task-repo/assets/49971693/39c47339-913e-4a1e-8693-711d8fff72c3">
and select a channel(created in step 1) to post to as an app.

## Create a configmap to override the default prometheus-alertmanager configmap.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-alertmanager
data:
  alertmanager.yml: |
    global:
      resolve_timeout: 1m

    receivers:
    - name: 'notifications'
      email_configs:
      - to: aiswaryasarath847@gmail.com
        from: aiswaryasarath847@gmail.com
        smarthost: smtp.gmail.com:587
        auth_username: aiswaryasarath847@gmail.com
        auth_identity: aiswaryasarath847@gmail.com
        auth_password: test test test test
        send_resolved: true
        headers:
          subject: "Prometheus - Alert"
          text: "{{ range .Alerts }} Hi, \n{{ .Annotations.summary }}\n{{ .Annotations.description }} {{end}}"

      slack_configs:
      - channel: '#sre'
        send_resolved: true
        api_url: 'https://hooks.slack.com/services/testhookfromStep4Above'

    route:
      group_wait: 10s
      group_interval: 2m
      receiver: 'notifications'
      repeat_interval: 2m

```
Edit the cm
``
kubectl edit cm prometheus-alertmanager -n sre
```
The output would be similar to this saying that it has been saved to a temp file.
```
A copy of your changes has been stored to "/tmp/kubectl-edit-1867798316.yaml"
```
Apply this change with the below command
```
 kubectl apply -f /tmp/kubectl-edit-1867798316.yaml -n sre
```
Delete the existing pod for the config changes to take effect.
```
kubectl get pods -n sre
# The pod name from the above command was prometheus-alertmanager-0
kubectl delete pod prometheus-alertmanager-0  -n sre
```
Wait for the pod of the Stateful set to delete and restart. 

## Check gmail and slack for alerts

<img width="665" alt="image" src="https://github.com/aishwaryasarath/sre-task-repo/assets/49971693/bf181041-f882-4e0b-bbcc-24cf551491ba">
<img width="825" alt="image" src="https://github.com/aishwaryasarath/sre-task-repo/assets/49971693/f03e7d5d-705a-455c-a15f-a6a343d5bc94">

