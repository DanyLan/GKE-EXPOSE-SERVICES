# Loadbalancer

A LoadBalancer service is the standard way to expose a service to the internet. On GKE, a Google Cloud controller will configure a [network load balancer](https://cloud.google.com/load-balancing/docs/network/) that will provide you with a single IP address that will forward all traffic to your service. This means you can send almost any kind of traffic to it, like HTTP, TCP, UDP, Websockets, gRPC, and so on.

The downside is that you can only expose one service per LoadBalancer, and you have to pay for a LoadBalancer for each service you want to advertise. In order to by-pass that enter the concept of Ingress.

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
    
Now exec into a pod and test locally

    kubectl get pods
    NAME                                   READY   STATUS    RESTARTS   AGE
    my-deployment-50001-6cc5b9f4d5-9sp8r   1/1     Running   0          23s
    my-deployment-50001-6cc5b9f4d5-fpg2x   1/1     Running   0          23s
    my-deployment-50001-6cc5b9f4d5-s22qx   1/1     Running   0          23s

    kubectl exec -it my-deployment-50001-6cc5b9f4d5-9sp8r -- sh

    apk add --no-cache nmap
    apk add --no-cache curl
    nmap localhost
    curl -v localhost:50001

Here is a manifest for a Service of type LoadBalancer:

    apiVersion: v1
    kind: Service
    metadata:
      name: my-lb-service
    spec:
      type: LoadBalancer
      selector:
        app: products
        department: sales
      ports:
      - protocol: TCP
        port: 60000
        targetPort: 50001

    kubectl apply -f my-lb-service.yaml
    
View the service:

    kubectl get svc

    NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)           AGE           
    my-lb-service   LoadBalancer   10.12.15.190   35.226.54.126   60000:31800/TCP   2m10s
    
Access your service

In your browser enter `http://35.226.54.126:60000` where 35.226.54.126 if the external IP address of the loadbalancer. 




