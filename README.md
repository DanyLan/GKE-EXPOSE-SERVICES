# GKE-EXPOSE-SERVICES

After building you application, there are five ways to expose it through the concept of a [Service](https://kubernetes.io/docs/concepts/services-networking/service/). The idea of a service is to provide a stable virtual IP to a group of endpoints (usually pods). The set pods "behind a service can change however clients only need the VIP, which does not change.
The five types of Services are

- [ClusterIP (default)](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/ClusterIP.md)
- NodePort
- LoadBalancer
- ExternalName
- Headless



https://stackoverflow.com/questions/41509439/whats-the-difference-between-clusterip-nodeport-and-loadbalancer-service-types
