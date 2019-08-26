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

Exec in your pods this way

    kubectl exec -it nginx-dbddb74b8-6slmz -- sh

and install packages and test/curl locally

    apt-get update
    apt-get install curl
    apt-get install nmap
    nmap localhost
    curl -v localhost
    





