# ExternalName

Mapping external services to internal ones gives you the ability to bring those services into the your cluster.

If the external service has a valid domain name and you do not need port remapping , then using the [`ExternalName`](https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps#creating_a_service_of_type_externalname) service type is a good solution. If you do not have a domain name, simply assign the IP address to an endpoint and use that instead. I will show case an example below

1. Create your cluster
2. Create a VM and get the internal IP and run a http service this way:

       python -m SimpleHTTPServer 27017
   
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/internalIP.png)

3. Now that we have the IP address, let us create the service with `kubectl create -f mongoservice.yaml`

       kind: Service
       apiVersion: v1
       metadata:
        name: mongo
       spec:
        type: ClusterIP
        ports:
        - port: 27017
          targetPort: 27017
          
4. At this point, the service does not know where to send traffic to, so let us create an Endpoint that will receive traffic from the aforementioned service `kubectl create -f mongoendpoint.yaml` 
          
       kind: Endpoints
       apiVersion: v1
       metadata:
        name: mongo
       subsets:
        - addresses:
            - ip: 10.128.15.234
          ports:
            - port: 27017
          
To view the endpoint

    kubectl get ep

    NAME         ENDPOINTS             AGE
    mongo        10.128.15.234:27017   18s

Now to test if `mongo` can reach the server at 10.128.15.234, create an nginx pod and run `wget mongo:27017`

    kubectl run nginx --image=nginx
    kubectl get pods
    kubectl exec -it nginx-dbddb74b8-6slmz -- sh
    
Install wget

    apt-get update
    apt-get install wget
    
Inside nginx pod expected output is

    --2019-09-04 17:28:33--  http://mongo:27017/
    Resolving mongo (mongo)... 10.12.2.169
    Connecting to mongo (mongo)|10.12.2.169|:27017... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 322 [text/html]
    Saving to: 'index.html.2'

    index.html.2                                                100%[=========================================================================================================================================>]     322  --.-KB/s    in 0s      

    2019-09-04 17:28:33 (25.8 MB/s) - 'index.html.2' saved [322/322]






 
 
