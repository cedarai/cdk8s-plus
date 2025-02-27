# Ingress

[Ingress] manages external access to services in a cluster, typically through
HTTP. Ingress may provide load balancing, SSL termination and name-based virtual
hosting.

!!! tip ""
    [API Reference](../../reference/cdk8s-plus-27/typescript.md#ingressv1beta1)

You must have an [Ingress controller] to satisfy an Ingress. Only creating an
Ingress resource has no effect.

[Ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[Ingress controller]: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers

The following example will route HTTP requests sent to the `/hello` url prefix
to a service associated with a deployment of the
[hashicorp/http-echo](https://github.com/hashicorp/http-echo) image.

```typescript
import * as kplus from 'cdk8s-plus-27';
import { Construct } from 'constructs';
import { App, Chart, ChartProps } from 'cdk8s';

export class MyChart extends Chart {
  constructor(scope: Construct, id: string, props: ChartProps = { }) {
    super(scope, id, props);

    const helloDeployment = new kplus.Deployment(this, "Deployment", {
      containers: [
        {
          image: 'hashicorp/http-echo',
          args: [ '-text', 'hello ingress' ],
          portNumber: 5678,
        }
      ]
    });

    const helloService = helloDeployment.exposeViaService({
        ports: [
          {
            port: 5678,
          }
        ]
      });

    const ingress = new kplus.Ingress(this, 'ingress');
    ingress.addRule('/hello', kplus.IngressBackend.fromService(helloService));
 }
}

const app = new App();
new MyChart(app, 'ingress');
app.synth();
```

You can use `addHostRule(host, path, backend)` to define a route that will only
apply to requests with this `Host` header. This can be used to implement virtual
hosts.

The `addDefaultBackend(backend)` and `addHostDefaultBackend(host, backend)`
methods can be used to define backends that will accept all requests that do not
match any other rules.

The TCP port used to route requests to services will be determined based on
which ports are exposed by the service (e.g. through `serve()`). If the service
exposes multiple ports, then a port must be specified via
`IngressBackend.fromService(service, { port })`.
