# CKA Mock exam 2
## where i needed the docs
- lookup storage class definition
- pv definition
	- how to mount localpath
	- add node affinity
		- matchexpression
		- matchfield
		- cluster1-controlplane ~ ➜  k apply -f pv.yaml 
Error from server (BadRequest): error when creating "pv.yaml": PersistentVolume in version "v1" cannot be handled as a PersistentVolume: json: cannot unmarshal object into Go struct field NodeSelectorTerm.spec.nodeAffinity.required.nodeSelectorTerms.matchExpressions of type []v1.NodeSelectorRequirement
apiVersion: v1
kind: PersistentVolume
metadata:
  name: orange-pv-cka07-str
spec:
  capacity:
    storage: 150Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: orange-stc-cka07-str
  hostPath:
    path: /opt/orange-data-cka07-str
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
     _     key: kubernetes.io/hostname_
          operator: In 
          values:
          - cluster1-controlplane
  - pvc definition
> how to send a command over a number of nodes via ssh if i have to look up information on different hosts?

- VPA 
	- everything to this question: "Deploy a **Vertical Pod Autoscaler (VPA)** named `analytics-vpa` for a deployment named `analytics-deployment` in the `cka24456` namespace. The **VPA** should automatically adjust the **CPU** and **memory** requests of the pods to optimize resource utilization. Ensure that the **VPA** operates in `Auto` mode, allowing it to `evict` and `recreate` pods with updated resource requests as needed."
	- https://kubernetes.io/docs/concepts/workloads/autoscaling/vertical-pod-autoscale/
	- solution: apiVersion: autoscaling.k8s.io/v1 kind: VerticalPodAutoscaler metadata: name: analytics-vpa namespace: cka24456 spec: targetRef: apiVersion: apps/v1 kind: Deployment name: analytics-deployment updatePolicy: updateMode: "Auto"
> how to check pods/applications with curl?- how to use curl inside troubleshoot pod?
- difference of port types in service
- reread: how to services select their pods (labels and stuff)
- configure env (from secret) (and read secrets page)
> question 12 solution?
	- - Under `rules:` -> `host:` change `example.com` to `kodekloud-ingress.app`
- Under `backend:` -> `service:` -> `name:` Change `example-service` to `nodeapp-svc-cka08-trb`
_- Change `port:` -> `number:` from `80` to `3000`_ <--- this was still wrong

- API Gateway
	- configure https/tls for gateway
	- how to check functionality of resources? (question 14)
- troubleshoot nodes
> question 16 answer?
**- i was missing the correct kulebet version**
whole answer:
#### SSH into the Control Plane

Run the following command to access **cluster2-controlplane**:

```sh
ssh cluster2-controlplane
```

#### 1. Check Cluster Status

Run `kubectl get nodes` to check the cluster status:

```sh
cluster2-controlplane ~ ➜  kubectl get nodes
The connection to the server cluster2-controlplane:6443 was refused - did you specify the right host or port?
```

This indicates that the **API server is down**.

---

#### 2. Verify Kubernetes Containers Are Running

Check if all the Kubernetes-related containers are running:

```sh
cluster2-controlplane ~ ➜ crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
fbbf1db37defb       ead0a4a53df89       5 hours ago         Running             coredns                   0                   4be392af8ae56       coredns-69f9c977-lsjwc
b7a2fc42b8b83       ead0a4a53df89       5 hours ago         Running             coredns                   0                   8fdd89e10d6d7       coredns-69f9c977-bfsfw
63f27f8d81c29       a0eed15eed449       5 hours ago         Running             etcd                      0                   3e093d91c4361       etcd-cluster2-controlplane
f07cb1acaa26f       0824682bcdc8e       5 hours ago         Exited              kube-controller-manager   0                   fba0d299aab89       kube-controller-manager-cluster2-controlplane
2f36c95ff0b7e       7ace497ddb8e8       5 hours ago         Exited              kube-scheduler            0                   11e422ba2d28b       kube-scheduler-cluster2-controlplane
```

The `kube-apiserver` container **is missing**.

---

#### 3. Check `kubelet` Logs

Since `kube-apiserver` is missing, check if `kubelet` is running:

```sh
cluster2-controlplane ~ ➜ systemctl status kubelet

Unit kubelet.service could not be found.
```

**This confirms that `kubelet` is missing or not installed.**

---

#### 4. Reinstall `kubelet`

Since `kubelet` is missing, we need to **install it manually**.

- Find the Correct Kubelet Version

```sh
cluster2-controlplane ~ ➜ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.0", GitCommit:"70d3cc986aa8221cd1dfb1121852688902d3bf53", GitTreeState:"clean", BuildDate:"2024-12-11T18:04:20Z", GoVersion:"go1.23.3", Compiler:"gc", Platform:"linux/amd64"}
```

The **`kubelet` version must match `kubeadm`**.

- Install `kubelet` (Matching Version)

```sh
sudo apt install -y kubelet=1.32.0-1.1
```

- Start the `kubelet` Service**

```sh
sudo systemctl start kubelet
```

---

#### 5. Verify That the Cluster is Restored

Wait a few moments for `kubelet` to bring back `kube-apiserver`, then check the cluster status:

```sh
cluster2-controlplane ~ ➜ kubectl get nodes

NAME                    STATUS   ROLES           AGE   VERSION
cluster2-controlplane   Ready    control-plane   27m   v1.32.0
cluster2-node01         Ready    <none>          26m   v1.32.0
```


# mock exam 3
- how to expose multiple pods one service
- custom columns & sort
- can you choose columns with k get customresourcedefinitions?
- HPA
- persistentvolumeclaim podspec
- ready/live probes for troubleshooting

## q1
> read carefully, did not change number for request

## q5
> did not specify namespace 

q8
> did not specify correct amount of storage request (150 instead of 100)

# q9
> you need to specify the namespce for the gateway which is the parentref

# q10
learn command kubectl get event
> otherwise correct, looks like a bug

# q11 
> again wrong capacity on pvc. BUT it is requested to get 100Mi, is has a capacity of 150Mi ?!

mine:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      initContainers:
        - name: busybox
          image: busybox
          restartPolicy: Always
          command: ['sh', '-c', 'sleep 3600']
          volumeMounts:
            - name: python-data
              mountPath: /usr/src
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
  volumeName: olive-pv-cka10-str
  storageClassName: olive-stc-cka10-str

### solution:

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: olive-stc-cka10-str
  volumeName: olive-pv-cka10-str
  resources:
    requests:
      storage: 100Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      - name: busybox
        image: busybox
        command:
          - "bin/sh"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: python-data
            mountPath: "/usr/src"
            readOnly: true
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str

---
apiVersion: v1
kind: Service
metadata:
  name: olive-svc-cka10-str
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 32006
  selector:
    app: olive-app-cka10-str
```

> Service does not matter

# q14
> lets learn what endpointslices are ig

Solve this question on: `ssh cluster3-controlplane`  
  
  
We have an **external** webserver running on `student-node` which is exposed at port `9999`. We have created a service called `external-webserver-cka03-svcn` that can connect to our local webserver from within the kubernetes **cluster3**, but at the moment, it is not working as expected.  
  
  
  
Fix the issue so that other pods within **cluster3** can use **external-webserver-cka03-svcn** service to access the webserver.  
  

Solution

##### SSH into the `cluster3-controlplane` host

```sh
ssh cluster3-controlplane
```

Let's check if the webserver is working or not:

```
cluster3-controlplane  ~ ➜  curl student-node:9999
...
<h1>Welcome to nginx!</h1>
...
```

Now we will check if service is correctly defined:

```
cluster3-controlplane  ~ ➜  kubectl describe svc -n kube-public external-webserver-cka03-svcn 
Name:              external-webserver-cka03-svcn
Namespace:         kube-public
.
.
Endpoints:         <none> # there are no endpoints for the service
...
```

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

First, obtain the IP address of the student node, easiest way is to ping it:

```
root@student-node ~ ✦  ping student-node
PING student-node (192.168.222.128) 56(84) bytes of data.
64 bytes from student-node (192.168.222.128): icmp_seq=1 ttl=64 time=0.023 ms
64 bytes from student-node (192.168.222.128): icmp_seq=2 ttl=64 time=0.030 ms
```

In this example : student-node IP is `192.168.222.128`

Next, use the IP address to create the EndpointSlice:

```
cluster3-controlplane  ~ ➜ kubectl  apply -f - <<EOF
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-webserver-cka03-svcn
  namespace: kube-public
  labels:
    kubernetes.io/service-name: external-webserver-cka03-svcn
addressType: IPv4
ports:
  - protocol: TCP
    port: 9999
endpoints:
  - addresses:
      - 192.168.222.128   # IP of student node
EOF
```

Finally check if the `curl test` works now:

```
cluster3-controlplane  ~ ➜  kubectl run -n kube-public --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-cka03-svcn
...
<title>Welcome to nginx!</title>
...
```


# q15 
> did not change the egress part

Solve this question on: `ssh cluster1-controlplane`  
  
  
An nginx-based pod called `cyan-pod-cka28-trb` is running under the `cyan-ns-cka28-trb` namespace and is exposed within the cluster using the `cyan-svc-cka28-trb` service.  
  
This is a `restricted` pod, so a network policy called `cyan-np-cka28-trb` has been created in the same namespace to apply some restrictions on this pod.  
  
  
Two other pods called `cyan-white-cka28-trb` and `cyan-black-cka28-trb` are also running in the `default` namespace.  
  
  
The nginx-based app running on the `cyan-pod-cka28-trb` pod is exposed internally on the default nginx port (`80`).  
  

**Expectation:** This app should `only` be accessible from the `cyan-white-cka28-trb` pod.  
  
  
**Problem:** This app is not accessible from anywhere.  
  
  
Troubleshoot this issue and fix the connectivity as per the requirement listed above.  
  
  
**Note:** You can exec into `cyan-white-cka28-trb` and `cyan-black-cka28-trb` pods and test connectivity using the `curl` utility.  
  
  
You may update the network policy, but make sure it is not deleted from the `cyan-ns-cka28-trb` namespace.

Solution

##### SSH into the `cluster1-controlplane` host

```sh
ssh cluster1-controlplane
```

  
  
Let's look into the network policy

```sh
kubectl edit networkpolicy cyan-np-cka28-trb -n cyan-ns-cka28-trb
```

Under `spec:` -> `egress:` you will notice there is not `cidr:` block has been added, since there is no restrcitions on `egress` traffic so we can update it as below. Further you will notice that the port used in the policy is `8080` but the app is running on default port which is `80` so let's update this as well (under `egress` and `ingress`):

- Change `port: 8080` to `port: 80`

```yaml
- ports:
  - port: 80
    protocol: TCP
  to:
  - ipBlock:
      cidr: 0.0.0.0/0
```

Now, lastly notice that there is no POD selector has been used in `ingress` section but this app is supposed to be accessible from `cyan-white-cka28-trb` pod under `default` namespace. So let's edit it to look like as below:

```yaml
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: default
    podSelector:
      matchLabels:
        app: cyan-white-cka28-trb
```

Now, let's try to access the app from `cyan-white-pod-cka28-trb`

```sh
kubectl exec cyan-white-cka28-trb -- sh -c 'curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local'
```

Also make sure its not accessible from the other pod(s)

```sh
kubectl exec cyan-black-cka28-trb -- sh -c 'curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local'
```

It should not work from this pod. So its looking good now.