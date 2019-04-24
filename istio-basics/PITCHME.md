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
---
### Configuring incoming requests
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
---
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
---
### Configuring outgoing requests
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
---
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
## Thank you.
