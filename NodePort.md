# NodePort

A NodePort service is the most primitive way to get external traffic directly to your service. A NodePort exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>.



![](https://www.edureka.co/community/?qa=blob&qa_blobid=5351364249810994154)

# Creating a Service of type NodePort

Here is the manifest for a deployment

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment-50000
    spec:
      selector:
        matchLabels:
          app: metrics
          department: engineering
      replicas: 3
      template:
        metadata:
          labels:
            app: metrics
            department: engineering
        spec:
          containers:
          - name: hello
            image: "gcr.io/google-samples/hello-app:2.0"
            env:
            - name: "PORT"
              value: "50000"

From the deployment, we are telling the container to listen on port 50000.

Copy manifest to a file name `my-deployment.yaml` and create the deployment:

    kubectl apply -f my-deployment.yaml
    
Here is a manifest for a Service of type NodePort whetr targetPort is 50000:

    apiVersion: v1
    kind: Service
    metadata:
      name: my-np-service
    spec:
      type: NodePort
      selector:
        app: metrics
        department: engineering
      ports:
      - protocol: TCP
        port: 80
        targetPort: 50000
        
Copy the manifest to a file named my-np-service.yaml, and create the Service:
 
     kubectl apply -f my-np-service.yaml
     
Now exec into a pod and test locally
 
    kubectl get pods
    NAME                                   READY   STATUS    RESTARTS   AGE
    my-deployment-50000-775f57d44c-27x4g   1/1     Running   0          23h
    my-deployment-50000-775f57d44c-29hlc   1/1     Running   0          23h
    my-deployment-50000-775f57d44c-cszjt   1/1     Running   0          23h
    
    kubectl exec -it my-deployment-50000-775f57d44c-27x4g -- sh
    
    apk add --no-cache nmap
    apk add --no-cache curl
    nmap localhost
    curl -v localhost:50000
    
You will notice that the port the pod is listening on is the targetPort 50000. We mentioned above that that NodePort is used to expose your service externally. In order to view the NodePort [30887] that was assigned, you may run the following command

    kubectl get svc
    kubernetes      ClusterIP   10.12.0.1     <none>        443/TCP        5d2h
    my-np-service   NodePort    10.12.1.114   <none>        80:30887/TCP   4h39m
    
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/Port-NodePort-TargetPort.png)
 
Keep in mind that creating a Service of type NodePort also exposes your service with a ClusterIP as shown below

    kubectl get service my-np-service --output yaml

    ...
    spec:
      clusterIP: 10.12.1.114
      externalTrafficPolicy: Cluster
      ports:
      - nodePort: 30887
        port: 80
        protocol: TCP
        targetPort: 50000
      selector:
        app: metrics
        department: engineering
      sessionAffinity: None
      type: NodePort
    status:
      loadBalancer: {}
      ...

Find the external IP address of your nodes

    kubectl get nodes -o wide
    NAME                                                STATUS   ROLES    AGE    VERSION          INTERNAL-IP     EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
    gke-standard-cluster-1-default-pool-f3abacde-b0hx   Ready    <none>   6d1h   v1.12.8-gke.10   10.128.15.229   35.194.41.135       Container-Optimized OS from Google   4.14.127+        docker://17.3.2
    gke-standard-cluster-1-default-pool-f3abacde-rmdb   Ready    <none>   6d1h   v1.12.8-gke.10   10.128.15.230   35.224.50.144       Container-Optimized OS from Google   4.14.127+        docker://17.3.2
 
Now check where the pods are located

    kubectl get pods -o wide
    
    NAME                                   READY   STATUS    RESTARTS   AGE   IP          NODE                                                NOMINATED NODE   READINESS GATES
    my-deployment-50000-775f57d44c-27x4g   1/1     Running   0          47h   10.8.1.10   gke-standard-cluster-1-default-pool-f3abacde-rmdb   <none>           <none>
    my-deployment-50000-775f57d44c-29hlc   1/1     Running   0          47h   10.8.1.8    gke-standard-cluster-1-default-pool-f3abacde-rmdb   <none>           <none>
    my-deployment-50000-775f57d44c-cszjt   1/1     Running   0          47h   10.8.1.9    gke-standard-cluster-1-default-pool-f3abacde-rmdb   <none>           <none>
    
They are all located on the same node `gke-standard-cluster-1-default-pool-f3abacde-rmdb` with external IP `35.224.50.144`

Now to expose and test externally, create a firewall rule to allow TCP traffic on the node port:

    gcloud compute firewall-rules create test-node-port --allow tcp:30877
    
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/test-node-port.png)
