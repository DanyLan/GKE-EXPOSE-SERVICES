# GKE-EXPOSE-SERVICES

After building you application, there are five ways to expose it through the concept of a [Service](https://kubernetes.io/docs/concepts/services-networking/service/). The idea of a service is to provide a stable virtual IP to a group of endpoints (usually pods). The set pods behind a service can change however clients only need the VIP, which does not change.
The five types of Services are

- [ClusterIP (default)](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/ClusterIP.md)
- [NodePort](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/NodePort.md)
- [LoadBalancer](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/LoadBalancer.md)
- [ExternalName](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/ExternalServices.md)
- [Headless](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/Headless.md)

# Differences

ClusterIP: Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster. This is the default ServiceType

NodePort: Exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>.

LoadBalancer: Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, to which the external load balancer will route, are automatically created.


A ClusterIP exposes the following:

- spec.clusterIp:spec.ports[*].port

You can only access this service while inside the cluster. It is accessible from its spec.clusterIp port. If a spec.ports[*].targetPort is set it will route from the port to the targetPort.

A NodePort exposes the following:

- [NodeIP]:spec.ports[*].nodePort
- spec.clusterIp:spec.ports[*].port
  
If you access this service on a nodePort from the node's external IP, it will route the request to spec.clusterIp:spec.ports[*].port, which will in turn route it to your spec.ports[*].targetPort, if set

A LoadBalancer exposes the following:

- spec.loadBalancerIp:spec.ports[*].port
- [NodeIP]:spec.ports[*].nodePort
- spec.clusterIp:spec.ports[*].port
  
You can access this service from your load balancer's IP address, which routes your request to a nodePort, which in turn routes the request to the clusterIP port. You can access this service as you would a NodePort or a ClusterIP service as well.
