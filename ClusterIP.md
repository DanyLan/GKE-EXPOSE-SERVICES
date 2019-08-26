# ClusterIP

A ClusterIP service is the default Kubernetes service. It gives you a service inside your cluster that other apps inside the cluster can communicate. Keep in mind that there is no external access.

We are going to create two deployments and check communication between them.

# Deployment 1

Here is the manifest for deployment 1 name my-deployment

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      selector:
        matchLabels:
          app: metrics
          department: sales
      replicas: 3
      template:
        metadata:
          labels:
            app: metrics
            department: sales
        spec:
          containers:
          - name: hello
            image: "gcr.io/google-samples/hello-app:2.0"

Create your file

`nano my-deployment.yaml`

Create your deployment 

`kubectl create -f my-deployment.yaml`

Check created pods

`kubectl get pods`

    my-deployment-7bc95fb476-2nl55   1/1     Running   0          19s
    my-deployment-7bc95fb476-mzqrp   1/1     Running   0          19s
    my-deployment-7bc95fb476-scw9s   1/1     Running   0          19s
    
# Deployment 2

kubectl run nginx --image=nginx

Check nginx pods

`kubectl get pods`

    my-deployment-7bc95fb476-425bv   1/1     Running   0          2d17h
    my-deployment-7bc95fb476-45mkx   1/1     Running   0          2d17h
    my-deployment-7bc95fb476-s4t44   1/1     Running   0          2d17h
    nginx-dbddb74b8-6slmz            1/1     Running   0          7s

Now we will check whether the above pods are working locally before testing interpod communication.

Exec in the nginx pod this way

    kubectl exec -it nginx-dbddb74b8-6slmz -- sh

and install packages and test/curl locally

    apt-get update
    apt-get install curl
    apt-get install nmap
    nmap localhost
    curl -v localhost
   
Exec in the my-deployment pod this way

    kubectl exec -it my-deployment-7bc95fb476-s4t44 -- sh

and install packages and test/curl locally

    apk add --no-cache nmap
    apk add --no-cache curl
    nmap localhost
    curl -v localhost:8080
    
Note: You will notice that image "gcr.io/google-samples/hello-app:2.0" was defined with port 8080 after doing the nmap and doing a curl -v localhost will give you an error.

# Exposing using clusterIP

Through console, go to 

`Menu>Kubernetes Engine>Workloads>nginx>Actions>Expose>Service type: Cluster IP.`

Get the Cluster IP this way

    kubectl get svc
    kubernetes    ClusterIP   10.12.0.1     <none>        443/TCP   3d
    nginx-rwgnr   ClusterIP   10.12.7.114   <none>        80/TCP    3m41s
    
Curl from my-deployment pod

    curl 10.12.7.114

Through `kubectl apply`

Here is a manifest for a service of type ClusterIP:

    apiVersion: v1
    kind: Service
    metadata:
      name: my-cip-service
    spec:
      type: ClusterIP
      selector:
        app: metrics
        department: sales
      ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
    
 Get cluster-IP
 
     kubectl get svc
     kubernetes       ClusterIP   10.12.0.1     <none>        443/TCP   3d
     my-cip-service   ClusterIP   10.12.6.178   <none>        80/TCP    7s
     nginx-rwgnr      ClusterIP   10.12.7.114   <none>        80/TCP    13m

Curl from nginx pod

    curl -v 10.12.6.178

Note that that we did not use port 8080 as we have exposed port 80 in manifest file



