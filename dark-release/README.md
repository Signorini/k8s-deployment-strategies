---
# A/B release [The best to test new features]:
A/B testing is very similar to a canary release, normally used for business decision-based in statistics and data rather than deployment itself. Technically point of view the implementation is the same as a canary release, a/b release consist of mark a pool of users based on some parameters, tracking using cookies/headers etc and send a specific version, can be two or more per group.

![ab](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/ab.png)

---

Ingress - Using nginx ingress