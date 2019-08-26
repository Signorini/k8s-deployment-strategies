---
# Rolling update [Fewer resources without outages]: 
In the process, you replace one service for a new version slowly, usually remove one machine of the pool update the software with a new version and put back on the pool. Very useful when the system is stateful, rebalance data, can get 0 downtimes, and don’t request a double resource as blue or canary releases, the concerns are hard to test multiple versions, hard to handle data schema and major releases.

Tools like AWS Code Deploy, spinnaker, kubernetes by default use rolling update to switch versions. 
The major downside for rolling release is, in a few moments you will have an inconsistent situation in the middle of the release.

![Rolling](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/rolling.png)

---

Native - Using built-in kubernetes functionalities