# ExternalName

Mapping external services to internal ones gives you the ability to bring those services into the your cluster.

If the external service has a valid domain name and you do not need port remapping , then using the [`ExternalName`](https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps#creating_a_service_of_type_externalname) service type is a good solution. If you do not have a domain name, simply assign the IP address to an endpoint and use that instead. I will show case an example below

1. Create your cluster
2. Create a VM and get the internal IP and run a http service this way:

`python -m SimpleHTTPServer 27017`
   
![](https://github.com/DanyLan/GKE-EXPOSE-SERVICES/blob/master/internalIP.png)

3. Now that we have the IP address, let us create the service

`kind: Service
apiVersion: v1
metadata:
 name: mongo
spec:
 type: ClusterIP
 ports:
 - port: 27017
   targetPort: 27017`
