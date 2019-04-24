### AGENDA
- Why kubernetes
- What is kubernetes
- Minikube Installation / Intro
- Kubectl
- Pod
- Service
+++
   ### AGENDA
- Deployment
- Volumes
- Config sets
- Secrets
---
### Kubernetes
- Cloud agnostic system for deployment of applications/services/databases
- Developed by google, now open source  <!-- .element: class="fragment" -->
- Promotes Infrastructure as code  <!-- .element: class="fragment" -->
- A microservices platform  <!-- .element: class="fragment" -->
- Out of the the box Service Discovery & scaling of applications  <!-- .element: class="fragment" -->
- Containerized deployment (Docker)  <!-- .element: class="fragment" -->
- Out of box resilience - Multi AZ <!-- .element: class="fragment" -->
+++
![Image](k8s-intro/assets/Slide6.1.png)
+++
![Image](k8s-intro/assets/Slide6.2.png)
+++
![Image](k8s-intro/assets/Slide6.png)
---
####  kubectl utility cli
```sh
brew install kubernetes-cli
kubectl version
```
---
#### Kubernetes Pods
- Pods are smallest unit of resource in Kubernetes
- Pod runs 1 or more containers in it
- They are scheduled and managed by services
+++
```sh
23:06 $ kubectl get pods | grep neo4j
neo4j-0                                    1/1       Running   4          6d
neo4j-1                                    1/1       Running   3          13d
neo4j-2                                    1/1       Running   0          13d
neo4j-ha-3247771895-8tkdj                  1/1       Running   2          6d
neo4jcc-0                                  0/1       Pending   0          6h
neo4jcc-1                                  1/1       Running   0          8d
neo4jcc-2                                  1/1       Running   3          8d
neo4jrr-0                                  0/1       Pending   0          6d
neo4jrr-1                                  1/1       Running   0          8d
neo4jrr-2                                  1/1       Running   2          8d
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="1"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2-5">HA Cluster</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="6-11">Causal Cluster</span>
---
### Minikube
- Minikube runs Kubernetes locally in a VM
- Runs a single node
- Can be used to test K8 stack manifests
- <i class="fa fa-hand-o-right" aria-hidden="true"> </i> Maximum number of pods are limited to resources of VM.
  - max 3 Neo4j instances.
+++
### Minikube installation
- brew install kubectl
- brew install cask
- brew cask install minikube
- or curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.24.1/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
+++
### VMware for Minikube installation
- WMware 
  - wget -P ~/Downloads/ http://download.virtualbox.org/virtualbox/5.2.2/VirtualBox-5.2.2-119230-OSX.dmg
  - double click the dmg file and begin installing virtualbox
- minikube start --vm-driver vmware
---
### Exercise - 1
#### Create a Pod
```
$ kubectl run hello-node --image=ajitchahal/hello-node:v3 --port=8080
$ kubectl get pods
$ kubectl port-forward hello-node 8080:8080
$ kubectl get deployments
$ kubectl expose deployment hello-node --type=NodePort
$ kubectl get services
   - note down hello-node   10.0.0.107   <nodes>       8080:32620/TCP   1m
$ minikube ip
run in browser http://192.168.99.100:32620/   
$ or  #minikube service hello-node  -n neo4j-k8s
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="1">Create a pod from a node-js app, that returns current date-time</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2">Check if pods are in running state</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="3">Lets access the service in pod</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="4">Creating pod creates a deployment</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="5">Lets expose the pod on a VM port</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="6-7">Note the node port</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="8-9">Get VM ip and run in a browser</span>
Note:
Do not do type load-balancer in AWS as it creates an ELB.
+++
### Exercise - 2
#### Create a pod & deployment & service via manifest
+++

<span class='menu-title slide-title'>Create a pod via manifest</span>
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: kuad-web-pod
  name: dpl-kuad-pod
  namespace: default
spec:
  volumes:
  - hostPath:
      path: /tmp
    name: kuard-data
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    imagePullPolicy: Always
    name: kuad-container-pod
    ports:
    - containerPort: 8080
      name: web-port
      protocol: TCP
    volumeMounts:
    - mountPath: /tmp-folder
      name: kuard-data


```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2">kubectl apply -f pod.yml</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="9-12">bind a local directory</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="21-24">mount the host folder as volume in container</span>
Note:
Try deleting a pod and see it is not getting recreated again
+++

<span class='menu-title slide-title'>Create deployment via manifest</span>
```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: dpl-kuad
  labels:
   run: kuad-example
spec:
  replicas: 1
  selector:
    matchLabels:
      run: kuad-web
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      name: kuad-pod
      labels:
        run: kuad-web
    spec:
      volumes:
      - name: kuard-data
        hostPath:
          path: /var
        #type: DirectoryOrCreate
      containers:
      - name: kuad-container
        imagePullPolicy: Always
        image: gcr.io/kuar-demo/kuard-amd64:1
        volumeMounts:
        - mountPath: /data
          name: kuard-data
        ports:
        - containerPort: 8080
          name: web-port
          protocol: TCP
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2">kubectl apply -f deployment.yml</span>
Note:
Try deleting a pod and see it is getting recreated again
+++

<span class='menu-title slide-title'>Expose pod via service</span>
```yml
kind: Service
apiVersion: v1
metadata:
  name: kuad-service
spec:
  selector:
    run: kuad-web
  type: NodePort
  ports:
  - protocol: TCP
    port: 8080 # port of service, it will be exposed to other pods inside the cluster
    targetPort: 8080 # port of container app runnning e.g. spring boot on 8080
    nodePort: 32711 # port opened on host VM so it is accessible outside cluster
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="2">kubectl apply -f service.yml</span>
Note:
Access the pod in browser http://192.168.99.100:32620/
+++
## Exercise - 3
### SSH into the container & try some tricks
```
kubectl exec -it dpl-kuad-5db8f969cb-9rzjz sh
wget hello-node-c4ccdf565-9ss8j:8080 -P /tmp/
wget hello-node:8080 -P /tmp/
nslookup hello-node
nslookup hello-node-c4ccdf565-9ss8j
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="1">Use - kubectl get pods | grep kuad to know pod name</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="2">Lets access our hello-node app from kuad app</span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="3">Lets try again with service</span>
+++
#### Moral: Services Expose pods to other pods
#### Moral-2: Services Expose pods to applications external to Kubernetes
---
### Connect to kubernetes in AWS
```
apiVersion: v1
kind: Config
clusters:
- name: k8s-neo4j-k8s-stage
  cluster:
    server: https://my-kubernetes.io:443
    certificate-authority-data: xxxx
users:
- name: neo4j-k8s
  user:
    client-certificate-data: xxxx
    client-key-data: xxxxx
contexts:
- name: neo4j-k8s
  context:
    cluster: k8s-neo4j-k8s-stage
    user: neo4j-k8s
    namespace: neo4j-k8s
current-context: neo4j-k8s
```
#### Copy content & Save to k8_stage.config file
+++
- To connect to Stage
  - export KUBECONFIG=~/kubernetes/k8_stage.config
  - export KUBECONFIG=~/.kube/config
---
### Statefulsets
- Static & Predictable names of pods
- DNS discovery of individual pods with services  <!-- .element: class="fragment" -->
- Persistent Volumes, that are independent of POD lifecycle  <!-- .element: class="fragment" -->
- Same PV gets attached to same POD, regardless of which node it scheduled on.  <!-- .element: class="fragment" -->
- Persistent volume claims  <!-- .element: class="fragment" -->
  - claims leads to creation of volumes in correct Availability Zones  <!-- .element: class="fragment" -->
+++
## Exercise - 4
### Play with statefulset
+++

<span class='menu-title slide-title'>Create a service for Statefulset pods</span>
```yml
apiVersion: v1
kind: Service
metadata:
  name: neo4j-ha
spec:
  clusterIP: None
  selector:
    app: neo4j-ha
  ports:
  - name: rest
    port: 7474

```
<span class="code-presenting-annotation fragment current-only" data-code-focus="2">save to a file and run kubectl apply -f svc.yml</span>

+++

<span class='menu-title slide-title'>Create a Statefulset</span>
```yml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: neo4j
spec:
  serviceName: neo4j-ha
  replicas: 2
  template:
    metadata:
      labels:
         app: neo4j-ha
    spec:
      containers:
        - name: neo4j-container
          imagePullPolicy: IfNotPresent
          image: ajitchahal/neo4j-test:v2
          ports:
          - containerPort: 5001
            name: ha-host-cord
          - containerPort: 6001
            name: host-data
          - containerPort: 6362
            name: backup
          - containerPort: 7474
            name: rest
          - containerPort: 7687
            name: bolt
          env:
          - name: NEO4J_dbms_mode
            value: HA
          - name: NEO4J_ha_initialHosts
            value: neo4j-0.neo4j:5001,neo4j-1.neo4j:5001 #,neo4j-2.neo4j:5001
          - name: NEO4J_AUTH
            value: neo4j/admin
          - name: NEO4J_ha_host_coordination
            value: 0.0.0.0:5001
          - name: NEO4J_ha_host_data
            value: 0.0.0.0:6001
          command: ["/bin/sh"]
          args: [ "-c",
                  "export NEO4J_ha_serverId=$(hostname | awk -F'-' '{print $2}') \
                  && /neo4j-enterprise-3.2.6/bin/neo4j-admin set-initial-password 'ajit' && /neo4j-enterprise-3.2.6/bin/neo4j console"]
          #bin/neo4j start" #sleep 1000000"

```
<span class="code-presenting-annotation fragment current-only" data-code-focus="2">save to a file and run kubectl apply -f set.yml</span>

+++

<span class='menu-title slide-title'>Create a service so we can expose pod of Statefulset</span>
```yml
apiVersion: v1
kind: Service
metadata:
  name: neo4j-ha-lb
spec:
  type: NodePort
  selector:
    app: neo4j-ha
  ports:
  - name: rest
    port: 7474
    targetPort: 7474
    nodePort: 30074
  - name: bolt
    port: 7687
    targetPort: 7687
    nodePort: 30087
```
<span class="code-presenting-annotation fragment current-only" data-code-focus="2">save to a file and run kubectl apply -f svc-lb.yml</span>

+++
#### Lets access some info about pods & access the neo4j in browser
```
kubectl get statefulsets
kubectl get pods
kubectl get services
http://192.168.99.100:30074/browser/
create(n:Emp {name:'Ajit'})
match(n) return n
```
---
#### Ich bedanke mich für Ihre aufmerksamkeit
#### Les agradezco su atención y su apoyo.
#### आपके ध्यान के लिए धन्यवाद.
---
