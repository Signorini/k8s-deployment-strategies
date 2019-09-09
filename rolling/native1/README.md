---
# Delay to spin up - Native1

To get visibility as deployment effect on grafana dashboard, we set a delay to start the container, this config can be found into:
```
// native1/app2.yml

readinessProbe:
    httpGet:
    path: /
    port: 80

    initialDelaySeconds: 15 # in seconds
    periodSeconds: 5
```
