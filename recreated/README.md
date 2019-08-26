---
# Recreate [Go horse update]:
It’s a dummy release, consist in turn off version a and turn on the version b using the same resources, causing downtimes, can it be strange nowadays, but this strategy was very used, scheduling maintenance times and switch off the application [remember copy and past file through ftp]. It’s the cheapest way to deploy a service.

We advise against this deploy normally the downtimes can long disable your customer to use your services.

![Recreate](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/recreate.png)

---

Native - Using built-in kubernetes functionalities

##  Setup the env

*Minikube* - Install minikube

*Prometheus* - Install prometheus using helm template.

*Grafana* - Install grafana and link prometheus as source.

## Create applications

Create a deployment v1

```
kubectl apply -f native/app1.yml
```

Expose a service using NodePort

```
kubectl apply -f native/expose.yml
```

## Do the deploy

Switch the version appling deployment v2

```
kubectl apply -f native/app2.yml
```