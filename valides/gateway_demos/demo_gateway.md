
Firstly, let's create a namespace with Istio injection enabled:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-istio
  labels:
    istio-injection: enabled
```

Create the gateway :
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: demo-gateway
  namespace: test-istio
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "test.cloudeefy.io"
```


Now, let's deploy the Virtual service:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: http-route
  namespace: test-istio
spec:
  hosts:
  - "test.cloudeefy.io"
  gateways:
  - demo-gateway
  http:
  - match:
    - uri:
        prefix: /hello
    route:
    - destination:
        host: hello-service
        port:
          number: 80
```

Finally, the Nginx deployment and service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-service
  namespace: test_istio
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-service
  template:
    metadata:
      labels:
        app: hello-service
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: test-istio
spec:
  selector:
    app: hello-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Here's what each YAML does:

1. The `Namespace` YAML creates a namespace called `test_istio` and enables Istio sidecar injection for this namespace.
2. The `Gateway` YAML sets up a basic HTTP gateway.
3. The `VirtualService` YAML routes traffic for `test.cloudeefy.io` and forwards requests going to `/hello` to a service named `hello-service`.
4. The `Deployment` YAML deploys a single pod of Nginx.
5. The `Service` YAML creates a Kubernetes service named `hello-service` that exposes the Nginx deployment.

You can save these YAMLs into separate files or combine them into one single file separated by `---`, and then run:

```bash
kubectl apply -f <filename>.yaml
```

This will create all these resources in your Kubernetes cluster. Make sure Istio is properly installed and configured in your cluster to make full use of these features.