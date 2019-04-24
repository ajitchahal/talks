### Service Mesh
- configurable infrastructure layer
- network‑based interprocess communication 
- ensures fast, reliable, and secure communication
- routing
- circuit breaker
- canary deployments
- service discovery
- load balancing
- encryption
- authentication and authorization
---
### Components of istio
- Mtls
- Ingress Gateway
- Egress Gateway
- Istio side car injector
- Istio policy
- Telemetry
- Citadel
- Pilot
- Galley
- More [Details....](https://istio.io/docs/concepts/what-is-istio/) 
---
### Resources of istio
- Envoy proxy
- Ingress Controller
- Istio Gateway 
- Virtual Service
- Service Entry
- Istio Authorization via istio RBAC
---
### Istio flow
![Image](istio-basics/assets/istio0.8-flow-diagram.png)
---
### How to deploy apps with istio
- Gateway - Define TLS/domain etc. 
- VirtualService - Define routes to applications
- ServiceEntry (To connecto services external to k8s cluster)
- Define service account and RBAC rules
---
### Configuring incoming requests
<span class='menu-title slide-title'>Add routing rules</span>
```yml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-my-gateway
  namespace: argonautsmunich
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      serverCertificate: /etc/istio/ingressgateway-certs/tls.crt
      privateKey: /etc/istio/ingressgateway-certs/tls.key
    hosts:
    - "*.ajit.de"

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: istio-my-vs
  namespace: argonautsmunich
spec:
  hosts:
  - "my-service.ajit.de"
  gateways:
  - istio-my-gateway
  http:
  - match:
    route:
    - destination:
        port:
          number: 8080
        host: my-nginx-with-istio
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="2"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="22">External host</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="27-28">External host</span>

---
### Configuring outgoing requests
<span class='menu-title slide-title'>Add egress rules</span>
```yml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: google-ext
spec:
  hosts:
  - www.google.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS

apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="6-7">External host</span>
---
### RBAC rules
<span class='menu-title slide-title'>Add egress rules</span>
```yml
---
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: argo-services-role
  namespace: argonauts
spec:
  rules:
  - services: ["*.argonauts.svc.cluster.local"]
    methods: ["*"]

apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: argo-services-role-binding
  namespace: argonauts
spec:
  roleRef:
    kind: ServiceRole
    name: "argo-services-role"
  subjects:
  - user: cluster.local/ns/istio-system/sa/istio-ingressgateway-service-account
  - user: cluster.local/ns/api-gateway/sa/ambassador
  - properties:
      source.namespace: "storks"
  - properties:
      source.namespace: "orks"
  - properties:
      source.namespace: "argonauts"
```
### RBAC - Service accounts
To ask for access to other service create SA for the applications
<span class='menu-title slide-title'>Add egress rules</span>
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-application
```
---
## Thank you.
