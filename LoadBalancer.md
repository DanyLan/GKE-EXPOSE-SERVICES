# Loadbalancer

A LoadBalancer service is the standard way to expose a service to the internet. On GKE, a Google Cloud controller will configure a [network load balancer](https://cloud.google.com/load-balancing/docs/network/) that will provide you with a single IP address that will forward all traffic to your service.

![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/Loadbalancer.png)
