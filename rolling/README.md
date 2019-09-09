---
# Rolling update [Fewer resources without outages]: 
In the process, you replace one service for a new version slowly, usually remove one machine of the pool update the software with a new version and put back on the pool. Very useful when the system is stateful, rebalance data, can get 0 downtimes, and don’t request a double resource as blue or canary releases, the concerns are hard to test multiple versions, hard to handle data schema and major releases.

Tools like AWS Code Deploy, spinnaker, kubernetes by default use rolling update to switch versions. 
The major downside for rolling release is, in a few moments you will have an inconsistent situation in the middle of the release.

![Rolling](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/rolling.png)

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

## Create first deployment

Create a deployment v1

```
kubectl apply -f native1/app1.yml
```

Expose a service using NodePort

```
kubectl apply -f native1/expose.yml
```

The service it's a single nginx [openrest], expose a single endpoint.

```
curl http://localhost:31028/;
```

Simple way to generate load into endpoint.
```
while; do; curl -H "Host: app.com" http://localhost:31028/; done;
```


## Do the second deployment

Switch the version appling deployment v2

```
kubectl apply -f native1/app2.yml
```

Kubernetes will remove one or more pods and then spin up a new pods, repeat the process until finish the pool.

This is the default configuration
```
spec:
  strategy:
    type: RollingUpdate  
```

To control how many pods can be removed at same time use `RollingUpdateStrategy` parameter.

```
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

- Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 25% of the desired number of Pods are up (25% max unavailable).
- Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 25% of the desired number of Pods are up (25% max surge).

## Delay to spin up

To show more clear the deployment effect on grafana dashboard, we set a delay to start the container, this config can be found into:
```
// native1/app2.yml

readinessProbe:
    httpGet:
    path: /
    port: 80

    initialDelaySeconds: 15 # in seconds
    periodSeconds: 5
```
