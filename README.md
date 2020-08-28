# usefulkube

Making `minikube` useful.

Assumes that the following tools make for a more useful local Kubernetes sandpit than `minikube start` and `minikube dashboard`:

* [Elastic Operator (ECK)](https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html) for log aggregation and search
* [Fluentbit](https://fluentbit.io/) for piping container logs to Elastic
* [Prometheus+Grafana](https://github.com/helm/charts/tree/master/stable/prometheus-operator) for instrumentation dashboards
* [Argo](https://argoproj.github.io/projects/argo) for container orchestration

Highly recommend [k9s](https://github.com/derailed/k9s) as a command-line level up for `kubectl`.

## Minikube Bootstrapping Config

Set parameters for creating the minikube node

```
minikube config set memory 16384
minikube config set cpus 4
```

Get Helm set up to pull in platform function charts:

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

## After Start

Create the platform namespace and install operators:

```
kubectl apply -f namespace.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/1.2.1/all-in-one.yaml
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/stable/manifests/install.yaml
```

Start the dashboard to track status of deployed services

```
minikube dashboard
```

Install the Helm required charts:

```
helm install prometheus stable/prometheus-operator --namespace platform
helm install logs ./logs --namespace platform
helm install fluent stable/fluent-bit -f fluentbit_values.yaml --namespace platform
```

```
kubectl get secret logs-es-elastic-user --namespace=platform -oyaml | grep -v '^\s*namespace:\s' | kubectl apply --namespace=default -f -
```

## Portal Logins

Kibana

```
echo `kubectl get secret logs-es-elastic-user -n platform -o=jsonpath='{.data.elastic}' | base64 --decode`
```

Prometheus

```
echo `kubectl get secret prometheus-grafana -n platform -o=jsonpath='{.data.admin-password}' | base64 --decode`
```

## Accessing Platform Services

```
kubectl port-forward --namespace platform service/prometheus-grafana 3000:80
kubectl port-forward --namespace platform service/logs-kb-http 5601
kubectl port-forward --namespace argo deployment/argo-server 2746
```

## Other Useful Services

```
helm install postgres bitnami/postgresql
helm install mongo bitnami/mongodb
```
