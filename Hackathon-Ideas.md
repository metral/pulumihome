This page is intended to capture a list of ideas for hackathon projects. Ideally these ideas are hackathon-sized or easily decomposable into hackathon-sized chunks. The precise definition of "hackathon-sized" is flexible, but "achievable in 4-6 hours" is a good starting point.

## The List
- [ ] Per-stack AWS bill evaluator
- [ ] Serverless Slack bot component
- [ ] [Factorio](http://factorio.com/) server, state persisted on S3
- [ ] Alerting stack (send SMS, automated voice, etc. in response to event)
- [ ] Remove now dead code referencing PPCs
- [ ] Simple implementation of an "import from terraform" service using https://github.com/pulumi/tf2pulumi
- [ ] A set of Babel extensions that restrict JS to a pure, hermitic, control-flow-less subset. For the crowd that like HCL because it's "not programming". Advantage: you can just flip a switch and turn on "real" JS at any time!
- [ ] A converter from a set of Swagger API definitions to a Pulumi provider
- [ ] A demo with [go-cloud](https://github.com/google/go-cloud), using one of their examples but not using Terraform

## Past Projects
- [Twilio Typescript Component](https://github.com/pulumi/examples/tree/master/twilio-ts-component)
- [k8s Jenkins Component](https://github.com/pulumi/examples/tree/master/kubernetes-ts-jenkins)
- [EKS Example](https://github.com/pulumi/examples/tree/pgavlin/eks/aws-ts-eks)