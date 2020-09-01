# Deployment Types in Kubernetes
## Blue-Green Deployment
In blue-green deployments, there are two production environments: blue and green. Blue is the current version with live traffic and green is the environment with the updated code. At any time, only one has live traffic.

To release a new version, code is deployed to the environment with no traffic where final tests are performed. Once IT is confident the application is ready, all traffic is routed to the green environment. Green is now live and the actual release executed.

This is the first time the new code is tested with a production load (real-life traffic). Risks still remain until the code is actually released, that will never go away. But if something goes wrong, IT can quickly reroute the traffic back to the blue version. All they have to do is closely monitor code behavior, this can be automated through proper tooling, to see if green works well or if a rollback is needed.
