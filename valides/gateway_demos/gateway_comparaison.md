# Kubernetes Gateway API Tutorial: Comparing Gateway to Ingress with Istio 

  

## Introduction 

  

The Kubernetes Gateway API is an evolution of the Kubernetes Ingress API, offering more expressive, flexible, and role-oriented network constructs. In this tutorial, we'll walk through different scenarios using the new features of the Kubernetes Gateway API and compare them with the traditional Ingress. We will use Istio as the GatewayClass for this tutorial. 

  

## Prerequisites 

  

- Kubernetes cluster up and running 

- Istio installed and configured 

- `kubectl` installed and configured 

- Basic understanding of Kubernetes Ingress and Istio 

  

## Scenarios 

  

### Scenario 1: Simple Routing 

  

#### Using Ingress 

  

In traditional Ingress, you might use a simple YAML file like this: 

  

```yaml 

apiVersion: networking.k8s.io/v1 

kind: Ingress 

metadata: 

  name: simple-ingress 

spec: 

  rules: 

  - http: 

      paths: 

      - path: /hello 

        pathType: Prefix 

        backend: 

          service: 

            name: hello-service 

            port: 

              number: 80 

``` 

  

#### Using Gateway API 

  

With Gateway, you can express the same thing with more clarity: 

  

```yaml 

apiVersion: networking.x-k8s.io/v1alpha1 

kind: Gateway 

metadata: 

  name: simple-gateway 

spec: 

  gatewayClassName: istio 

  listeners: 

  - protocol: HTTP 

    port: 80 

    routes: 

      kind: HTTPRoute 

      namespaces: 

        from: All 

``` 

  

```yaml 

apiVersion: networking.x-k8s.io/v1alpha1 

kind: HTTPRoute 

metadata: 

  name: http-route 

spec: 

  rules: 

  - matches: 

    - path: 

        type: Prefix 

        value: /hello 

    forwardTo: 

    - serviceName: hello-service 

      port: 80 

``` 

  

### Scenario 2: Host-based and Path-based Routing 

  

#### Using Ingress 

  

```yaml 

apiVersion: networking.k8s.io/v1 

kind: Ingress 

metadata: 

  name: host-path-ingress 

spec: 

  rules: 

  - host: example.com 

    http: 

      paths: 

      - path: /hello 

        pathType: Prefix 

        backend: 

          service: 

            name: hello-service 

            port: 

              number: 80 

``` 

  

#### Using Gateway API 

  

```yaml 

apiVersion: networking.x-k8s.io/v1alpha1 

kind: HTTPRoute 

metadata: 

  name: http-host-path-route 

spec: 

  hostnames: 

  - "example.com" 

  rules: 

  - matches: 

    - path: 

        type: Prefix 

        value: /hello 

    forwardTo: 

    - serviceName: hello-service 

      port: 80 

``` 

  

### Scenario 3: Traffic Splitting 

  

#### Using Ingress 

  

Traffic splitting is not native to Ingress and usually requires an ingress controller's custom resource. 

  

#### Using Gateway API 

  

With Gateway, traffic splitting is straightforward: 

  

```yaml 

apiVersion: networking.x-k8s.io/v1alpha1 

kind: HTTPRoute 

metadata: 

  name: http-traffic-split 

spec: 

  rules: 

  - forwardTo: 

    - serviceName: v1-service 

      port: 80 

      weight: 50 

    - serviceName: v2-service 

      port: 80 

      weight: 50 

``` 

  

## Conclusion 

  

In this tutorial, we've shown you how to use the Kubernetes Gateway API with Istio as a GatewayClass for different scenarios, including basic routing, host and path-based routing, and traffic splitting. The Gateway API provides more expressive and flexible constructs compared to the traditional Ingress, making it easier to manage complex routing configurations. 

  

Feel free to test these examples in your environment, and you'll discover the power and flexibility that comes with the Kubernetes Gateway API. 