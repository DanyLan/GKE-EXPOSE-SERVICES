# NodePort

A NodePort service is the most primitive way to get external traffic directly to your service. ANodePort exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>.



![](https://www.edureka.co/community/?qa=blob&qa_blobid=5351364249810994154)




