---
date: 2018-03-29T20:57:54+08:00
title: Roadmap
weight: 30
---

Quo vadis? 

## Canary test and release a responsive frontend

_“We need to upgrade our website frontend to make it responsive”_
  
Developing a responsive web frontend is often a major undertaking, requiring a large investment of hours and extensive testing. Until you go live, it's difficult to predict how the upgrade will be received by users - will it actually improve important metrics, will it work on all browser, devices and resolutions, etc.?   

But why develop this new responsive frontend in one go, having to go for the dreaded and risky big-bang release? Using Vamp you can apply a canary release to introduce the new frontend to a selected cohort of users, browsers and/or devices. This would require a minimal investment of development and delivering real usage data:

1. __Start small:__ Build the new frontend for only one specific browser/resolution first to measure effectiveness. Vamp can deploy the new responsive frontend and route a percentage of supported users with this specific browser and screen-resolution there. All other users will continue to see the old version of your website.
2. __Optimise:__ With the new responsive frontend in the hands of real users with this specific browser/resolution, you can measure actual data and optimise accordingly without negatively affecting the majority of your users.
3. __Scale up:__ Once you are satisified with the performance of the new frontend, you can use Vamp to scale up the release, developing and canary releasing one browser/resolution at a time. Of course other cohort combinations are also possible, Vamp is open and supports all HAProxy ACL rules.

## Move from monoliths and VM's to microservices

_“We want to move to microservices, but we can’t upgrade all components at once and want to do a gradual migration”_

Refactoring a monolithic application to a microservice architecture is a major project. A big bang style re-write to upgrade all components at once is a risky approach and requires a large investment in development, testing and refactoring.

Why not work incrementally? Using AIE's routing you could introduce new services for specific application tiers like the frontend or business logic layers, and move traffic with specific conditions to these new services. These services in turn can connect to your legacy systems again, using AIE's proxying. A typical example would be introducing an angular based frontend or node.js based API microservice. You can send 2% of your incoming traffic to this new microservice frontend, which in turn connects with the legacy backend system. This way you can test your new microservices in a small and controlled way and avoid a big bang release. You can introduce new services one by one, test them in production and increase traffic until you migrated your entire application from a monolith to microservices.

1. __Start small:__ You can build e.g. one new frontend component. AIE will deploy this to run alongside the legacy monolithic system.
2. __Activate smart routing:__ AIE can route traffic behind the scenes, so a small percentage of visitors is sent to the new frontend service, while the new frontend is routed by AIE to the legacy backend. You can continue transferring components from the legacy monolithic system to new microservices and AIE can adapt the routing as you go.
3. __Remove legacy components:__ Once all services have been transferred from the legacy monolith, you can start removing components from the legacy system.

## Resolve client-side incompatibilities after an upgrade

_“We upgraded our website self-management portal, but our biggest client is running an unsupported old browser version”_
  
Leaving an important client unable to access your services after a major upgrade is a big and potentially costly problem. The traditional response would be to rollback the upgrade asap - if that's even possible.  

Why rollback? Using AIE's smart conditional routing you could send specific clients or browser-versions to an older version of your portal while others can enjoy the benefits of your new upgraded portal. Because AIE supports SLA based autoscaling for (Docker) containers, you can deploy the old version on the same infrastructure as the new version is running on. This also avoids having to provision costly over dimensioned DTAP environments for only a small user-base, leveraging your existing infrastructure efficiently.

1. __Re-deploy:__ AIE can (re)deploy a (containerised) compatible version of your portal to run side-by-side with the upgraded version.
2. __Activate smart routing:__ AIE can route all users with e.g. a specific IP, browser or location to a compatible version of the portal. Other clients will continue to see the new upgraded portal.
3. __Resolve the incompatibility:__ Once the client upgrades to a compatible browser AIE can automatically route them to the new portal version.

## Technology stack

- Golang/Java/Python
- OpenStack
- AWS
- Service Brokers
- Kubernetes(Helm, Istio)
- Terraform
- InSpec

## Contributing

Did you found an bug or you would like to suggest a new feature? I'm open for feedback. Please open a new [issue](https://github.com/digitalcraftsman/hugo-material-docs/issues) and let me know.

You're also welcome to contribute with [pull requests](https://github.com/digitalcraftsman/hugo-material-docs/pulls).
