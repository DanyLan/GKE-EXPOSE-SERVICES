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
           

Copy manifest to a file name `my-deployment.yaml` and create the deployment:

    kubectl apply -f my-deployment.yaml
    
Here is a manifest for a Service of type NodePort:

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
        port: 8080
        
Copy the manifest to a file named my-np-service.yaml, and create the Service:
 
     kubectl apply -f my-np-service.yaml
     
Now exec into a pod and test locally
 
    kubectl get pods
    NAME                                   READY   STATUS    RESTARTS   AGE
    my-deployment-50000-86c6fd8fbd-h54db   1/1     Running   0          24m
    my-deployment-50000-86c6fd8fbd-lk4qw   1/1     Running   0          24m
    my-deployment-50000-86c6fd8fbd-xwwpq   1/1     Running   0          24m
    
    kubectl exec -it my-deployment-50000-775f57d44c-27x4g -- sh
    
    apk add --no-cache nmap
    apk add --no-cache curl
    nmap localhost
    curl -v localhost:8080
    
You will notice that the port the pod is listening on is the targetPort 8080 after doing the `nmap localhost`. We mentioned above that that NodePort is used to expose your service externally. In order to view the NodePort [31066] that was assigned, you may run the following command

    kubectl get svc
    kubernetes      ClusterIP   10.12.0.1      <none>        443/TCP          10d
    my-np-service   NodePort    10.12.10.230   <none>        8080:31066/TCP   72s
    
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/port-nodeport-target.png)
 
Keep in mind that creating a Service of type NodePort also exposes your service with a ClusterIP as shown below

    kubectl get service my-np-service --output yaml

    ...
    spec:
      clusterIP: 10.12.10.230
      externalTrafficPolicy: Cluster
      ports:
      - nodePort: 31066
        port: 8080
        protocol: TCP
        targetPort: 8080
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
    my-deployment-50000-86c6fd8fbd-h54db   1/1     Running   0          3m54s   10.8.1.18   gke-standard-cluster-1-default-pool-f3abacde-rmdb   <none>           <none>
    my-deployment-50000-86c6fd8fbd-lk4qw   1/1     Running   0          3m54s   10.8.1.17   gke-standard-cluster-1-default-pool-f3abacde-rmdb   <none>           <none>
    my-deployment-50000-86c6fd8fbd-xwwpq   1/1     Running   0          3m54s   10.8.0.17   gke-standard-cluster-1-default-pool-f3abacde-b0hx   <none>           <none>none>
    
Pod `my-deployment-50000-86c6fd8fbd-h54db` is located at node node `gke-standard-cluster-1-default-pool-f3abacde-rmdb` with external IP `35.224.50.144`

Now to expose and test externally, create a firewall rule to allow TCP traffic on the node port:

    gcloud compute firewall-rules create test-node-port --allow tcp:31066
    
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/testnodeport.png)

Access your service this way

In browser enter [NODE_IP_ADDRESS]:[Node_Port]

`http://35.224.50.144:31066/`

    Hello, world!
    Version: 2.0.0
    Hostname: my-deployment-50000-86c6fd8fbd-lk4qw

# Port, TargetPort and NodePort

A successful request can be made from outside the cluster to the node’s IP address and service’s `nodePort`, forwarded to the service’s `port`, and received on the `targetPort` by the pod.

    kind: Service
    apiVersion: v1
    metadata:
      name: port-example-svc
    spec:
      # Make the service externally visible via the node
      type: NodePort 

      ports:
        # Which port on the node is the service available through?
        - nodePort: 31234

        # Inside the cluster, what port does the service expose?
        - port: 8080

        # Which port do pods selected by this service expose?
        - targetPort: 

      selector:
        # ...

nodePort
This setting makes the service visible outside the Kubernetes cluster by the node’s IP address and the port number declared in this property. The service also has to be of type NodePort (if this field isn’t specified, Kubernetes will allocate a node port automatically).

port
Expose the service on the specified port internally within the cluster. That is, the service becomes visible on this port, and will send requests made to this port to the pods selected by the service.

targetPort
This is the port on the pod that the request gets sent to. Your application needs to be listening for network requests on this port for the service to work.
