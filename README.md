---
# Application deploy strategies on Kubernetes
Which is the best strategy to deploy and manage release? There are a lot of tools, ways and flavours, using api on the cloud or simple ftp on datacenter, different types and sizes, big monolithic or a simple lambda function, a vital question is why? Why now, why this? Do we suffer for any cargo cult feeling? Premature optimise? Tools? Or the maturity level? Today we are going to talk about deployment strategies in kubernetes perspective, I used a simple minikube environment to show how each strategy works.

Moreover, to point out a release and deploy it's two different things. The release is when you promote a new environment and deploy its an intrinsic to CD pipelines, release its When and deploy is How.

## Helping you to choose, You can do some questions:
1 — How often it’s your lifecycle release? Where are you getting at? 

2 — Is it a stressed system?

3 — Impact on a customer?

4 — How many efforts it’s required to apply the strategy?

5 — If it is broken how long time I need to going rollback?


The first point, deployments seem simple, however the hard part is to adopt the strategy on existing culture, change tools it’s fast, change people it’s slow, most of the time the decision its embasement accordingly the maturity and the way how each team works, after then start to improve step by step.

The second point is, normally we use 2 to 3 strategy at the same time and change the strategy accordingly the context.

## Let's talk about
*Recreate:* Destroy the old version and up the new version.
*Rolling update:* The version b is slowly rolled out replacing version a.
*Blue/green:* New version is shipped alongside to old version and the traffic its switch off.
*Canary release:* New version its deployed to a subset of users and gradually increment for all users.
*A/B release:* AAs a canary release, the subset it’s defined by specific conditions.
*Shadowing:* The traffic is sending to both versions, and version b don’t impact on the response.

---

## Recreate [Go horse update]:
It’s a dummy release, consist in turn off version a and turn on the version b using the same resources, causing downtimes, can it be strange nowadays, but this strategy was very used, scheduling maintenance times and switch off the application [remember copy and past file through ftp]. It’s the cheapest way to deploy a service.

We advise against this deploy normally the downtimes can long disable your customer to use your services.

![Recreate](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/recreate.png)

[Show me the code - Recreate release](https://github.com/Signorini/k8s-deployment-strategies/tree/master/recreated)

---

## Rolling update [Fewer resources without outages]: 
In the process, you replace one service for a new version slowly, usually remove one machine of the pool update the software with a new version and put back on the pool. Very useful when the system is stateful, rebalance data, can get 0 downtimes, and don’t request a double resource as blue or canary releases, the concerns are hard to test multiple versions, hard to handle data schema and major releases.

Tools like AWS Code Deploy, spinnaker, kubernetes by default use rolling update to switch versions. 
The major downside for rolling release is, in a few moments you will have an inconsistent situation in the middle of the release.

![Rolling](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/rolling.png)

[Show me the code - Rolling release](https://github.com/Signorini/k8s-deployment-strategies/tree/master/rolling)

---

## Blue/green [The best for major releases]:
The version b is deployed alongside to version a, after all traffic a requirements will be switched to new version, which is a safer deployment, you can do a test, rollback will be straightforward, concerns boil down in doubling resource utilization each time a deployment happens, and depends on how often a deploy happens this strategy can be expensive.

![blue-green](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/blue-green-grafana.png)

[Show me the code - Blue/gree](https://github.com/Signorini/k8s-deployment-strategies/tree/master/blue-green)

---

## Canary release [Update version with low risks]: 
The canary release consist into deploy a version b alongside a version a, but route a subsets of users to a new version, you can start with a small percentage, 1% to a new version and 99% to old version, and increase this value accordingly, this release is excellent when you don’t know the impact of the new release, the concerns takes time to finish the deployment, can be expensive to handle the deployments flows and is typically very slow deployment normally more than rollout deployment.

![Canary](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/canary.png)

[Show me the code - Canary release](https://github.com/Signorini/k8s-deployment-strategies/tree/master/canary)

---

## A/B release [The best to test new features]:
A/B testing is very similar to a canary release, normally used for business decision-based in statistics and data rather than deployment itself. Technically point of view the implementation is the same as a canary release, a/b release consist of mark a pool of users based on some parameters, tracking using cookies/headers etc and send a specific version, can be two or more per group.

![ab](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/ab.png)

[Show me the code - A/B release](https://github.com/Signorini/k8s-deployment-strategies/tree/master/dark-release)

---

## Shadow release [Testing high critical services without impact/risk]:
Traffic shadowing is a deployment pattern where production traffic is asynchronously copied to a non-production service for testing. It can be fit for situations to test critical apps, or apps hard to revert like payments gateways is good for persistent services, performance testing and/or behaviour without swift data or control the flow without impact.

![Shadow](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/shadown.png)

[Show me the code - Shadowing](https://github.com/Signorini/k8s-deployment-strategies/tree/master/shadow)

---

## Finally, catch up
We have multiple strategies with pros and cons, early times normally budget and time efforts always are a critical value to use. However, tools like Kubernetes and Istio/App Mesh came away to turn easy for every company to implement, now we can choose multiple types of deployment for the same apps accordingly each situation, minor changes can be done by rolling release, big changes can be finished by blue/green, new features can handle by A/B and new providers can be tested using shadow release.
