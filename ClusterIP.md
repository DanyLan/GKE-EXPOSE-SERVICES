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





