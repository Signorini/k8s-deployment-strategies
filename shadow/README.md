---
# Shadow release [Testing high critical services without impact/risk]:
Traffic shadowing is a deployment pattern where production traffic is asynchronously copied to a non-production service for testing. It can be fit for situations to test critical apps, or apps hard to revert like payments gateways is good for persistent services, performance testing and/or behaviour without swift data or control the flow without impact.

![Shadow](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/shadown.png)

---
Istio - Using Istio virtual gateway