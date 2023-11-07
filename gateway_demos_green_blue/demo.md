# Scaling with Istio: What You Need to Know

## Introduction


Istio is a robust open-source platform tailored to offer a plethora of features essential for efficiently connecting, securing, controlling, and observing microservices. As the scale of microservices expands, the complexities of managing traffic, reinforcing security, and assuring high availability escalate. Istio, being a cornerstone of our tech stack, is meticulously crafted to tackle these burgeoning challenges head-on. At Cloud Café, we also have a burgeoning interest in the Cilium solution. We're gearing up to experiment with it in the near future. Given that the Kubernetes Gateway API is both portable and standardized, transitioning to a different class for our gateway promises to be seamless.

## The Role of Istio in Our Tech Stack

In our stack, Istio serves as a control plane that configures the sidecar proxies deployed alongside our application's microservices. The key roles of Istio in our tech stack include:

- **Traffic Routing and Management:** Dynamic request routing and A/B testing.
- **Security:** Automatically encrypting data, and providing powerful authentication and authorization features.
- **Observability:** Providing insights into performance and service dependencies.

## To start create your demo apps
The service A is a simple python application 

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    banner = (
        "*********************************\n"
        "*                               *\n"
        "*        ---> Service A <---    *\n"
        "*                               *\n"
        "*********************************\n"
    )
    return banner

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)

```

Build it and do the same thing for the service B

```Dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py app.py

CMD ["python", "app.py"]
```




## How Istio Helps in Scaling

### Creating an Istio Gateway

Firstly, let's deploy the Istio Gateway:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: demo-gateway
  namespace: testistio
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "test.example.io"
```


### Deploy a Service (Service A)

Deploy a sample microservice, let's call this `Service A`:

```bash
kubectl apply -f  service-a.yaml

```


### Test Service Accessibility (Service A)

To test if `Service A` is accessible:

```bash
curl http://test.example.io
```

You should have the following response:

```bash
curl -v  test.example.io
> GET / HTTP/1.1
> Host: test.example.io
> User-Agent: curl/7.87.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 170
< server: istio-envoy
< date: Sat, 21 Oct 2023 11:28:48 GMT
< x-envoy-upstream-service-time: 7
< 
*********************************
*                               *
*        ---> Service A <---    *
*                               *
*********************************
```


### Deploy Another Service (Service B)

Deploy another microservice, let's call this `Service B`:

```bash
kubectl apply -f  service-b.yaml
```

### Test Service Accessibility (Service B)
```bash
curl -v  test.example.io
> GET / HTTP/1.1
> Host: test.example.io
> User-Agent: curl/7.87.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-type: text/html; charset=utf-8
< content-length: 170
< server: istio-envoy
< date: Sat, 21 Oct 2023 11:33:34 GMT
< x-envoy-upstream-service-time: 7
< 
*********************************
*                               *
*        ---> Service B <---    *
*                               *
*********************************
```

### Canary Deployment (A/B Testing)

In this section, we'll modify our VirtualService to route 90% of traffic to `Service A` and 10% to `Service B`:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-virtual-service
spec:
  hosts:
  - "test.example.io"
  gateways:
  - demo-gateway
  http:
  - route:
    - destination:
        host: service-a
      weight: 90
    - destination:
        host: service-b
      weight: 10
```

### Demonstrating Canary with a Script

Create a script to make multiple calls to the services and observe the traffic distribution:

```bash
for i in {1..100}; do curl -Lv test.example.io; done
```

### Blue/Green Deployment

To test a Blue/Green deployment, you can update the VirtualService to route traffic exclusively to `Service A` or `Service B`.

### Demonstrating Blue/Green with a Script

Similar to the Canary script, this will help you verify that 100% of traffic is routed to either `Service A` or `Service B`.

```bash
for i in {1..100}; do curl -Lv test.example.io; done
```



## User-Specific Routing
Extend your Canary deployment to consider user groups. You could direct all internal users to the new Service B while serving the original Service A to external users.


First of all delete the previous VirtualService :

```bash
kubectl delete -f canary-virtualservice.yaml
```
Deploy a new VirtualService: 

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-virtual-service
spec:
  hosts:
  - "test.example.io"
  gateways:
  - demo-gateway
  http:
  - match:
    - headers:
        user-group:
          exact: "internal"
    route:
    - destination:
        host: service-b
      weight: 100
  - match:
    - headers:
        user-group:
          exact: "external"
    route:
    - destination:
        host: service-a
      weight: 100
  - route:  # Default route if no header is matched
    - destination:
        host: service-a
      weight: 90
    - destination:
        host: service-b
      weight: 10
```

In this example:

- Traffic with a header `user-group: internal` will be routed 100% to `service-b`.
- Traffic with a header `user-group: external` will be routed 100% to `service-a`.
- Traffic without any of these specific headers or other values will be distributed 90% to `Service A` and 10% to `Service B`, just like in your initial Canary setup.

To test these routing conditions, you can use `curl` like so:

```bash
curl -H "user-group: internal" http://test.example.io
curl -H "user-group: external" http://test.example.io
curl http://test.example.io  # For default routing
```

This approach allows you to selectively route traffic based on user group criteria, providing more flexibility in your Canary deployment strategy.


## Wrapping Up

Istio is a powerful tool for managing microservices at scale. It offers a feature-rich platform that includes powerful traffic management rules, security features, and observability tools. Through its robust features, Istio plays a vital role in scaling our applications, thus being an essential part of our technology stack.