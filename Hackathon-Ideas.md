This page is intended to capture a list of ideas for hackathon projects. Ideally these ideas are hackathon-sized or easily decomposable into hackathon-sized chunks. The precise definition of "hackathon-sized" is flexible, but "achievable in 4-6 hours" is a good starting point.

## Requests for 2/23
- [ ] A "normal" Kubernetes app - live in the inner loop of Kubernetes application development
  - Pat
- [ ] A cloud+kubernetes app - use RDS and/or S3 with a Kubernetes app
   - Discourse? [Matt] + EKS + Windows
- [ ] Helm Test Harness
- [ ] Helm [Justin]
  - Two tier app
  - MySQL Helm chart
- [ ] Kubernetes on every platform
   - Try writing apps on: { GKE, AKS, EKS, Minikube }
   - Write high-order provider libraries for: { GKE }
   - Try writing apps on { macOS, Linux, Windows }
   - Better test coverage
- [ ] Heptio QuickStart [Joe]
- [ ] OpenFaaS [Pat] + GKE + macOS
- [ ] "Blueprints" Sidecars (Envoy, Prometheus, Istio, Linkerd) 
  - [Cyrus for Prometheus] Node client app
  - [Sean Envoy]
- [ ] Spinnaker CI/CD integration [Chris]

## Other Ideas
- [ ] Per-stack AWS bill evaluator
- [ ] Serverless Slack bot component
- [ ] [Factorio](http://factorio.com/) server, state persisted on S3
- [ ] Alerting stack (send SMS, automated voice, etc. in response to event)
- [ ] Remove now dead code referencing PPCs
- [ ] Simple implementation of an "import from terraform" service using https://github.com/pulumi/tf2pulumi
- [ ] A set of Babel extensions that restrict JS to a pure, hermitic, control-flow-less subset. For the crowd that like HCL because it's "not programming". Advantage: you can just flip a switch and turn on "real" JS at any time!
- [ ] A converter from a set of Swagger API definitions to a Pulumi provider
- [ ] A demo with [go-cloud](https://github.com/google/go-cloud), using one of their examples but not using Terraform
- [ ] Pulumify a deployment of Hashicorp Vault and mix it up with @ellismg's application-level secrets
- [ ] Google Calendar resource provider and Pulumi app for managing shared team vacation calendar
- [ ] PagerDuty resource provider and Pulumi app for managing oncall schedule
- [ ] "Deploy with Pulumi" VS Code extension
- [X] Experiment with GraphQL-based API in the service and front-end feature that uses it (@swgillespie did it - doesn't really work out)
- [ ] Deploy Gatsby to S3/CloudFront with Pulumi
- [ ] Analyze a Pulumi program's checkpoint / resources and generate the specific IAM policy necessary to update the stack.

## Past Projects
- [Twilio Typescript Component](https://github.com/pulumi/examples/tree/master/twilio-ts-component)
- [k8s Jenkins Component](https://github.com/pulumi/examples/tree/master/kubernetes-ts-jenkins)
- [EKS Example](https://github.com/pulumi/examples/tree/pgavlin/eks/aws-ts-eks)