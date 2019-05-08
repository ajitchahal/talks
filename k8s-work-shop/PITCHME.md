### AGENDA
-	Connecting to k8s
-	Intro to k8s resources of cds
-	Creating a pod/deployment/service
-	Exposing apps to internet via ingress
-	Accessing services outside k8s cluster
-	Metrics
- Logging
-	Accessing aws s3 logs
-	Creating a k8s deployment pipeline
-	Intro to helm charts 
- Concourse
- Jenkins
---
### Connecting to k8s
- Download the kubeconfig from https://login.us.k8s.yaas.io/
- In future a permanent kube config with ldap credentials.
- Copy it to a file  (~/.kube/stage-g)
- Export $KUBECONFIG=~/.kube/stage-g
- ```kubectl -n kiwis get pods```
- Trouble shooting 
  - HA proxy url https://argonautsmunich-g-api-int.stage.us.k8s.yaas.io 
  - Is reachable from vpn/office and Jenkins
  - Idefix manages this url
---
### k8s resources of cds
- istio Service mesh
  - Side car injection
    - ```kubectl label namespace argonautsmunich istio-injection=enabled````
    - Deployment
    - ```yml
      spec:
        template:
          metadata:
            annotations:
              sidecar.istio.io/inject: “false”
       ``` 

+++
- Ingress
  - Istio
    - Define virtual service [Ref](https://wiki.hybris.com/x/_ruHG)
    -   
  - K8s nginx - for static content
  - Ambassador for public apis
    - Authenticated url with JWT tokens 

---
