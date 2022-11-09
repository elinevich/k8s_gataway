# The Kubernetes API Gateway: Ingress replacement or evolution?
## Deploy Gateway API with GKE, Contour and NGINX implementations

## Theses

1. Introdaction: answering the question what is the Gateway API? Will it replace the Ingress?

### What is the Gateway API? Will it replace the Ingress?

The Gateway API is an open source project managed by the SIG-NETWORK community, which provides a collection of resources that model service networking in Kubernetes. 

The Gateway is not a Ingress replacement, it is an evolution of Ingress that provides the same function, delivered as a superset improved of the Ingress capabilities.


2. Reveal the problems of Ingress.

### What problems does the Ingress have?

2.1 Ingress is supporting only one user role – the Kubernetes administrator or operator, who manages the configuration.

2.2. the proliferation of annotations and custom resource definitions (CRDs) in many Ingress implementations, where they unlock the capabilities of different data planes and implement features that aren’t built into the Ingress resource, like header‑based matching, traffic weighting, and multi‑protocol support. Gateway API delivers such capabilities as part of the core API standard.

3. Reveal the advantages of Gateway API.

### What advantages does the Gateway API have over the Ingress?

3.1. Describe what cross namespace routing is and how it works in the Gateway API.

### Cross namespace routing
> Cross namespace routing allows user access control to be applied differently across namespaces for Routes and Gateways, dividing access and control to different parts of the cluster routing configuration.

The Ingress demands that resources need to be in the same namespace. It’s not a big deal, if your cluster is managed by one team, but one-to one relationship can be a problem, if you are sharing cluster between a few teams.

Unlike the Ingress the Kubernetes Gateway API provides cross namespace routing, that allows to use one-to-many relationship between the Gateway and Route and deploy services to the different namespaces.

3.2. Describe what routing is and how it works in the Gateway API.

### Routing
> Routing allows to match on HTTP traffic and direct it to Kubernetes backends.

One of the most important features missing in Ingress is advanced traffic routing. Up until now, this was resolved by way of a service mesh, which made it complex and tightly coupled with the mesh implementation.

You can implement numerous protocols with Gateway API, including support for TCPRoute, HTTPRoute and GRPCRoute.

> Note! The GRPCRoute and TCPRoute resources included in the "Experimental" channel of Gateway API.

3.3. Describe what HTTP path redirects and rewrites are and how it works in the Gateway API.

### HTTP path redirects and rewrites
> A redirects allow to give more than one URL address to a page, a rewrites - to completely separate the URL from the resource. 

This is a necessary and powerful thing, which is available in Ingress only through annotations.
Filters to path redirects and rewrites became available with v1beta1 version of the Gateway API, but still in experimental mode.

3.4. Describe what traffic splitting is and how it works in the Gateway API.

### Traffic splitting
> Traffic splitting allows to specify weights to shift traffic between different backends, which you can ombine with A/B or canary strategies to achieve complex rollouts in a simple way.

The Gateway API supports typed Route resources and typed backends. In this way it is possible to create a flexible API, supporting various protocols (HTTP, GRPC) and different backend (Kubernetes Services, storage buckets, or functions).

3.5 Describe what TLS is and how it works in the Gateway API.

### TLS
> TLS is the cryptographic protocol that powers encryption for many network applications. 

The Gateway API supports TLS configuration at various points in the network path between the client and service — for upstream and downstream independently. Depending on the listener configuration, various TLS modes and route types are possible. The cert-manager supporting is also available.

You also can configure the Gateway to reference a certificate in a different namespaces.

4. Describe the resources of Gateway API.
### Gateway API resources

- GatewayClass
- Gateway
- Route Resources

![how gateway works](/assets/img/diagram.png)

4.1. Define the GatewayClass

### GatewayClass

A GatewayClass is a resource that defines a template for TCP/UDP (level 4) load balancers and HTTP(S) (level 7) load balancers in a Kubernetes cluster. 
Defines a cluster-scoped resource that's a template for creating load balancers in a cluster.
4.2. Define the Gateway

### Gateway

Defines where and how the load balancers listen for traffic. Cluster operators create Gateways in their clusters based on a GatewayClass.
4.3. Define the Route Resources

### Route Resources

Route resources define protocol-specific rules for mapping requests from a Gateway to Kubernetes Services. It includes resources like HTTPRoute, TLSRoute, TCPRoute, UDPRoute, GRPCRoute.

5. Еhis is practise part, which shows how to deploy Gateway API, using Contour, GKE and NGINX implementations.

The Gateway API is supported by many projects. But in this article I will show, how to deploy Kubernetes API Gataway resources, using  implementations and integrators of the Contour, the NGINX Kubernetes Gateway and recently released - the Google Kubernetes Engine.
5.1 Deploying the demo with the Contour Gateway API

```
CODE PART
```

5.2 Deploying the demo with the GKE Gateway API

```
CODE PART
```

5.3 Deploying the demo with the NGINX Kubernetes Gateway

```
CODE PART
```

6. This part contains the main conclusions.

### Сonclusions

The Gateway API is the newest way to expose Kubernetes API, positioned as role-oriented, portable, expressive and extensible standart for developing.

Using advantages of the API, it became possible to create flexible and portable applications not only for the end user, but also for the multiple team of managers, developers and administarors.

Despite the advatages of the the Gateway API, an important features like request redirect and rewrite are in experimental mode. Of course, it is very promising and has a great impact in the future, but we recommend waiting for a more stable implementation of some features, but for now pay your attention to the ingress controller. For example, [NGINX Ingress Controller](https://www.nginx.com/products/nginx-ingress-controller/) can be a good solution to the problem with redirect and rewrite.

And notice that there is no good or bad technology, picking the right tool for theapplication based on where and how you are going to use it. 


8. This part contains the sources, used to write the article

### Sources

This article was created using the following sources:
[Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/),
[Contour Gateway API](https://projectcontour.io/guides/gateway-api/),
[GKE Gateway API](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api).
[NGINX Kubernetes gateway](https://github.com/nginxinc/nginx-kubernetes-gateway).
