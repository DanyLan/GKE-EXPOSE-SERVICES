# Loadbalancer

A LoadBalancer service is the standard way to expose a service to the internet. On GKE, a Google Cloud controller will configure a [network load balancer](https://cloud.google.com/load-balancing/docs/network/) that will provide you with a single IP address that will forward all traffic to your service.

![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/Loadbalancer.png)

# Creating a Service of type LoadBalancer

Here is a manifest for a deployment

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment-50001
    spec:
      selector:
        matchLabels:
          app: products
          department: sales
      replicas: 3
      template:
        metadata:
          labels:
            app: products
            department: sales
        spec:
          containers:
          - name: hello
            image: "gcr.io/google-samples/hello-app:2.0"
            env:
            - name: "PORT"
              value: "50001"
           
Notice the containers are listening on port 50001
              
    kubectl apply -f my-deployment-50001.yaml
    

