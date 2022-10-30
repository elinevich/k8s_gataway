# Kubernetes API Gateway in the GKE Gateway controller and the Contour implementations


* [What is the Gateway API?](#what-is-the-gateway-api)
* [What are the differences between Ingress and Gateway API?](#what-are-the-differences-between-ingress-and-gateway-api)
* [Gateway API resources](#gateway-api-resources)
* [Deploying the demo with the Contour Gateway API](#deploying-the-demo-with-the-contour-gateway-api)
* [Deploying the demo with the GKE Gateway API](#deploying-the-demo-with-the-gke-gateway-api)
* [Which API Gateway should be used?](#which-api-gateway-should-be-used)
* [Sources](#sources)

## What is the Gateway API?

Gateway API is an open source project managed by the SIG-NETWORK community. The Gateway API evolves the Ingress resource and improves it.

Gateway and Ingress are both open source standards for routing traffic.
Gateway is not a Ingress replacement, Gateway is an evolution of Ingress that provides the same function, delivered as a superset of the Ingress capabilities. 

## What are the differences between Ingress and Gateway API?

Ingress primarily targets exposing HTTP applications with a simple, declarative syntax. 
Gateway API supports for many commonly used networking protocols (e.g. HTTP, TLS, TCP, UDP) as well as tightly integrated support for Transport Layer Security (TLS).
Unlike Ingress, Gateway API supports cross-namespace routing.

## Gateway API resources

- GatewayClass
- Gateway
- Route Resources

![Текст с описанием картинки](/assets/img/diagram.png)

### GatewayClass
A GatewayClass is a resource that defines a template for TCP/UDP (level 4) load balancers and HTTP(S) (level 7) load balancers in a Kubernetes cluster. 
Defines a cluster-scoped resource that's a template for creating load balancers in a cluster.

### Gateway
Defines where and how the load balancers listen for traffic. Cluster operators create Gateways in their clusters based on a GatewayClass.

### Route Resources
Route resources define protocol-specific rules for mapping requests from a Gateway to Kubernetes Services. It includes resources like HTTPRoute, TLSRoute, TCPRoute, UDPRoute, GRPCRoute.

The Gateway API is supported by many projects. But in this article I will show, how to deploy Kubernetes API Gataway resources, using the Contour and the Google Kubernetes Engine implementations and integrators.

## Prerequisites
The following prerequisites must be met before using Gateway API:

- a kubernetes cluster
- the kubectl command-line tool

> Note! The Contour supports different clusters (refer to [the compatibility matrix](https://projectcontour.io/resources/compatibility-matrix/) for cluster version requirements). Unlike the Contour the GKE Gateway works with GKE version 1.20 or later and supported by [VPC-native (Alias IP)](https://cloud.google.com/kubernetes-engine/docs/concepts/alias-ips) clusters only.

## Deploying the demo with the Contour Gateway API
1. Go to the contour directory:
```
cd contour
```

2. Create Gateway API CRDs:
```
kubectl apply -f https://raw.githubusercontent.com/projectcontour/contour/v1.23.0/examples/gateway/00-crds.yaml
```
3. Create a GatewayClass:
```
kubectl apply -f gatewayClass.yaml
```
4. Create a Gateway in the projectcontour namespace:
```
kubectl apply -f gateway.yaml
```
5. Deploy Contour:
```
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Сreated with this command:

- namespace projectcontour to run Contour
- contour CRDs
- contour RBAC resources
- contour Deployment / Service
- envoy DaemonSet / Service
- contour ConfigMap

6. Update the Contour config map to enable Gateway API processing by specifying a gateway controller name, and restart Contour to pick up the config change:

```
kubectl apply -f configMap.yaml
kubectl -n projectcontour rollout restart deployment/contour
```

7. Deploy the test application:

```
kubectl apply -f https://raw.githubusercontent.com/projectcontour/contour/v1.23.0/examples/example-workload/gatewayapi/kuard/kuard.yaml
```
The kuard resources were created with this command in the default namespace:

- deployment to run kuard as the test application.
- service to expose the kuard application on TCP port 80.
- HTTPRoute, attached to the contour Gateway, to route requests for local.projectcontour.io to the kuard service.

7. Verify that the kuard resources are available:

```
kubectl get po,svc,httproute -l app=kuard
```

```
NAME                         READY   STATUS    RESTARTS   AGE
pod/kuard-75bff7b748-8gvdk   1/1     Running   0          24m
pod/kuard-75bff7b748-l4g6d   1/1     Running   0          24m
pod/kuard-75bff7b748-nhccq   1/1     Running   0          24m

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kuard   ClusterIP   10.245.133.50   <none>        80/TCP    24m

NAME                                        HOSTNAMES                     AGE
httproute.gateway.networking.k8s.io/kuard   ["local.projectcontour.io"]   22s
```

8. Testing the Gateway API

```
$ kubectl -n projectcontour port-forward service/envoy 8888:80
```

In another terminal make a request to the application via the forwarded port (note, local.projectcontour.io is a public DNS record resolving to 127.0.0.1 to make use of the forwarded port):

```
$ curl -i http://local.projectcontour.io:8888
```

You will receive a 200 response code along with the HTML body of the main kuard page.

You can also open http://local.projectcontour.io:8888/ in a browser.


## Deploying the demo with the GKE Gateway API
1. Go to the gke directory:
```
cd gke
```

2. Create Gateway API CRDs:

```
kubectl apply -k "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.0"
```
This command installs the v1beta1 CRDs:

```
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
```

3. Create the Gateway in your cluster:
```
kubectl apply -f gateway.yaml
```
4. Validate that the Gateway has deployed correctly:

```
kubectl describe gateways.gateway.networking.k8s.io external-http
```

5. Deploying the demo applications

```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/gke-networking-recipes/main/gateway/gke-gateway-controller/app/store.yaml
```
Сreated with this command:

- store-v1 Deployment / Service
- store-v2 Deployment / Service
- store-german Deployment / Service

6. Verify again, that resources are available:

```
kubectl get svc
```
```
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
store-german   ClusterIP   10.245.225.125   <none>        8080/TCP   71m
store-v1       ClusterIP   10.245.227.191   <none>        8080/TCP   71m
store-v2       ClusterIP   10.245.232.108   <none>        8080/TCP   71m
```

```
kubectl get pods
```
```
NAME                        READY   STATUS    RESTARTS   AGE
store-german-66dcb75977-5gr2n   1/1     Running   0          20m
store-v1-65b47557df-jkjbm       1/1     Running   0          20m
store-v2-6856f59f7f-sq889       1/1     Running   0          20m
```

7. Deploy the HTTProute in your cluster:
```
kubectl apply -f route.yaml
```
8. Validate that the store HTTPRoute has been applied successfully:

```
kubectl describe httproute.gateway.networking.k8s.io store
```

These routing rules will process HTTP traffic in the following manner:

![Текст с описанием картинки](/assets/img/routes.png)

8. Testing

Retrieve the IP address from the Gateway to send traffic to your application:

```
kubectl get gateways.gateway.networking.k8s.io external-http -o=jsonpath="{.status.addresses[0].value}"
```
or get the IP address of the Gateway by looking at the output of

```
kubectl describe gateway external-http
```

Make a request


```
curl -H "host: store.example.com" IP
```

Replace IP with the IP address from the previous step.


## Which API Gateway should be used?
The GKE Gateway controller is Google’s implementation of the Kubernetes Gateway API. In this way it has to integrate with GCP’s features such as Cloud Load Balancing, Network Endpoint Groups (NEGs), etc.

The GKE GatewayClasses support different [capabilities](https://cloud.google.com/kubernetes-engine/docs/how-to/gatewayclass-capabilities) depending on their underlying load balancer.

The GKE Gateway controller only supports GatewayClasses, Gateways, and HTTPRoutes. TCPRoutes, UDPRoutes, and TLSRoutes are not supported.
The Contour supports the same resources, but includes TLSRoutes.

As mentioned before, the Contour works with any clusters, but the GKE Gateway supports Google Kubernetes engine and VPC-native clusters only.

## Sources

This article was created using the following sources:
[Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/),
[Contour Gateway API](https://projectcontour.io/guides/gateway-api/),
[GKE Gateway API](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api).