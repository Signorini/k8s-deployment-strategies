---
# Canary release [Update version with low risks]: 
The canary release consist into deploy a version b alongside a version a, but route a subsets of users to a new version, you can start with a small percentage, 1% to a new version and 99% to old version, and increase this value accordingly, this release is excellent when you don’t know the impact of the new release, the concerns takes time to finish the deployment, can be expensive to handle the deployments flows and is typically very slow deployment normally more than rollout deployment.

![Canary](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/canary.png)


---

##  Ways

**Native** - Using built-in kubernetes functionalities

**Ingress** - Using nginx ingress

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

## Deploy canary release

Deploy version 2 on kubernetes, they will run side by side

```
kubectl apply -f native/app2.yml
```

The trick is, app1 it's deployed as 9 pods, and app2 only 1 pod, both use the same label to be matched by nodeport service, the nodeport will route 90% of traffic to app1 and 10% to app2, if you like to increase this number, you can:
```
kubectl scale deployment my-nginx --replicas=9
// Will send 50% of the traffic in both versions
```

If all is good, remove the first version.
```
kubectl delete -f native/app1.yml
```


# Ingress

## Create first deployment

Create a deployment v1

```
kubectl apply -f ingress/app1.yml
```

Create the ingress and point out to app1 deployment.

```
// Setup the ingress rbac and config maps
kubectl apply -f ingress/ingress.yml

// Create a ingress rules to point out to service version 1
kubectl apply -f ingress/ingress-v1.yml

// Expose ingress externaly
kubectl apply -f ingress/expose-ingress.yml
```

Checking ingress rules
```
paths:
- backend:
    serviceName: my-cn1 // version 1
    servicePort: 80 // expose port 80
- backend:
    serviceName: my-cn1
    servicePort: 32111 // expose prometheus metrics.
```

## Do canary

To start to switch the traffic, deploy the canary release rule.
```
kubectl apply -f ingress/ingress-canary.yml
```

Ingress have a built-in canary release funcationality, can be activate using two annotations
```
nginx.ingress.kubernetes.io/canary: "true" // enabling canary release
nginx.ingress.kubernetes.io/canary-weight: "10" // percentage to switch to second application
```

When all it's good you can set to send 100% traffic to version 2
```
kubectl apply -f ingress/ingress-v2.yml
```