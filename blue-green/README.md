---
# Blue/green [The best for major releases]:
The version b is deployed alongside to version a, after all traffic a requirements will be switched to new version, which is a safer deployment, you can do a test, rollback will be straightforward, concerns boil down in doubling resource utilization each time a deployment happens, and depends on how often a deploy happens this strategy can be expensive.

![blue-green](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/blue-green-grafana.png)

---

##  Ways

**Native** - Using built-in kubernetes functionalities

##  Setup the env

To start, we need a kubernetes cluster with prometheus and grafana installed and running.

**Minikube** - The easiest way to get a kubernetes cluster for develop. [Install minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

**Helm** - It's package manager for Kubernetes, is the easy way to install service on kubernetes. [Install Helm](https://helm.sh/docs/using_helm/#quickstart)

**Prometheus** - Install prometheus using helm template. [Install prometheus](https://github.com/helm/charts/tree/master/stable/prometheus)
```
helm install --name prometheus stable/prometheus
```

**Grafana** - Grafana dashboard. [Install grafana](https://github.com/helm/charts/tree/master/stable/grafana)
```
helm install --name grafana stable/grafana
```

### Connected prometheus into grafana

Add prometheus as source on grafana.

### Checking if everything is ok

```
> kubectl get pods

NAME                                             READY   STATUS    RESTARTS   AGE
grafana-58cdc7db6c-46srl                         1/1     Running   3          26d
prometheus-alertmanager-6db559d695-42zf8         2/2     Running   6          26d
prometheus-kube-state-metrics-5db7466b57-5brsq   1/1     Running   6          26d
prometheus-node-exporter-mfwps                   1/1     Running   3          26d
prometheus-pushgateway-59bd88779d-2h9cn          1/1     Running   7          36d
prometheus-server-5f74c4749-bcjnk                2/2     Running   21         36d
```

--- 

### Create a grafana dashboard

You can import grafana dashboard using my template, /grafana/templates.json

To test each deployment I use simple querie to aggregate all nginx requests.

```
sum(irate(nginx_http_requests[5m])) by (version)
```

To count all containers running on kubernetes, use the querie below.

```
sum(kube_pod_container_info{image=~"signorini/nginx-openrest:.*"}) by (image)
```

---

# Native

## Create first deployment

Create a deployment v1

```
kubectl apply -f native/app1.yml
```

Expose a service using NodePort

```
kubectl apply -f native/expose.yml
```

The service it's a single nginx [openrest], expose a single endpoint.

```
curl http://localhost:31028/;
```

Simple way to generate load into endpoint.
```
while; do; curl -H "Host: app.com" http://localhost:31028/; done;
```

## Deploy a green release

Deploy version 2 on kubernetes, they will run side by side

```
kubectl apply -f native/app2.yml
```

To switch the traffic you can change the matched selector on service
```
kubectl apply -f native/expose2.yml
```

We have two version running in the same time and only one service routing the traffic.

```
kubectl get deploy
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
cn-v1                           2/2     2            2           27d
cn-v2                           2/2     2            2           27d

kubectl get svc
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
my-blue                         ClusterIP   10.109.115.2     <none>        80/TCP,32111/TCP               31d
```


If all is good, we can remove the first version.
```
kubectl delete -f native/app1.yml
```
