---
# Canary release [Update version with low risks]: 
The canary release consist into deploy a version b alongside a version a, but route a subsets of users to a new version, you can start with a small percentage, 1% to a new version and 99% to old version, and increase this value accordingly, this release is excellent when you don’t know the impact of the new release, the concerns takes time to finish the deployment, can be expensive to handle the deployments flows and is typically very slow deployment normally more than rollout deployment.

![Canary](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/canary.png)


---

Native - Using built-in kubernetes functionalities

Ingress - Using nginx ingress