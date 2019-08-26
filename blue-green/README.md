---
# Blue/green [The best for major releases]:
The version b is deployed alongside to version a, after all traffic a requirements will be switched to new version, which is a safer deployment, you can do a test, rollback will be straightforward, concerns boil down in doubling resource utilization each time a deployment happens, and depends on how often a deploy happens this strategy can be expensive.

![blue-green](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/blue-green-grafana.png)

---

Native - Using built-in kubernetes functionalities