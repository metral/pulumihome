# Pulumi Terminology

#### Deployment time API, runtime API
In a Pulumi program, the code to provision infrastructure runs in a separate "phase" that happens when you deploy the application. Whenever there is inline code in a `cloud.Function` or `cloud.Task`, that is code that executes in the running cloud program. Other code that creates new resources is a *deployment time* operation, such as provisioning a `Task` or `HttpEndpoint`. (TODO: fix up once the cloud.Function part is decided.)