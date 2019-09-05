# Headless

A headless service is where we set the clusterIP as `None`:

    apiVersion: v1
    kind: Service
    metadata:
      labels:
        name: mariadb
      name: mariadb
    spec:
      ports:
        - port: 3306
      clusterIP: None
      selector:
        name: mariadb
        
Check that mariadb service has no clusterIP this way:

    kubectl get svc
    NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
    mariadb       ClusterIP   None           <none>        3306/TCP    60s
    
The concept of headless service can be further explained in [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) which requires that particular type of service for network identity of the pods. Let us build a StatefulSet along with its service.

<pre>
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # Label selector that determines which Pods belong to the StatefulSet
                 # Must match spec: template: metadata: labels
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx # Pod template's label selector
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi</pre>
          
If you do a `kubectl get svc`, you will notice that there is no IP for clusterIP.

    kubectl get svc
    NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
    nginx         ClusterIP   None           <none>        80/TCP      7m31s

At this point you may be asking yourself how communication will be made. If clusterIP is set to `none`, Kubernetes will not allocate an IP for the service, in this case, kube-dns will set a domain name for every endpoints that the service points to in the following format

    <podname>.<servicename>.<namespace>.svc.cluster.local
    
Podname and namespace can be obtained this way

    kubectl get pods --all-namespaces
    NAMESPACE     NAME                                                           READY   STATUS    RESTARTS   AGE
    default       web-0                                                          1/1     Running   0          125m
    default       web-1                                                          1/1     Running   0          125m
    default       web-2                                                          1/1     Running   0          124m

For service name, `kubeclt get svc`

    NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
    nginx         ClusterIP   None           <none>        80/TCP      126m
    
This can all be tested through busybox and pinging the pod this way

    kubectl run -it --rm --restart=Never busybox --image=busybox sh
    / #ping web-0.nginx.default.svc.cluster.local


    
    

